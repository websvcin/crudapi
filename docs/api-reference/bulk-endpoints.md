---
title: Bulk endpoints
parent: API reference
nav_order: 2
---

# Bulk endpoints
{: .no_toc }

Three ways to load many rows into a table at once, without one-row-at-a-time CRUD calls.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Base address

```
/api/v1/{tenant}/{database}/bulk/...
```

Same credential rules as [CRUD endpoints](crud-endpoints.html). There are **three independent bulk
sources** — a CSV file, a filtered copy of another table, or an external API — each with its own template
endpoint and its own import endpoint, all driving the same underlying bulk pipeline underneath. This is also
exactly what powers the tenant console's **Bulk import** screen — see
[Features → Bulk import](../features/bulk-import.html) for the UI walkthrough.

## Source 1 — CSV file upload

**Get a template** — `GET /bulk/template/{table}/csv?includeAnnotations=true`

Downloads a one-row CSV shaped for that table: a header for every importable column (excluding
auto-increment ids and auto-managed timestamps), with a sample value in the data row. With
`includeAnnotations=true` (the default), headers for foreign-key columns and enum columns carry extra
machine-readable hints inline:

- Foreign key: `CustomerName|customer_id (map:customers.id=CustomerName)`
- Enum: `status (enum:active,inactive,pending)`

**Import** — `POST /bulk/upload/{table}` (multipart form, field `file`)

```bash
curl -X POST -H "X-API-Key: drk_..." \
  -F "file=@widgets.csv" \
  "https://your-host/api/v1/acme/reports/bulk/upload/widgets?executionMode=Auto&chunkSize=1000"
```

| Query parameter | Default | Does |
|---|---|---|
| `executionMode` | `Auto` | `Auto` runs immediately if the row count is small (see below), otherwise queues it; `Realtime` always runs immediately in this request; `Background` always queues it. |
| `chunkSize` | `1000` | Rows per batch insert internally. |
| `notifyEveryNRows` | `500` | How often a queued job's progress is checkpointed. |

Rejected: a missing file, a file over the configured size limit, or an extension not in the allowed list
(`.csv` by default — both are configurable platform-wide, see
[Admin console → Settings](../admin-console/settings.html)).

## Source 2 — copy rows from another table, with a filter

**Get a template** — `GET /bulk/template/{table}/filter` — returns an example body shape, not a file.

**Import** — `POST /bulk/filter/{table}`

```bash
curl -X POST -H "X-API-Key: drk_..." -H "Content-Type: application/json" \
  -d '{"sourceTable":"orders_2024","filter":"orders_2024.status eq '\''closed'\''","data":{"archived":true}}' \
  "https://your-host/api/v1/acme/reports/bulk/filter/widgets"
```

`sourceTable` is read with the same [OData filter](filtering-odata.html) syntax CRUD list endpoints use.
Every matched row is copied into `{table}`, with `data` applied as a fixed override on every copied row
(e.g. stamping `archived: true` on all of them). `sourceTable` and `{table}` don't need to be the same
shape — only the columns in common (plus whatever `data` supplies) actually get written.

## Source 3 — pull from an external API

**Get a template** — `GET /bulk/template/{table}/api-source` — an example request shape with a starter
column mapping.

**Import** — `POST /bulk/api-source/{table}`

```bash
curl -X POST -H "X-API-Key: drk_..." -H "Content-Type: application/json" \
  -d '{
        "endpoint": "https://external-api.example.com/customers",
        "method": "GET",
        "headers": { "Authorization": "Bearer <their token>" },
        "mapping": { "name": "external_name", "email": "external_email" }
      }' \
  "https://your-host/api/v1/acme/reports/bulk/api-source/widgets"
```

CrudApi calls `endpoint` itself (`GET` or `POST`, with your `headers`/`body`), maps each returned object's
fields per `mapping` (`{ targetColumn: sourceField }`), and bulk-inserts the result.

## Execution mode: realtime vs background

All three import endpoints share `executionMode`:

- **`Realtime`** — runs in the same HTTP request; the response is the final result.
- **`Background`** — queued immediately (via Hangfire); the response is a `jobId` and an estimated row
  count, not a final result.
- **`Auto`** (default) — behaves like `Realtime` when the row count is at or under the platform's
  realtime-row threshold (10 by default), and like `Background` above it. This exists so a small ad-hoc
  import gets an instant answer while a large one doesn't tie up the request.

A **realtime** response:

```json
{ "status": "completed", "referenceId": "orders:9f2a...", "totalRecords": 42, "successCount": 40, "failedCount": 2 }
```

A **queued** response:

```json
{ "status": "queued", "referenceId": "orders:9f2a...", "jobId": "173", "estimatedRecords": 50000 }
```

## Row-policy enforcement on filter and API-source imports

If the caller's role carries a [row policy](../features/row-policies.html), it's enforced on the filter and
API-source paths before anything is written: rows that would violate the policy are rejected rather than
silently included. When *every* row in the batch is rejected, the response is a CSV attachment listing each
rejected row and the reason, instead of a success body — so a blocked bulk import is as diagnosable as a
blocked single-row write. (CSV upload enforces the same per-row rules inside the pipeline itself.)

## Errors

A realtime run that has any failed rows returns a **CSV attachment** of just the failed rows and their error
messages (`{table}_import_errors.csv`, `{table}_filter_errors.csv`, or `{table}_api_errors.csv` depending on
the source) rather than a JSON body — so a partially-failed import is easy to re-run for just the rows that
didn't make it. A fully successful run, or a queued run, returns plain JSON as shown above.
