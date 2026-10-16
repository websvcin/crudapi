# Changelog

All notable changes to CrudApi will be documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

---

## [Unreleased]

---

## Since 1.0.0 (current: v1.0.97)

Builds auto-increment as `1.0.x` on every push — this section rolls up the features added since
the first public release rather than listing every individual build.

### Added
- **Admin console rewrite** (`WebSvc.CrudAdmin` 2.0.0): the separate Roles, Policies, Users,
  Provisioning Keys, Issuance Keys, JWT Sessions, Tenant Sessions and API Clients pages are
  replaced by a unified **Access model**; a two-tier **Developer API** key system
  (platform- and workspace-scoped) replaces ad hoc key handling; **LoginLink** adds tenant
  self-service login links as an alternative to password/JWT login; **Managed databases** adds
  policy-gated, script-driven database creation; the old Backup and Migrate-to-MySQL pages are
  replaced by a single **Storage & data** page, which also gained a live, no-restart switch onto a
  freshly-provisioned MySQL store during first-run setup. **Breaking**: the old page routes above
  are gone — see the [docs site](https://websvcin.github.io/crudapi/admin-console/) for where each
  capability moved to.
- **Unregister a tenant database** (`WebSvc.CrudAdmin` 2.0.1): a database can now be removed from
  a workspace's own registry — CORS origins, row policies, rate limits and cached config state go
  with it — without the underlying database ever being touched.
- **Config Visualizer tenant-scoped sessions** (`WebSvc.ConfigVisualizer` 1.1.0): a signed handoff
  can now mint a session locked to one tenant and database, not only platform-admin, enforced on
  every request — not just at login — so one tenant can never reach another's config editor.
- **SqliteAdmin per-file read-only protection** (`WebSvc.SqliteAdmin` 1.2.0): specific database
  files can be marked browse/query/export-only, independent of making the whole browser read-only.
- **Runtime data consolidated** into a single `App_Data` folder — previously scattered across
  separate `Data/`, `RateLimitData/`, `ConfigVisualizerData/`, and `bulk-files/` locations, each
  needing its own Docker volume.
- **Visual config editor** (`/configvisualizer`): tree/field/inspector editor over a database's
  AI-generated dashboard config — edit field labels, types, and relation pickers (including
  fixing relation label fields the generator leaves empty), preview the JSON live, and publish as
  a new git commit. Version history reads real commits, with one-click restore. Reached via a
  signed one-click handoff from the admin console's Databases page, same pattern as the
  rate-limiting admin and SQLite browser.
- **Tenant-facing database registration**: a tenant admin can register and test-connect their own
  databases from the Tenant Portal, not just Global Admins from `/admin` — including the
  platform-admin "Test Connection" button, which previously called a route that didn't exist.
  Duplicate database keys now return a clean `409` instead of a raw SQL error.
- **Stateless API-key auth for the Tenant Portal**: `Authorization: Bearer` support alongside the
  existing cookie login, for server-to-server callers that shouldn't need a browser session.
- **Roles-export endpoint**: `GET /api/v1/{tenant}/admin/roles-export` — lets an external identity
  system (e.g. an SSO console) sync a tenant's global + tenant-specific roles.
- **Rate limiting** (`/ratelimitadmin`): per-credential, per-IP, and global-per-database traffic
  limits, plus tenant-wide aggregate quotas on daily, monthly, or custom periods with auto-reset.
  Configurable per database from its own admin UI, with live usage tracking and a Global Admin /
  Tenant Admin view — separate login, with a one-click signed handoff from the main admin UI.
- **Self-service Tenant Portal** (`/tenant/login`): a tenant admin can manage their own
  databases and API keys without needing Global Admin access.
- **Provisioning keys**: scoped, revocable keys for automating database/tenant setup.
- **Cross-app signed handoff**: one-click, no-re-login navigation between the main admin UI,
  the rate-limiting admin, the SQLite browser, and the config editor.
- **First-run setup wizard** (`/setup`): replaces the old seeded default credential — a fresh
  install now has no default login; the wizard creates the real first admin account.
- Editable per-database tenant ownership from the admin UI.

### Fixed
- Cross-app links (Rate Limits, SQLite browser) now open in a new tab instead of navigating
  away from the admin console.

### Removed
- The seeded `admin` / `admin@123` default credential — see "First-run setup wizard" above.

---

## [1.0.0] — 2026-05-27

### 🎉 First public release

#### Added
- Multi-tenant CRUD API platform with built-in IAM
- Three authentication modes: API Key, Basic Auth, JWT
- Role-based + permission-based + row-level access control
- Per-database CORS allow-list (admin UI)
- Bulk CSV import with FK resolution, enum validation, error CSV
- Admin UI: databases, users, roles, permissions, audit, sessions, settings
- Standalone SQLite database browser (`/sqliteadmin`)
- Beautiful landing page (`/`) and in-app docs (`/help`)
- Live settings (event-driven cache; no restart for most changes)
- Audit logging with retention and archive
- Session limits and password policy
- JWT issuer/audience grace period for safe rotation
- Multi-arch Docker image (amd64 + arm64)
- Non-root container user

#### Engines
- MySQL ✅
- SQL Server 🔜 Planned
- PostgreSQL 🔜 Planned
- SQLite 🔜 Planned

#### Known limitations
- MySQL only for now (other dialects on roadmap)
- MFA settings are placeholder (no provider yet)
- Notify URL change requires restart
- GitHub/AI tokens stored plaintext in DB — **resolved**: stored encrypted at rest via Data
  Protection as of a later build, decrypted only in memory when read

---

## How to upgrade

```bash
docker pull ghcr.io/websvcin/crudapi:latest
docker compose up -d
```

Always back up `App_Data/` before upgrading.
