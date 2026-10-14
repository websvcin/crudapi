---
title: CRUD endpoints
parent: API reference
nav_order: 1
---

# CRUD endpoints
{: .no_toc }

The core of the product: every registered table, as a generic REST resource.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Base address

```
/api/v1/{tenant}/{database}/crud/{table}
```

- `{tenant}` — the tenant's slug.
- `{database}` — the database's own key (unique within that tenant; two different tenants may use the same
  database key, since the pair `{tenant}/{database}` together is what's actually unique).
- `{table}` — a real table name in that database. There is no separate step to "expose" a table — every
  table reachable by your credential's role is already a live endpoint.

Every request needs a credential header — see
[Concepts → Authentication & credentials](../concepts/authentication-and-credentials.html) for which header
to use for which credential type.

## List rows — `GET /crud/{table}`

Returns every row the caller's role and row policy allow, narrowed by whichever query parameters you send.

| Parameter | Example | Does |
|---|---|---|
| `filter` | `?filter=orders.status eq 'open'` | OData-style filter — see [Filtering](filtering-odata.html). **Every column must be qualified as `table.column`** (`status` alone is refused). |
| `limit` | `?limit=50` | Caps the number of rows returned. |
| `orderBy` | `?orderBy=created_at:desc` | Sort order, as `column:asc` or `column:desc`; comma-separate for multiple columns (`?orderBy=status:asc,created_at:desc`). |
| `groupBy` | `?groupBy=status` | Group rows by a column. |
| `join` | `?join=customers` | Include related rows from a foreign-keyed table. |
| `display` | `?display=name,email` | Return only the named columns. |

**An empty result set is a `404`, the same as `GET {table}/{column}/{value}` below returning nothing** —
there's no "200 with an empty array" case on either read path. If your integration treats "no rows matched"
as a normal, expected outcome (e.g. a dashboard list that's sometimes legitimately empty), handle a `404`
from this endpoint as that case, not as an error to surface to a user.

## Get one record — `GET /crud/{table}/{column}/{value}`

A convenience shortcut for "the row where `{column}` equals `{value}`" — equivalent to
`GET /crud/{table}?filter={table}.{column} eq '{value}'&limit=1`, but returns the single object directly
(or a `404`) instead of an array.

```bash
curl -H "X-API-Key: drk_..." \
  "https://your-host/api/v1/acme/reports/crud/widgets/id/42"
```

## Create a row — `POST /crud/{table}`

Body is a flat JSON object of column → value. If a [row policy](../features/row-policies.html) applies to
the caller, its fixed columns (e.g. `region = 'APAC'`) are automatically filled in on the created row — you
don't need to (and for a single-value policy column, can't override it to something the policy forbids).

```bash
curl -X POST -H "X-API-Key: drk_..." -H "Content-Type: application/json" \
  -d '{"name":"New Widget","status":"active"}' \
  "https://your-host/api/v1/acme/reports/crud/widgets"
```

Returns the created row, including any server-assigned fields (auto-increment id, default timestamps).

## Update a row — `PUT /crud/{table}/{idColumn}/{idValue}`

Body is the column(s) to change. Only the columns you send are touched. If a row policy applies, the target
row must *already* match it — attempting to update a row outside your policy's scope is refused, not
silently redirected to a different row.

```bash
curl -X PUT -H "X-API-Key: drk_..." -H "Content-Type: application/json" \
  -d '{"status":"shipped"}' \
  "https://your-host/api/v1/acme/reports/crud/widgets/id/42"
```

## Delete a row — `DELETE /crud/{table}/{idColumn}/{idValue}`

Same row-policy enforcement as update: a row outside your policy's scope can't be deleted, and the response
for that case is a clear `row_policy_forbidden`, not a misleading `404` that implies the row never existed.

```bash
curl -X DELETE -H "X-API-Key: drk_..." \
  "https://your-host/api/v1/acme/reports/crud/widgets/id/42"
```

## Errors

Every error response is JSON with at least `error` (a short machine-readable code) and `errorDescription`
(a human-readable sentence). Common ones:

| HTTP status | `error` | Meaning |
|---|---|---|
| 401 | `unauthorized` | No credential, or it wasn't recognized. |
| 403 | `forbidden_action` | The credential's role doesn't hold the permission this action needs (e.g. `widgets.delete`). |
| 403 | `row_policy_forbidden` | The target row exists, but is outside the caller's row policy. |
| 404 | — | No matching row(s) — from a list query that matched nothing, or a `GET .../{column}/{value}` whose value doesn't exist. |
| 400 | `server_error` (with detail) | Usually a filter whose column isn't qualified as `table.column` — see [Filtering](filtering-odata.html). |

## What permission a request actually needs

Permissions are expressed as `{table}.{action}` — e.g. `widgets.read`, `widgets.create`, `widgets.update`,
`widgets.delete` — and a role's permissions use `*` as a wildcard for "every table" (`*.read` = read every
table). There's no separate "endpoint permission" concept distinct from the table name: if a role can
`{table}.read`, the corresponding `GET` works; if it can't, you get `forbidden_action`, not a `404` that
hides whether the table even exists.
