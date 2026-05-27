# Installation

Three ways to install CrudApi. Pick what fits your environment.

---

## 🐳 Option 1 — Docker (recommended)

### Single command

```bash
docker run -d -p 8080:80 \
  -v $(pwd)/data:/app/Data \
  --name crudapi \
  ghcr.io/web-svc/crudapi:latest
