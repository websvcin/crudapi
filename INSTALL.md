# Installation

Three ways to install CrudApi. Pick what fits your environment.

---

## 🐳 Option 1 — Docker (recommended)

### Single command

```bash
mkdir -p $(pwd)/data
chown -R 1000:1000 $(pwd)/data   # the image runs as non-root uid 1000

docker run -d -p 8080:80 \
  -v $(pwd)/data:/app/App_Data \
  --name crudapi \
  ghcr.io/websvcin/crudapi:latest
```

### With docker-compose (better for production)

```bash
curl -O https://raw.githubusercontent.com/websvcin/crudapi/main/docker-compose.yml
curl -O https://raw.githubusercontent.com/websvcin/crudapi/main/.env.example

mv .env.example .env
# Edit .env if needed

docker compose up -d
docker compose logs -f
```

Visit **http://localhost:8080**.

---

## 📦 Option 2 — Pre-built ZIP

Download the latest release: https://github.com/websvcin/crudapi/releases/latest

```bash
wget https://github.com/websvcin/crudapi/releases/latest/download/crudapi-latest.zip
unzip crudapi-latest.zip -d crudapi
cd crudapi
dotnet CrudApi.dll
```

Requires **.NET 8 runtime** installed on the host.

---

## 🛠 Option 3 — Plesk / shared hosting

1. Download the latest ZIP
2. Upload to your hosting provider
3. Configure ASP.NET Core hosting (see your provider's docs)
4. Set environment variables via the control panel

---

## ⚠️ Important: persistent storage

CrudApi writes everything it needs to keep — the admin registry (every tenant, database, and
credential), the Data Protection keys that encrypt stored passwords/connection strings, setup
state, background jobs, rate-limit and Config Editor data, and uploaded files — to a single
**`/app/App_Data`** folder inside the container. That folder **must** be mounted as a volume, or
you lose everything the next time the container is recreated (a redeploy, a host reboot, an image
update all discard the container's writable layer).

No shell access on your hosting panel (so you can't `chown`)? Open the host directory to
world-write instead (`rwx` for "Other") so the non-root container user can still write to it.

**Upgrading an older container** that still mounts the separate `/app/Data` and `/app/bulk-files`
paths from an earlier release? Leave those mounts as they are — a mounted folder can't be renamed,
so the app keeps using it right where it is, and says so in the startup log. To consolidate onto
the single `App_Data` folder, stop the container, move the old `bulk-files` contents into a
`bulk-files` subfolder inside your data directory, then mount that same host directory at
`/app/App_Data` instead.

---

## 🔑 First-time setup

After install:

1. Visit **http://localhost:8080/setup** — there is no default account; this wizard creates
   your real first admin login.
2. Sign in to `/admin`, create your first **tenant** under **Tenants**, and copy its console key.
3. Go to `/{your-tenant-slug}/login`, sign in with the console key, and register your first
   database under **Databases** in the tenant workspace.
4. Create an API key under **Keys** in the same tenant workspace.
5. Start calling the API at `/api/v1/{tenant}/{database}/crud/{table}`.

See [QUICKSTART.md](QUICKSTART.md) for a guided walkthrough.

---

## 🩺 Health check

There's no dedicated `/health` endpoint — check that the app is responding at all:

```bash
curl -fsS http://localhost:8080/
```

A `200` response means the container is up.

---

## 🆘 Troubleshooting

| Symptom | Fix |
|---|---|
| 404 on every URL | Container probably crashed. `docker logs crudapi` |
| Admin data resets on restart | You forgot the `-v $(pwd)/data:/app/App_Data` volume mount, or the mounted directory isn't writable by uid 1000 |
| 500 errors after first start | Check `appsettings.json` for malformed JSON |
| CORS errors in browser | Add your origin from a database's own **Web origins · CORS** action in the tenant workspace |
| Behind a reverse proxy | See [DEPLOY.md](DEPLOY.md) — forwarded headers |

Full docs: https://websvcin.github.io/crudapi
