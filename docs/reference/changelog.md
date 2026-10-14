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
