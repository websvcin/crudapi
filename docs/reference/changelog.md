---
title: Changelog
parent: Reference
nav_order: 3
---

# Changelog
{: .no_toc }

Notable, user-facing milestones. For the complete commit-by-commit history, see the repository itself.
{: .fs-6 .fw-300 }

---

## Recent

- **A workspace's own role now applies from a LoginLink sign-in regardless of the "own roles" setting**
  (`WebSvc.CrudAdmin` 2.1.7): the fix just below (2.1.6) introduced a regression — a role created directly for a
  tenant by a platform admin (bypassing tenant self-service) stopped being recognized from a LoginLink sign-in
  for any workspace that hasn't explicitly turned "own roles" on, the default for a new workspace. That setting
  governs *building* a role through tenant self-service; it never governed *using* one that already exists.
  Fixed without touching tenant self-service role assignment, which is unaffected.
- **A LoginLink sign-in now gets its built-in role's permissions applied** (`WebSvc.CrudAdmin` 2.1.6): a person
  signing in through LoginLink with a built-in role (Reader, Writer, Editor, Database admin) never actually got
  that role's permissions — the lookup only ever matched a workspace's own custom roles, so every real data call
  through a LoginLink-authenticated app failed with a permissions error. Resolved the same tested way the admin
  console and key-assignment paths already do; a LoginLink token still never grants Owner. See
  [Features → LoginLink integration](../features/loginlink-integration.html).
