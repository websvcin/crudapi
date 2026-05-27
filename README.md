<div align="center">

# 🗃️ CrudApi

**Multi-tenant CRUD API platform with built-in IAM, bulk import, admin UI, and SQLite browser.**

[![Docker](https://img.shields.io/badge/docker-ghcr.io-blue?logo=docker)](https://github.com/websvcin/crudapi/pkgs/container/crudapi)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![.NET](https://img.shields.io/badge/.NET-8.0-purple)](https://dotnet.microsoft.com/)
[![Release](https://img.shields.io/github/v/release/websvcin/crudapi)](https://github.com/websvcin/crudapi/releases/latest)

[📖 Docs](https://websvcin.github.io/crudapi) ·
[🚀 Quickstart](QUICKSTART.md) ·
[🐳 Docker](https://github.com/websvcin/crudapi/pkgs/container/crudapi) ·
[💬 Issues](https://github.com/websvcin/crudapi/issues)

</div>

---

## ⚡ One-minute install

```bash
docker run -d -p 8080:80 \
  -v $(pwd)/data:/app/Data \
  --name crudapi \
  ghcr.io/websvcin/crudapi:latest
```

Open **http://localhost:8080** — landing page → admin console → register your first database → done.

Default admin login: `admin` / `admin@123` (change immediately).

---

## ✨ Why CrudApi?

| Feature | What you get |
|---|---|
| ⚡ **Zero-config CRUD** | Register a MySQL database → every table becomes a REST endpoint |
| 🔐 **IAM built in** | Users, roles, permissions, row-level policies, audit |
| 🔑 **3 auth modes** | API Key / Basic Auth / JWT — switch per credential |
| 📥 **Bulk CSV import** | Streaming, foreign-key resolution, enum validation, error CSV |
| 🌐 **Multi-tenant** | Host many databases; per-database CORS & credentials |
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
| `http://localhost:8080/admin` | Admin console (login required) |
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
