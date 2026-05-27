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

## Option 2 — Pre-built ZIP
Download the latest release: https://github.com/web-svc/crudapi/releases/latest

wget https://github.com/web-svc/crudapi/releases/latest/download/crudapi-v1.0.0.zip
unzip crudapi-v1.0.0.zip -d crudapi
cd crudapi
dotnet CrudApi.dll

# View logs
docker compose logs -f
