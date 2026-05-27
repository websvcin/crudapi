# Changelog

All notable changes to CrudApi will be documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

---

## [Unreleased]

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
- GitHub/AI tokens stored plaintext in DB (documented; use OS encryption)

---

## How to upgrade

```bash
docker pull ghcr.io/web-svc/crudapi:latest
docker compose up -d
```

Always back up `Data/` before upgrading.
