---
title: Docker
parent: Deployment
nav_order: 1
---

# Docker
{: .no_toc }

The recommended way to run CrudApi — a ready-to-run image is published on every push to `main`.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Pull the published image

No need to clone or build anything:

```bash
docker pull ghcr.io/websvcin/crudapi:latest
# or
docker pull websvcin/crudapi:latest
```

Images are published to both GHCR and Docker Hub on every push to `main`, tagged `latest`, the current
`<major>.<minor>` series, and (on a tagged release) the exact release version — see
[CI/CD](azure-pipelines.html).

## The one thing you must not skip: a persistent volume

{: .warning }
> **One host directory must be mounted, or you will lose all data the next time the container restarts.**
> The image writes everything — the admin registry (every tenant, database, and credential), the Data
> Protection keys that encrypt stored passwords/connection strings, setup state, background jobs,
> rate-limit and Config Editor data, and uploaded files — to `/app/App_Data`. A container's writable layer
> is discarded every time it's recreated: a redeploy, a host reboot, pulling a new image version. This has
> already happened to a real deployment before this warning existed.

```bash
mkdir -p /path/on/host/crudapi-data
# the image runs as non-root uid 1000 — the host directory must be writable by it
chown -R 1000:1000 /path/on/host/crudapi-data

docker run -d \
  --name crudapi \
  --restart unless-stopped \
  -p 8081:80 \
  -v /path/on/host/crudapi-data:/app/App_Data \
  ghcr.io/websvcin/crudapi:latest
```

The image already listens on port 80 by default — no `ASPNETCORE_HTTP_PORTS` variable needed unless you
want a different port.

### Upgrading from an older container

If your existing container already mounts the **old** separate `/app/Data` and `/app/bulk-files` paths,
leave those mounts as they are — a mounted folder can't be renamed, so the app keeps using it right where
it is, and says so in the startup log. To consolidate onto the single `App_Data` folder, stop the
container and mount the same host directory at `/app/App_Data` instead (move the old `bulk-files`
contents into a `bulk-files` subfolder inside it first).

### No shell access on your hosting panel?

Some managed control panels don't give you `chown`. Open the host directory to world-write instead (`rwx`
for "Other") so the non-root container user can still write to it — less precise than `chown`, but works
from a file manager alone.

## What's inside `App_Data`

| Path | Holds |
|---|---|
| `keys/` | Data Protection keys — these decrypt everything else that's encrypted. |
| `bootstrap.db` | The password to the admin store, locked together with `keys/`. |
| `system.db` | Every tenant, database, credential, role, setting (SQLite mode). |
| `hangfire/` | Background job state (bulk import queue, etc.). |
| `ratelimit/` | Rate-limiting configuration and usage history. |
| `configvisualizer/` | Config Visualizer session state. |
| `bulk-files/` | Uploaded files from bulk import. |

The whole folder can be relocated with `Storage__DataRoot`.

## First run

Visit `http://your-host:8081/setup` — there's no default account; the [setup wizard](setup-wizard.html)
creates the real first admin login. The real Swagger UI is at `/docs/index.html` — not
`/swagger/index.html`, which only exists in local development.

## Worked example: deploying via Plesk

1. **Prepare the codebase** — clone the repo (`https://dev.azure.com/web-svc/CrudApi/_git/CrudApi` for
   contributors with access, or just pull the published image as above for a straightforward deployment).
2. **Build the image**: `docker build -t crudapi:v1 .`, then `docker save -o crudapi-v1.tar crudapi:v1` and
   download the tarball if building outside Plesk directly.
3. **Upload** — Plesk → Docker → Images → Upload an image. Remove any existing image with the same
   version tag first.
4. **Run the container** — map container port `80` to whatever external port you want (e.g. `8081`),
   enable auto-start on reboot, and — **required, do not skip** — add a persistent volume:
   `host:/var/lib/crudapi/data` → `container:/app/App_Data`. Without this, the folder lives only in the
   container's writable layer and is wiped on the next recreate.
5. **Set your tenant database connection** (if testing against one already reachable from the container),
   e.g. `ASPNETCORE_ConnectionStrings__Default1=Server=172.17.0.1;Port=3306;Database=yourdb;Uid=...;Pwd=...;`.
6. **Configure proxy rules** — Plesk → Subscriptions → your domain → Docker Proxy Rules, with WebSocket
   support enabled.
7. **Verify** — Swagger should be reachable at `https://your-domain/docs/index.html`.

See [Environment variables](environment-variables.html) for the full list of what can be configured this
way, and [Setup wizard](setup-wizard.html) for what happens the very first time you open the app.
