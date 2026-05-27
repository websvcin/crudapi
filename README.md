<div align="center">

# 🗃️ CrudApi

**Multi-tenant CRUD API platform with built-in IAM, bulk import, admin UI, and SQLite browser.**

[![Docker](https://img.shields.io/badge/docker-ghcr.io-blue?logo=docker)](https://github.com/web-svc/crudapi/pkgs/container/crudapi)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![.NET 8](https://img.shields.io/badge/.NET-8.0-purple)](https://dotnet.microsoft.com/)
[![Latest Release](https://img.shields.io/github/v/release/web-svc/crudapi)](https://github.com/web-svc/crudapi/releases/latest)

📖 [**Documentation**](https://web-svc.github.ontainer/crudapi) ·
💬 [**Issues**](https://github.com/web-svc/crudapi/issues)

</div>

---

## ⚡ One-minute install

```bash
docker run -d -p 8080:80 \
  -v $(pwd)/data:/app/Data \
  --name crudapi \
  ghcr.io/web-svc/crudapi:latest
