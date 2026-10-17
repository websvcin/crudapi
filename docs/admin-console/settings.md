---
title: Settings
parent: Admin console
nav_order: 8
---

# Settings
{: .no_toc }

Two separate screens, both under "Settings" in the sidebar: **Security Settings** (`/admin/settings`) and
**Host Settings** (`/admin/hostsettings`).
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Security Settings

Platform-wide security policy, grouped into six cards. Every setting here is **event-driven**: saving
invalidates the in-memory cache immediately, and the next request sees the new value — no restart needed,
unlike several Host Settings below.

| Group | Settings |
|---|---|
| **JWT Configuration** | Issuer, Audience, Token Expiry (min), Signing Algorithm (HS256/384/512), Require HTTPS, Key Retention (hours). |
| **API Key Policy** | Default Expiry (days; `0` = never expires by default), Max Keys Per Tenant (`0` = unlimited). |
| **Audit Configuration** | Retention (days), Log Read Operations, Archive Before Purge, Archive Directory, Purge Hour (UTC), Purge Batch Size. |
| **Session Management** | Max Concurrent Sessions (`0` = unlimited; beyond the limit, new logins are rejected with `429 session_limit_reached`). |
| **Multi-Factor Auth** | Enabled, Provider — shown as **📅 Planned**, not yet enforced in code. |
| **Password Policy** | Min Length, Require Uppercase, Require Number, Require Symbol — enforced on create/change; existing passwords aren't retroactively checked. |

Every setting is tagged **✓ Live** (enforced the moment you save) or **📅 Planned** (a placeholder for a
future release, not yet enforced) — MFA is currently the only planned group. A few settings carry extra
behavior worth knowing:

- **JWT Issuer / Audience** — changing either starts a grace period: tokens signed under the old value keep
  working for `jwt.issuer_grace_days` days, so you don't invalidate every live token the instant you save.
- **Password requirements** — apply going forward only; nobody is logged out or forced to reset for not
  meeting a newly-tightened rule retroactively.

## Host Settings

Settings specific to this host's integrations — bulk import limits, outbound notifications, GitHub
publishing, the AI provider that powers config generation, and the shared LoginLink server address. Every
field has an inline guide (what it is, why it matters, how to fill it in, where to get the value) — click
**Open all guides** to expand every one at once.

{: .warning }
> Most settings here apply immediately via the same live cache as Security Settings. **Notify, GitHub, and
> AI settings are the exception — they need the app restarted to take effect.** Saving a new GitHub token,
> for instance, updates what's stored, but the running process keeps using whatever token it loaded at
> startup until it's restarted. This is the single most common cause of "I updated the token but it still
> fails" reports — restart after changing any of these three groups.

### Decided during setup

A read-only summary of choices made in the one-time [setup wizard](../deployment/setup-wizard.html): whether
setup completed, where CrudAdmin keeps its own data, whether automatic database provisioning is configured,
and how many admins exist — each with a link to where it can actually be changed (mostly
[Storage & data](storage-databases.html)).

### Provisioning account

The credential CrudAdmin uses to create real databases on a tenant's behalf (see
[Tenants → Managed databases](tenants.html#managed-databases)). Three kinds:

| Kind | What it needs |
|---|---|
| **A MySQL server you run** | One admin login that can `CREATE`, `CREATE USER`, `GRANT` — used to create a database and a scoped-down user per tenant. |
| **Plesk server** | Panel host/IP, port (default 8443), the target subscription's webspace id, and either your Plesk admin login or a secret key, plus whether to verify its TLS certificate. One subscription per tenant. |
| **cPanel account** | Panel host/IP, port (default 2083), account username, and an API token (created in cPanel under Security → Manage API Tokens, shown once). |

A **Test** action validates the credential live before saving; nothing is stored until the test passes.

### Bulk import limits

- **Realtime row threshold** — uploads at or under this row count process while the user waits; larger
  ones queue in the background (default 10 — the same setting [Bulk endpoints](../api-reference/bulk-endpoints.html#execution-mode-realtime-vs-background)' `Auto` execution mode reads).
- **Max file size (MB)** — largest upload allowed (default 10; can't exceed any smaller limit your own
  web server or reverse proxy already imposes).
- **Allowed extensions** — comma-separated, with the dot (e.g. `.csv,.tsv`; default `.csv`).
- **Google Drive key file / credentials JSON** — stored and ready, but **not yet read** by bulk import in
  this build; see [Reference → Known limitations](../reference/known-issues.html).

### Notify

- **Enabled** — turns outbound notifications on/off.
- **Base URL** — the notification service's address, no trailing path. **Requires a restart.**

### GitHub

- **Owner** / **Repo** — the GitHub repository [Generate Config](../features/generate-config-dashboards.html)
  publishes to (just the names, no slashes).
- **Personal access token** — a GitHub PAT with push rights to that repo. Stored encrypted; never shown
  again after saving, only who last changed it and when.
- **Config path prefix** — the folder prefix inside the repo configs are written under (default
  `databases`).

{: .warning }
> **Requires a restart to take effect.** If a config push fails with `Unauthorized — Bad credentials`
> right after saving a new token here, this is almost always why — restart the app, then retry. See the
> diagnostic endpoint `GET /api/admin/diagnostics/config-snapshot`, which compares what's loaded into the
> running process against what Host Settings holds live, to confirm which one is stale before you spend
> time re-checking the token itself.

### AI

- **Provider** — OpenAI, Gemini, or AzureOpenAI.
- **URL** — leave blank for the provider's normal address; for Azure OpenAI, your resource endpoint.
- **Model** — e.g. `gpt-4o-mini`.
- **API key** — stored encrypted, same as the GitHub token. **Requires a restart.**

This is the model [Generate Config](../features/generate-config-dashboards.html) calls to produce table
groupings, descriptions, and icons — see [Known limitations](../reference/known-issues.html) for the cost
and reliability implications of that dependency.

### LoginLink server

- **Issuer** — your LoginLink deployment's address, set once here so two independent features — new-tenant
  provisioning (on the [Tenants](tenants.html) page) and admin sign-in (on the
  [Admin Users](admins.html) page) — can never silently point at two different servers.
- **CrudApi's own API identifier** — what an app (like a tenant's own separate product, calling the data
  API as its users) asks LoginLink for a token addressed to, instead of needing to be the one specific app
  each workspace already trusts. Left blank, this defaults to the host's own configured public address
  (`Public:BaseUrl`) — a fixed value, the same one CrudAdmin's own tokens use — shown on this page as
  "In effect right now: …" so it's never a silent guess. If `Public:BaseUrl` *also* isn't set, the page
  says so with a warning: without it, the identifier falls back to whatever address served each individual
  request, which can differ behind a reverse proxy and cause correctly-addressed tokens to be refused
  intermittently — set `Public:BaseUrl` for a stable value. Editable either way, and changing it
  re-registers the new value in every workspace already connected to LoginLink.
  Full detail: [Features → LoginLink integration](../features/loginlink-integration.html#crudapi-as-a-registered-api).
