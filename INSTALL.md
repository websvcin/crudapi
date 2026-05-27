# Installation

Three ways to install CrudApi. Pick what fits your environment.

---

## 🐳 Option 1 — Docker (recommended)

### Single command

```bash
docker run -d -p 8080:80 \
  -v $(pwd)/data:/app/Data \
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
wget https://github.com/websvcin/crudapi/releases/latest/download/crudapi-v1.0.0.zip
unzip crudapi-v1.0.0.zip -d crudapi
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

CrudApi keeps these in `/app/Data`:

- `system.db` — admin users, roles, permissions, sessions
- `hangfire.db` — background job state
- `audit-*.log` — security audit trail
- `sqliteadmin-*.json` — viewer sessions

**Always mount this folder as a volume** in Docker, or you lose your admin users on every container restart.

---

## 🔑 First-time setup

After install:

1. Visit **http://localhost:8080**
2. Click **Admin Console**
3. Login: `admin` / `admin@123`
4. **Immediately change the password** at `/admin/adminusers`
5. Register your first database
6. Create an API key
7. Start calling the API

See [QUICKSTART.md](QUICKSTART.md) for a guided walkthrough.

---

## 🩺 Health check

```bash
curl http://localhost:8080/health
# {"status":"healthy"}
```

---

## 🆘 Troubleshooting

| Symptom | Fix |
|---|---|
| 404 on every URL | Container probably crashed. `docker logs crudapi` |
| Admin DB resets on restart | You forgot the `-v $(pwd)/data:/app/Data` volume mount |
| 500 errors after first start | Check `appsettings.json` for malformed JSON |
| CORS errors in browser | Add your origin in Admin → Databases → CORS |
| Behind a reverse proxy | See [DEPLOY.md](DEPLOY.md) — forwarded headers |

Full docs: https://websvcin.github.io/crudapi
