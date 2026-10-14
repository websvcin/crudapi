---
title: Architecture
parent: Concepts
nav_order: 1
---

# Architecture
{: .no_toc }

How the pieces fit together, and where each one lives.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The shape of the system

CrudApi is a single ASP.NET Core host that combines:

- **The data API** — generic CRUD over whatever MySQL database you register, versioned
  (`/api/v{n}/{tenant}/{database}/...`), covering plain CRUD, bulk import, and database/schema actions
  (introspection, generating a dashboard config, running scripts).
- **The admin console** (`/admin`) — platform-operator UI and API for tenants, databases, access policy,
  platform-wide developer keys, audit, and host settings.
- **The tenant console** (`/{slug}/...` and `/tenant/...`) — each tenant's own self-service UI and API for
  its databases, keys, people, roles, and usage.
- A handful of **companion surfaces** bundled alongside it: a rate-limiting admin UI, a visual config editor,
  and a lightweight SQLite browser for the host's own storage — each reachable from the consoles above, not
  separate products you set up independently.

## Request pipeline, in order

Every request passes through, in this order:

1. **Setup gate** — until the very first admin account exists, everything except `/setup` is blocked.
2. **Tenant/database resolution** — the URL's `{tenant}/{database}` segments are resolved to a real tenant
   and a decrypted connection string before anything else runs.
3. **Authentication** — whichever credential header is present (`X-API-Key`, `Authorization: Basic`,
   `Authorization: Bearer`, or a signed-in console session) is validated against that tenant and database.
4. **Rate limiting** — if a limit applies to this credential/database pair, it's enforced here, before the
   request reaches any business logic.
5. **Authorization** — the resolved identity's role is checked against the specific `{resource}.{action}`
   this request needs.
6. **Row policy** — if the matched role carries a row policy, it's attached here and merged into the query
   every controller action below actually runs.
7. The matching **controller action** runs: plain CRUD, a bulk operation, or a database/schema action.

Every step above writes to the audit log on both success and refusal, so a denied request is just as visible
after the fact as an accepted one.

## Multi-engine by design, MySQL today

The CRUD layer resolves its SQL dialect per request rather than assuming one engine throughout — so adding a
second database engine is a matter of registering a new dialect, not restructuring the controllers. MySQL is
the only engine actually bundled today; see the admin console's **Engine roadmap** page for what's planned.

## Where data actually lives

- **Your tenant data** stays exactly where it already is — in whichever MySQL database(s) you register.
  CrudApi never takes ownership of that schema; it only reads and writes it through the credential and role
  you've configured.
- **Everything CrudApi itself needs to remember** — tenants, databases (connection details, encrypted),
  accounts, roles, permissions, row policies, audit history, host settings — lives in the platform's own
  store, set up once during the [setup wizard](../deployment/setup-wizard.html) (SQLite by default, or MySQL
  if you choose that at setup).

## Companion surfaces

- **Rate-limiting admin** — per-database, per-credential request limits, with its own global and per-tenant
  views. Reached from the consoles via a signed single-sign-on handoff, not a separate login.
- **Config visualizer** — a visual tree/field editor over the dashboard configuration a database's
  **Generate Config** action produces, reached the same signed-handoff way from a database's own page.
- **Audit log** — every authenticated action, success or refusal, is recorded with who, what, when, and the
  outcome, exportable from the admin console.

See [Admin console](../admin-console/) and [Tenant console](../tenant-console/) for a page-by-page tour of
every screen these pieces expose, and [Deployment](../deployment/) for how to actually stand this up.
