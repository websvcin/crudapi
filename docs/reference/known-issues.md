---
title: Known limitations
parent: Reference
nav_order: 1
---

# Known limitations
{: .no_toc }

A consolidated list of genuine, current limitations — not a change log, and not a list of resolved
historical bugs. Each links to the page with full context.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Data API

- **MySQL is the only supported engine for tenant data today.** The architecture resolves its SQL dialect
  per request, so adding another engine doesn't require restructuring — but none beyond MySQL is bundled
  yet. See [Concepts → Architecture](../concepts/architecture.html#multi-engine-by-design-mysql-today) and
  [Admin console → Engine roadmap](../admin-console/engine-roadmap.html).
- **A background bulk-import job's result isn't persisted anywhere queryable.** A queued import (row count
  over the realtime threshold) returns a `jobId`, but there's currently no endpoint to poll that job's
  final outcome — you'd confirm completion by checking the data itself. See
  [API reference → Bulk endpoints](../api-reference/bulk-endpoints.html#execution-mode-realtime-vs-background).
- **A multi-statement script is not wrapped in one transaction.** If a script fails partway through,
  earlier statements stay applied. See
  [API reference → Database & schema endpoints](../api-reference/database-endpoints.html#run-a-script--post-databasescript).
- **Bulk import's Google Drive source is stored but not yet read.** The credential fields exist in Host
  Settings, ready for when this ships. See
  [Admin console → Settings → Host Settings](../admin-console/settings.html#host-settings).

## Generate Config

- **No built-in cost control.** Each call is a real, metered AI request, and nothing today stops repeated
  or automated regeneration from running up real cost. See
  [Features → Generate Config & dashboards](../features/generate-config-dashboards.html).
- **GitHub is the only publishing backend**, with no fallback — if your GitHub integration is unreachable,
  publishing fails even though the config itself still generates successfully. See
  [Admin console → Settings → GitHub](../admin-console/settings.html#github).
- **Generated `accessRoles` are placeholders (`"root"`), not your platform's real role names.** A frontend
  that compares a signed-in person's actual role against these fields won't get a meaningful result yet.
  See [Features → Generate Config & dashboards](../features/generate-config-dashboards.html).
- **A saved GitHub token or AI key needs a restart to take effect.** Changing either in Host Settings
  updates what's stored, but the running process keeps using the value it loaded at boot until restarted.
  See [Admin console → Settings → Host Settings](../admin-console/settings.html#host-settings).

## LoginLink

- **"Check it still works" can report "no credential stored" even when one is.** It also requires a
  LoginLink app object to exist, and conflates that separate missing-prerequisite into the same message.
  See [Features → LoginLink integration](../features/loginlink-integration.html#known-rough-edge).

## Platform / operations

- **No automated test suite for this repository's own code.** The private packages this platform depends
  on each carry substantial test suites in their own repositories; this repo's own thin-host code (local
  controllers, builders, bridge classes) currently has none. See
  [Deployment → CI/CD](../deployment/azure-pipelines.html#tests).
- **Reverse-proxy trust defaults to "trust anything" if left unconfigured.** Set
  `ReverseProxy__TrustedProxyIPs` explicitly in every real deployment — see
  [Deployment → Environment variables](../deployment/environment-variables.html#reverse-proxy-trust).
