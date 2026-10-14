---
title: Authentication & credentials
parent: Concepts
nav_order: 3
---

# Authentication & credentials
{: .no_toc }

Every credential type CrudApi supports, what each is for, and exactly how it's checked.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Two different APIs, two different credential families

CrudApi actually has **two separate authenticated surfaces**, and this is the single most important thing
to understand before picking a credential type:

1. **The data API** (`/api/v1/{tenant}/{database}/...`) — your application's own CRUD traffic. Secured by
   one of four credential types, below.
2. **The management API** (`/api/admin/...` and `/api/tenant/...`) — configuring the platform or a
   workspace itself (creating tenants, databases, roles, keys…). Secured by a completely different
   credential family: [Platform Developer Keys and Workspace Developer Keys](#developer-keys-the-management-api).

A key that can read and write your `orders` table has **no ability whatsoever** to create a new tenant or
mint another key — those are different permission systems entirely, by design.

## Data-API credentials

| Credential | Prefix | Header | Lifetime | Typical use |
|---|---|---|---|---|
| API client key | `drk_` | `X-API-Key: drk_...` | Until revoked (optional expiry) | Server-to-server — the common case |
| Basic Auth login | — | `Authorization: Basic <base64(user:pass)>` | Until revoked | Simple server-to-server, legacy tooling |
| Issuance-key JWT | — (a plain JWT) | `Authorization: Bearer <token>` | Short-lived, you choose | Minting a token per end-user from your own backend |
| LoginLink token | — (a plain JWT) | `Authorization: Bearer <token>` | Short-lived, LoginLink-issued | A real person signed in through your LoginLink-connected app |

A frequent mistake: **`Authorization: Bearer` is not a universal "put any credential here" header.** It's
specifically for JWTs (issuance-key tokens and LoginLink tokens). An API client key always goes in
`X-API-Key`, never `Authorization: Bearer` — sending it as a Bearer token gets you an authentication
failure, not a degraded-but-working request.

### API client key

Created from the tenant workspace's **Keys** tab (or by a platform administrator, on a tenant's behalf, from
the admin side). Bound to one or more specific databases and a role. Shown once at creation — if it's lost,
revoke it and create a new one; CrudApi never stores the raw value, only a hash.

### Basic Auth login

A username/password pair, created the same way as an API client key, for tools that already speak HTTP
Basic Auth and can't easily add a custom header.

### Issuance-key JWT

An **issuance key** is itself a credential, but not one you put directly on requests — you present it to
CrudApi's token endpoint to mint a short-lived JWT for a specific subject (typically one of your own
end-users), and *that* JWT is what goes on the actual data request. This is the pattern for "my backend
signs its users into their own scoped view of the data," without creating a standing API key per end-user.

### LoginLink token

If a tenant has connected its workspace to a [LoginLink](../features/loginlink-integration.html)
organisation, a person who signs in through that connection gets a token CrudApi accepts directly — no
separate key-management step on CrudApi's side at all.

## Roles

A role is what a credential is allowed to *do* — read, add, change, delete — wherever it reaches. Which
*database(s)* it reaches is chosen separately, when the key is created; the same role can be handed to one
key limited to a single database and to another that covers every database a tenant owns.

| Role | Allows |
|---|---|
| **Reader** | Read only. |
| **Writer** | Read, add, and change rows. Cannot delete. |
| **Editor** | Full CRUD (create, read, update, delete). The recommended default for a key that needs real write access. |
| **Database admin** | Everything Editor does, plus generating the dashboard configuration and running schema-changing scripts. Does *not* include loading starter data. |
| **Owner** | Everything, including loading starter data. Total system access on the database(s) it applies to. |

Database admin and Owner are more powerful and sit behind an extra guardrail: a platform administrator has
to explicitly allow a tenant to hand either of them out before that tenant can create a key with one. See
[Admin console → Access model](../admin-console/access-model.html).

A tenant can also build its **own custom roles**, composed only from safe, non-destructive actions — never
including configuration generation, schema scripts, or "everything" — if a platform administrator has turned
that on for them. See [Tenant console → Roles](../tenant-console/roles.html).

## Row policies: restricting *which rows*, not just which tables

A role controls *actions* on *tables*. A **row policy** goes one level deeper: it restricts a role to only
the rows of a table that match a condition — for example, "the Support role can only see orders where
`region` is APAC." Row policies are enforced at the same point every CRUD request already validates
permissions, so they can't be bypassed by calling a different endpoint. Full detail:
[Features → Row policies](../features/row-policies.html).

## Developer keys: the management API

Separate from everything above, **Developer Keys** are what automate the *management* surfaces — creating
tenants, databases, roles, other keys, and so on — rather than reading or writing your actual data.

| | Platform Developer Key | Workspace Developer Key |
|---|---|---|
| Prefix | `pdk_` | `wdk_` |
| Scope | The whole platform, across every tenant | One specific tenant's workspace |
| Header | `Authorization: Bearer pdk_...` | `Authorization: Bearer wdk_...` |
| Grants access by | **Capability** (e.g. "Tenants," "Databases," "Access model") | **Capability** (e.g. "Users," "Roles," "Databases") |
| Created from | Admin console → Developer API | Tenant console → Developer API |

Each capability maps to a fixed, documented set of real routes — the full mapping is shown directly on each
Developer API screen, and is enforced centrally rather than scattered across each individual endpoint, so it
can't silently drift out of sync with what the key actually reaches.

Both key types deliberately **cannot mint more keys of their own type** (a Platform Developer Key can't
create another Platform Developer Key; a Workspace Developer Key can't create another Workspace Developer
Key) — a guardrail against a leaked key escalating itself. A Platform Developer Key *can* mint a
Workspace Developer Key for a specific tenant in one call (via the `SupportView` capability), without that
self-escalation risk, since it's minting a *different*, narrower-scoped credential, not a copy of itself.

See [Admin console → Developer API](../admin-console/developer-api.html) and
[Tenant console → Developer API](../tenant-console/developer-api.html) for the full capability lists.
