# Installation

Three ways to install CrudApi. Pick what fits your environment.

---

## 🐳 Option 1 — Docker (recommended)

### Single command

```bash
docker run -d -p 8080:80 \
  -v $(pwd)/data:/app/Data \
  -v $(pwd)/bulk-files:/app/bulk-files \
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

CrudApi writes to **two** folders inside the container — both must be mounted as volumes, or
you lose data the next time the container is recreated (a redeploy, a host reboot, an image
update all discard the container's writable layer):

- **`/app/Data`** — `system.db` (admin users, roles, permissions, every registered tenant and
  database), the encryption keys protecting stored credentials, `hangfire.db` (background job
  state), and audit logs.
- **`/app/bulk-files`** — files uploaded through the bulk-import feature.

If you're also running the rate-limiting admin (`/ratelimitadmin`, enabled by default), its own
SQLite store defaults to a path *outside* `/app/Data` — point it at a subfolder of your already-
mounted data volume with an environment variable, or its limits/quotas/usage history are lost on
every container recreation too:

```bash
-e RateLimitAdmin__DataDirectory=/app/Data/RateLimitAdmin
```

**Always mount both folders as volumes** in Docker, or you lose your admin users, uploaded
files, and rate-limit configuration on every container restart.

---

## 🔑 First-time setup

After install:

1. Visit **http://localhost:8080/setup** — there is no default account; this wizard creates
   your real first admin login.
2. Log in and register your first database under **Admin → Databases**.
3. Create an API key under **Admin → API Clients**.
4. Start calling the API.

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
| Admin DB resets on restart | You forgot the `-v $(pwd)/data:/app/Data` volume mount |
| Uploaded files disappear after redeploy | You forgot the `-v $(pwd)/bulk-files:/app/bulk-files` volume mount |
| Rate limits/quotas reset after redeploy | Set `RateLimitAdmin__DataDirectory` to a subfolder of your mounted data volume — see above |
| 500 errors after first start | Check `appsettings.json` for malformed JSON |
| CORS errors in browser | Add your origin in Admin → Databases → CORS |
| Behind a reverse proxy | See [DEPLOY.md](DEPLOY.md) — forwarded headers |

Full docs: https://websvcin.github.io/crudapi
