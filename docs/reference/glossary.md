---
title: Glossary
parent: Reference
nav_order: 2
---

# Glossary
{: .no_toc }

Every term used across this documentation, in one place.
{: .fs-6 .fw-300 }

---

**API client key** — A data-API credential (`drk_` prefix, sent as `X-API-Key`) bound to specific
database(s) and a role. See [Concepts → Authentication & credentials](../concepts/authentication-and-credentials.html).

**Audit log** — The platform-wide record of every authenticated action, success or refusal. See
[Admin console → Audit logs](../admin-console/audit-logs.html); a tenant's own equivalent is
[Tenant console → Activity](../tenant-console/activity.html).

**Basic Auth login** — A username/password data-API credential for tools that can't send a custom header.

**Bootstrap (`bootstrap.db`)** — A small file remembering where the platform's own data actually lives,
locked together with the `keys` folder. See [Admin console → Storage & data](../admin-console/storage-databases.html).

**Capability** — A named bundle of routes a [Developer Key](#developer-key) can be scoped to (e.g.
"Tenants," "Databases," "Access model").

**Console key** — A credential that signs one person into a tenant's console only — no database access,
no role. See [Admin console → Tenants](../admin-console/tenants.html#console-key).

**Data Protection keys** — Cryptographic keys (in the `keys/` folder) that decrypt everything the platform
stores encrypted — database passwords, GitHub tokens, AI keys, and more. Losing them means losing the
ability to use that encrypted data, even though the data itself is intact.

**Database admin** — A built-in role: Editor's permissions plus generating dashboard configs and running
scripts. See [Concepts → Authentication & credentials](../concepts/authentication-and-credentials.html#roles).

**Developer Key** — A capability-scoped credential for automating a *console* (not the data API) — either
a **Platform Developer Key** (`pdk_`, platform-wide) or a **Workspace Developer Key** (`wdk_`, one
tenant). See [Admin console → Developer API](../admin-console/developer-api.html).

**Editor** — A built-in role: full CRUD (create, read, update, delete) on every table it reaches.

**Engine** — The underlying SQL database technology (MySQL today) a registered database runs on. See
[Admin console → Engine roadmap](../admin-console/engine-roadmap.html).

**Generate Config** — The action that introspects a database's schema and produces the JSON configuration
driving an auto-built dashboard frontend. See [Features → Generate Config & dashboards](../features/generate-config-dashboards.html).

**Guardrail** — A platform-wide or per-tenant rule about *what a tenant is allowed to hand out* to its own
keys and people — distinct from a role itself. See [Admin console → Access model](../admin-console/access-model.html#tab-3--tenant-guardrails).

**Issuance key** — A credential your own backend presents to CrudApi to mint a short-lived JWT for one of
your end-users, rather than being used directly on data requests itself.

**LoginLink** — A separate sign-in product CrudApi integrates with, at four distinct points (platform team
sign-in, tenant team sign-in, tenant end-user sign-in, and tenant auto-provisioning). See
[Features → LoginLink integration](../features/loginlink-integration.html).

**Managed database** — A database CrudAdmin creates on a tenant's behalf (via a Plesk, cPanel, or MySQL
provisioning account) rather than the tenant registering one it already has. See
[Features → Managed databases](../features/managed-databases.html).

**Owner** — The most powerful built-in role: `*.*`, every action on every table plus every database action,
including loading starter data.

**Permission** — A single `{resource}.{action}` string (e.g. `widgets.read`, `database.script`) describing
one thing a role may do. See [API reference → Permissions model](../api-reference/permissions.html).

**Platform Developer Key** — see **Developer Key**.

**Reader** — A built-in role: read-only.

**Recovery kit** — A downloadable bundle of `bootstrap.db` plus the `keys` folder — both required to open
the platform's own encrypted data; losing either without this kit is unrecoverable. See
[Admin console → Storage & data](../admin-console/storage-databases.html#recovery-kit).

**Role** — A named bundle of permissions. Five are built in (Reader, Writer, Editor, Database admin,
Owner); a tenant may build its own custom roles if its platform administrator allows it.

**Row policy** — A restriction narrowing a role to specific *rows* of a table (not just which tables), e.g.
"only rows where `region` is APAC." See [Features → Row policies](../features/row-policies.html).

**Scope** (of a grant) — *Where* a role applies: one database, all of a tenant's databases, or the whole
platform. Distinct from the role itself, which says *what* is allowed.

**Signing key** — The cryptographic key that signs JWTs issued by an issuance key or LoginLink. See
[Admin console → Signing keys](../admin-console/signing-keys.html).

**Support view** — A platform administrator's read-only, time-limited, logged view into a tenant's own
console. See [Admin console → Tenants](../admin-console/tenants.html#support-view).

**Tenant** — One customer/workspace; its data, keys, roles, and people are completely isolated from every
other tenant. See [Concepts → Multi-tenancy](../concepts/multi-tenancy.html).

**Tier** — A numeric sensitivity ranking (1–4) on a permission, deciding whether a tenant may hold it
freely or needs platform approval. See [Admin console → Access model](../admin-console/access-model.html#tab-1--permissions-and-tiers).

**Wide grant** — A credential reaching more than one database of a single tenant (all of a tenant's
databases, or the whole platform) — the review list on [Access model](../admin-console/access-model.html#tab-4--wide-grants).

**Workspace Developer Key** — see **Developer Key**.

**Writer** — A built-in role: read, add, and change rows — no delete.
