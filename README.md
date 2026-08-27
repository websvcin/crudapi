<div align="center">

# 🗃️ CrudApi

**Multi-tenant CRUD API platform with built-in IAM, rate limiting, bulk import, admin UI, and SQLite browser.**

![Docker Pulls](https://img.shields.io/docker/pulls/websvcin/crudapi?logo=docker)
![GHCR](https://img.shields.io/badge/ghcr.io-available-blue?logo=github)
![License](https://img.shields.io/badge/license-MIT-green)
![.NET](https://img.shields.io/badge/.NET-8.0-purple)
![Release](https://img.shields.io/github/v/release/websvcin/crudapi)

📖 https://websvcin.github.io/crudapi ·
🚀 [QUICKSTART.md](QUICKSTART.md) ·
🐳 https://hub.docker.com/r/websvcin/crudapi ·
💬 https://github.com/websvcin/crudapi/issues

</div>

---

## ⚡ One-minute install

```bash
docker run -d -p 8080:80 \
  -v $(pwd)/data:/app/Data \
  -v $(pwd)/bulk-files:/app/bulk-files \
  --name crudapi \
  websvcin/crudapi:latest
```

Open **http://localhost:8080/setup** — there's no default login; this creates your first admin account.

---

## ✨ Why CrudApi?

| Feature | What you get |
|---|---|
| ⚡ **Zero-config CRUD** | Register a MySQL database → every table becomes a REST endpoint |
| 🔐 **IAM built in** | Users, roles, permissions, row-level policies, audit |
| 🔑 **3 auth modes** | API Key / Basic Auth / JWT — switch per credential |
| 🚦 **Rate limiting** | Per-credential, per-IP, and global limits, plus tenant-wide daily/monthly/custom quotas that auto-reset |
| 📥 **Bulk CSV import** | Streaming, foreign-key resolution, enum validation, error CSV |
| 🌐 **Multi-tenant** | Host many databases; per-database CORS & credentials; a self-service Tenant Portal |
| 🧰 **Admin UI** | Visual database registration, role editor, audit viewer |
| 🗂️ **SQLite browser** | Built-in viewer for any `.db` file on the server |
| 📚 **In-app docs** | `/help` page bundled with the app |
| 🐳 **Docker-ready** | Multi-arch (amd64 + arm64), non-root, ~200MB |

---

## 🚀 Get started

1. **Quickstart** → [QUICKSTART.md](QUICKSTART.md) — 5-minute walkthrough
2. **Install** → [INSTALL.md](INSTALL.md) — Docker, manual, env vars
3. **Deploy** → [DEPLOY.md](DEPLOY.md) — production guide
4. **Docs site** → https://websvcin.github.io/crudapi

---

## 🧰 What's inside

Once the app is running:

| URL | Purpose |
|---|---|
| `http://localhost:8080/` | Landing page |
| `http://localhost:8080/help` | In-app documentation |
| `http://localhost:8080/setup` | First-run wizard — creates your admin account |
| `http://localhost:8080/admin` | Admin console (login required) |
| `http://localhost:8080/tenant/login` | Self-service Tenant Portal — a tenant admin manages their own databases/keys |
| `http://localhost:8080/ratelimitadmin` | Rate-limit admin — per-database, per-IP, and tenant quota controls |
| `http://localhost:8080/sqliteadmin` | SQLite browser (separate login) |
| `http://localhost:8080/docs` | Swagger / OpenAPI spec |
| `http://localhost:8080/api/v1/{db}/crud/{table}` | The actual CRUD API |

---

## 📥 Download

- **Docker:** `docker pull ghcr.io/websvcin/crudapi:latest`
- **Docker Hub:** `docker pull websvcin/crudapi:latest`
- **ZIP (.NET binaries):** https://github.com/websvcin/crudapi/releases/latest
- **Source code:** Private — contact us for licensing

---

## 📜 License

[MIT](LICENSE) © websvcin

Built with .NET 8, MySQL, SQLite, Docker.