- **A config generated through the data API is now recorded the same way an admin-generated one is:**
  generating a config via `GET .../database/generate-config` (the route a tenant's own app calls, not the
  admin console's own button) pushed the file to GitHub correctly but never updated the database's own row —
  so `config_file_url` stayed empty and a caller reading that row couldn't find the file it had just
  published. Both paths now record the same way. No caller-side change needed; the field simply starts
  showing up.
- **Generate Config was broken outright for a short window** after the no-restart fix below shipped: a
  `DependencyInjectionException` ("No service for type 'AI.Providers.OpenAIProvider' has been registered")
  on every call. A regression in that same fix — removing the only thing that fed the AI provider type into
  app configuration at startup also broke a different package's own internal registration that depended on
  it. Fixed by building the AI provider directly instead of through that indirection; pinned with a new
  regression test so this exact failure mode can't silently return.
- **The Users page shows a person's email, not just their raw subject id** (`WebSvc.CrudAdmin` 2.1.5): a
  LoginLink-authenticated person with no display name set used to show only their opaque subject
  id forever — their email is already resolved on every LoginLink data-API call, it just never reached the
  record this page reads from. Threaded through the existing usage pipeline instead of a new lookup: an
  existing person's row is backfilled the next time they make a request, never overwriting a display name or
  email already set. See [Tenant console → Users](../tenant-console/users.html#the-users-table).
- **GitHub, AI, and Notify Host Settings no longer require a restart:** these three groups used to be read
  from CrudAdmin's database exactly once, at boot, into a frozen snapshot — changing a GitHub token, the AI
  provider/model, or Notify's base URL silently kept using the old value until the process restarted. They're
  now read fresh from Host Settings on every use, the same live behavior every other setting on this host
  already had. Also fixed in the same pass: leaving the AI URL field blank previously fell back to a built-in
  default that didn't match the request CrudApi actually sends, failing every Generate Config call outright —
  it now defaults to the correct endpoint. The one remaining exception: the separate Config Editor
  (WebSvc.ConfigVisualizer) still reads its own copy of the GitHub config-path-prefix once at startup. See
  [Admin console → Settings](../admin-console/settings.html#host-settings).
- **Auto-approve LoginLink tenant admins into the console** (`WebSvc.CrudAdmin`, pending release): a tenant's
  own console previously turned away every new LoginLink sign-in until an already-linked person approved them
  from Access → People — real friction for someone who'd already proven they're an admin of the same connected
  LoginLink organisation. "Who may sign in" is now a 3-way choice: the existing strict (approve everyone) and
  open (approve no one) modes, plus a new middle option that auto-approves any sign-in whose LoginLink account
  is a `tenant_admin` in that organisation. See
  [Features → LoginLink integration](../features/loginlink-integration.html#who-may-sign-in-to-a-tenants-own-console-point-2).
- **Platform address becomes a Host Setting** (`WebSvc.CrudAdmin` 2.1.3): CrudAdmin's own public address —
  behind its JWT issuer/audience and the LoginLink API identifier — no longer has to be set via the
  `Public:BaseUrl` environment variable. It's now a live Host Setting (editable with no restart) also captured
  during first-run setup; an existing deployment that already sets the env var needs no action, since the
  setting simply takes precedence when present. See
  [Admin console → Settings](../admin-console/settings.html#platform-address).
- **Registered-API identifier and registration-failure reporting fixed** (`WebSvc.CrudAdmin` 2.1.2): the
  identifier CrudApi registers itself under in LoginLink now reliably matches the one it validates
  against — it prefers the deployment's own `Public:BaseUrl`, rather than falling back to whichever
  address served an individual request, which could differ behind a reverse proxy and refuse correctly
  addressed tokens intermittently. A failed "register CrudApi as an API" attempt is also no longer silent:
  the reason is kept and shown on the Tenant console's Access page (and in `GET /api/tenant/loginlink/data`'s
  `apiRegisterError`) until a later attempt succeeds. See
  [Features → LoginLink integration](../features/loginlink-integration.html#crudapi-as-a-registered-api).
- **CrudApi as a registered API in LoginLink** (`WebSvc.CrudAdmin` 2.1.0–2.1.1): a tenant's own separate
  app (not just the one app CrudAdmin itself registers) can now call the data API as its signed-in users —
  LoginLink grants it access to CrudApi as an API (a resource server), and it requests a token addressed to
  CrudApi directly rather than needing to be CrudAdmin's own trusted app. A workspace can now only ever
  connect to its own LoginLink organisation — never the platform's top-level one, and never one another
  workspace already uses. See [Features → LoginLink integration](../features/loginlink-integration.html#crudapi-as-a-registered-api).
- **Admin console rewrite** (`WebSvc.CrudAdmin` 2.0.0): the separate Roles, Policies, Users, Provisioning
  Keys, Issuance Keys, JWT Sessions, Tenant Sessions and API Clients pages are replaced by a unified
  [Access model](../admin-console/access-model.html); a two-tier [Developer API](../admin-console/developer-api.html)
  key system (platform- and workspace-scoped) replaces ad hoc key handling; [LoginLink](../features/loginlink-integration.html)
  adds tenant self-service login links as an alternative to password/JWT login; [Managed databases](../features/managed-databases.html)
  adds policy-gated, script-driven database creation; the old Backup and Migrate-to-MySQL pages are
  replaced by a single [Storage & data](../admin-console/storage-databases.html) page. Breaking: those
  old page routes are gone — see [Admin console](../admin-console/index.html) for where each capability
  moved to.
- **Unregister a tenant database** (`WebSvc.CrudAdmin` 2.0.1): a database can now be removed from a
  workspace's own registry — CORS origins, row policies, rate limits and cached config state go with it —
  without the underlying database ever being touched. See
  [Tenant console → Databases](../tenant-console/databases.html#actions-menu).
- **Config Visualizer tenant-scoped sessions** (`WebSvc.ConfigVisualizer` 1.1.0): a signed handoff can now
  mint a session locked to one tenant and database, not only platform-admin. The scope is checked on every
  request, not just at login, so one tenant can never reach another's config editor — not even by editing
  the URL directly. See [Features → Generate Config & dashboards](../features/generate-config-dashboards.html#the-config-visualizer).
- **SqliteAdmin per-file read-only protection** (`WebSvc.SqliteAdmin` 1.2.0): specific database files
  (by name or full path) can be marked browse/query/export-only, independent of making the whole SQLite
  browser read-only.
- **Generate Config** now reports publishing status honestly — a failed GitHub push no longer looks
  identical to a full success — and gained an admin-triggered route plus a configurable path. See
  [Features → Generate Config & dashboards](../features/generate-config-dashboards.html).
- **Config Visualizer** shipped — a visual editor over a generated dashboard config, including a "Modern"
  non-technical editing mode. See
  [Features → Generate Config & dashboards](../features/generate-config-dashboards.html#the-config-visualizer).
- **Roles export endpoint** added, feeding LoginLink's role-sync connector and any other integration that
  needs this platform's real role names externally.
- **Rate limiting** (`WebSvc.RateLimitAdmin`) wired in as a companion surface, reached by signed handoff
  from a database's own Actions menu. See [Features → Rate limiting](../features/rate-limiting.html).
- **Runtime data consolidated** into a single `App_Data` folder — previously scattered across separate
  `Data/`, `RateLimitData/`, `ConfigVisualizerData/`, and `bulk-files/` locations, each needing its own
  Docker volume. See [Deployment → Docker](../deployment/docker.html#whats-inside-app_data).
- **Database-scoped routing**: the data API's address changed from `/api/v1/{db_key}/...` to
  `/api/v1/{tenant}/{db}/...`, since a database key only needs to be unique within its own tenant, not
  globally. A breaking change to v1 with no parallel v2 — every caller needed the added tenant segment.
- **The first-run setup wizard** shipped, gating the entire app behind creating a real first admin account.
  See [Deployment → Setup wizard](../deployment/setup-wizard.html).
- **Credential-exposure hardening**: a committed credentials file was removed from the working tree and
  `.gitignore`-blocked going forward; forwarded-header trust became configurable
  (`ReverseProxy__TrustedProxyIPs`) instead of unconditionally trusting any proxy.

## Earlier

- Generic Docker Hub/GHCR deployment documented, alongside the original Plesk-specific walkthrough.
- Persistent-volume requirements for the data directory first documented, after a real deployment lost
  data from a missing volume mount.
- Production Swagger UI fixed to fetch its spec from the correct path (`/docs`, not `/swagger`).

---

For specific open limitations rather than historical changes, see
[Reference → Known limitations](known-issues.html).
