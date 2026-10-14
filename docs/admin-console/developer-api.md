---
title: Developer API
parent: Admin console
nav_order: 3
---

# Developer API
{: .no_toc }

`/admin/developerapi` — every admin action as a REST call, for scripts and machine-to-machine jobs that
can't drive the UI. 149 routes across 13 capabilities, plus the one key type that calls them.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Platform Developer Keys

A **Platform Developer Key** (`pdk_`) is capability-scoped — created, named, and revoked entirely from this
page. It is the one key type for every platform-level automation need (it also absorbed what used to be a
separate "Platform API key"; see [below](#the-old-platform-api-key)).

1. **Create a key** — tick the capabilities it needs, name it. Shown once; copy it immediately.
2. **Call an endpoint** — send it as `Authorization: Bearer <key>`. No token exchange step.
3. **Scoped, not all-or-nothing** — a key missing a capability gets `403 insufficient_scope` on that route.

## The 13 capabilities

| Capability | Covers |
|---|---|
| **Tenants** | Create, search, suspend, managed databases — `/api/admin/tenants*`, `/tenant-sessions*`. Can optionally be narrowed to specific tenants with a creation quota — see below. |
| **Support view** | Open a tenant's console as the tenant, mint its console key, or mint it a Workspace Developer Key directly. |
| **LoginLink** | Connect, self-service, provisioning — `/api/admin/tenants/{id}/loginlink/*`, `/tenants/loginlink/*`. |
| **Access model** | Access defaults, roles, permissions, row policies — `/api/admin/access/*`, `/roles`, `/permissions`, `/row-policies`. |
| **Users** | The `/api/admin/users` surface. |
| **Databases** | Registrations, CORS, engine roadmap — `/api/admin/databases*`, `/engine-roadmap`, `/supported-engines`. |
| **Storage** | Migrate-to-MySQL, recovery kit, provisioning account — `/api/admin/storage/*`, `/migrate-mysql/*`. |
| **Signing keys** | Rotate or retire the platform's own JWT signing keys — `/api/admin/signing-keys/*`. Its own capability, deliberately not folded into anything else, since it's the most sensitive one. |
| **API credentials** | The tenant data-API's own credentials & sessions, excluding tenant-owned API client keys — `/basic-auth-clients`, `/issuance-keys`, `/jwt-sessions`, `/jwt/issue`. |
| **Admins** | Admin accounts, platform settings, Host Settings — `/admin-users*`, `/settings`, `/host-settings`. |
| **Audit** | Read and purge the audit log — `/api/admin/audit-logs*`. |
| **Dashboard** | Read-only platform stats — `/api/admin/dashboard*`. |
| **Surfaces** | Read-only "who can do what" report — `GET /api/admin/surfaces`. |

**Never reachable by any Developer Key, regardless of capability:** creating or revoking Developer Keys
themselves (human console session only — so a leaked key can never mint itself a replacement), and the
one-time setup/bootstrap diagnostics routes.

## Narrowing the Tenants capability

Ticking **Tenants** reveals an optional reach-and-quota panel — this is the "old Platform API key" use
case, now just a configuration of this one key type:

| Reach | Meaning |
|---|---|
| **One tenant** | Smallest blast radius — a service that only ever works for a single customer. |
| **Selected tenants** | A specific few you pick, searchable, add as many as needed. |
| **All tenants** | Every tenant, including ones added later. A creation limit still applies if set. |
| **Only tenants it creates** | For signup automation — the key can only ever see or change tenants *it itself created*; optionally require new tenant ids to start with a given prefix. |

Plus an optional **tenant creation limit** (blank = unlimited) and an optional **expiry date** (blank =
never).

## Try it — signup automation example

A key with Tenants (reach: "Only tenants it creates") and LoginLink:

```bash
# 1. Create a tenant — honors reach & quota if the key was narrowed
curl -X POST https://host/api/admin/tenants \
  -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" \
  -d '{"id":"acme","name":"Acme Co","slug":"acme"}'
# 200 { "ok": true, "loginlink": { "status": "off" } }
# 403 { "error": "tenant_id_not_allowed" }  — id doesn't match the key's required prefix
# 429 { "error": "quota_exceeded" }          — the key's tenant-creation limit was already reached

# 2. Turn on LoginLink self-service for it — requires the LoginLink capability
curl -X PUT https://host/api/admin/tenants/acme/loginlink/self-service \
  -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" \
  -d '{"enabled":true}'
# 200 { "enabled": true }
```

## Representative endpoints

| Method | Path | Capability |
|---|---|---|
| `GET` | `/api/admin/tenants` | Tenants |
| `POST` | `/api/admin/tenants` | Tenants |
| `PUT` | `/api/admin/tenants/{id}/loginlink/self-service` | LoginLink |
| `POST` | `/api/admin/tenants/{id}/support-view` | Support view |
| `GET` | `/api/admin/access/overview` | Access model |
| `GET` | `/api/admin/users` | Users |
| `POST` | `/api/admin/databases` | Databases |
| `GET` | `/api/admin/migrate-mysql/status` | Storage |
| `POST` | `/api/admin/signing-keys/rotate` | Signing keys |
| `GET` | `/api/admin/admin-users` | Admins |
| `GET` | `/api/admin/audit-logs` | Audit |
| `GET` | `/api/admin/dashboard/overview` | Dashboard |
| `GET` | `/api/admin/surfaces` | Surfaces |

The full set of all 149 routes, grouped by capability, is listed directly on this screen's own "Getting
started" panel — this table is a representative sample, not the complete list.

## Errors

| Error | Meaning |
|---|---|
| `401 unauthorized` / `invalid_token` | No `Authorization` header, or the key is wrong, revoked, expired, or malformed. |
| `403 insufficient_scope` | A valid key missing the capability this route requires — or, for Tenants, the key's reach doesn't cover this tenant. |
| `400 validation_failed` | A malformed request body. |
| `429 quota_exceeded` | A reach-restricted key that already hit its tenant-creation limit. |

## The old "Platform API key"

What used to be a separate credential type (`pvk_`, via `X-Provisioning-Key`) is now just the Tenants
capability's optional reach-and-quota setting on a Platform Developer Key — nothing changed underneath, and
any already-issued `pvk_` key keeps working exactly as before. New keys are simply never created that way
from this page anymore.

See [Tenant console → Developer API](../tenant-console/developer-api.html) for the equivalent,
workspace-scoped key type a tenant creates for itself.
