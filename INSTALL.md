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

# Download the compose file
curl -O https://raw.githubusercontent.com/web-svc/crudapi/main/docker-compose.yml
curl -O https://raw.githubusercontent.com/web-svc/crudapi/main/.env.example

# Configure (optional)
mv .env.example .env
# Edit .env

# Run
docker compose up -d

# View logs
docker compose logs -f
