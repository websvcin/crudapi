---
title: Developer API
parent: Tenant console
nav_order: 9
---

# Developer API
{: .no_toc }

Every action you can take from this console, as a REST call — for scripts and machine-to-machine jobs that
can't click through a UI. 105 routes across 9 capabilities, all scoped to this workspace only.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

{: .note }
> Separate from the [Keys](keys.html) tab (API key, Basic login, issuance keys) — those call your
> tenant's **data** API. This page is for automating the **console** itself: adding users, requesting
> LoginLink, reading activity, and everything else this console's UI can do.

## Workspace Developer Keys

A **Workspace Developer Key** (`wdk_`) is capability-scoped, created and revoked entirely from this page.

1. **Create a key** — tick what it needs, name it. Shown once; copy it then.
2. **Call an endpoint** — `Authorization: Bearer <key>`. No token exchange.
3. **Scoped to this workspace** — resolved from the key itself, never from the request, so it can never
   see or touch another tenant's data. A missing capability gets `403 insufficient_scope`.

## The 9 capabilities

| Capability | Covers |
|---|---|
| **Users** | List, create, update — `/api/tenant/users*`. |
| **Overview** | Read-only workspace stats — `/overview`, `/dashboard`, `/usage`. |
| **Settings** | Workspace settings & security policy — `/settings`, `/security`. |
| **Databases** | Registrations, CORS, tables, scripts, row policies, managed databases — `/databases*`, `/row-policies`, `/managed-databases*`. |
| **Access** | Sessions, grants, access options — `/access`, `/access/grants`, `/access/sessions/*`. |
| **LoginLink** | Connect, self-service, org/people/roles — `/access/loginlink/*`, `/loginlink/*`. |
| **Roles** | This workspace's own custom roles — `/roles*`. |
| **API credentials** | Data-API credentials & sessions — API clients, Basic logins, issuance keys, JWT sessions — `/api-clients`, `/basic-auth`, `/issuance-keys`, `/jwt-sessions`, `/keys`. |
| **Audit** | Read the activity log — `/activity*`, `/audit-logs`. |

**Never reachable by any key, regardless of capability:** creating or revoking Developer Keys themselves
(`/developer-keys`), and console keys (`/console-keys` — a different, more powerful credential; letting a
Developer Key mint one would be an escalation path).

## Try it — onboard a user and request LoginLink

```bash
# 1. Create a user — requires the Users capability
curl -X POST https://host/api/tenant/users \
  -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" \
  -d '{"externalId":"kenji@acme.example","displayName":"Kenji Sato"}'
# 200 { "id": 318 }
# 409 { "error": "user_exists" } — externalId already exists in this workspace

# 2. Request LoginLink self-service — requires the Access & LoginLink capabilities
curl -X POST https://host/api/tenant/access/loginlink/request-connect \
  -H "Authorization: Bearer $KEY"
# 200 { "requested": true, "requestedAt": "2026-10-11T.." }
# 409 { "error": "already_allowed" } — the platform admin already switched it on
```

## Representative endpoints

| Method | Path | Capability |
|---|---|---|
| `GET` | `/api/tenant/users` | Users |
| `POST` | `/api/tenant/users` | Users |
| `GET` | `/api/tenant/overview` | Overview |
| `PUT` | `/api/tenant/settings` | Settings |
| `POST` | `/api/tenant/databases` | Databases |
| `POST` | `/api/tenant/databases/{id}/generate-config?ifExists=ask\|replace\|use_existing` | Databases |
| `GET` | `/api/tenant/databases/{id}/config-file` | Databases |
| `POST` | `/api/tenant/access/sessions/{id}/end` | Access |
| `POST` | `/api/tenant/access/loginlink/request-connect` | LoginLink |
| `POST` | `/api/tenant/roles` | Roles |
| `POST` | `/api/tenant/api-clients` | API credentials |
| `GET` | `/api/tenant/activity` | Audit — filters: `event`, `q`, `from`, `to`, `outcome`, `database`, `limit` (max 500) |

## Errors

| Error | Meaning |
|---|---|
| `401 invalid_key` | No `Authorization` header, or the key is wrong, revoked, or malformed. |
| `403 insufficient_scope` | A real key missing the capability this route requires, or a route never reachable by any key at all. |
| `400 validation_failed` / `409 conflict` | A malformed body, or state that already matches — nothing fails silently. |

See [Admin console → Developer API](../admin-console/developer-api.html) for the platform-wide equivalent.
