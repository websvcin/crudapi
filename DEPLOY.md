# 🚀 Production Deployment

Complete production guide: persistence, reverse proxy, HTTPS, backups.

---

## 🏗 Architecture (recommended)

```
Internet
   ↓
[Reverse Proxy: Caddy/nginx/Traefik]    ← TLS terminates here
   ↓ (HTTP)
[CrudApi container on port 8080]
   ↓
[Persistent volume: ./data → /app/Data]
   ↓
[Your MySQL databases]
```

---

## 📦 docker-compose.yml (production)

```yaml
version: "3.8"

services:
  crudapi:
    image: ghcr.io/websvcin/crudapi:v1.0.0   # Pin the version!
    container_name: crudapi
    restart: unless-stopped
    expose:
      - "80"
    volumes:
      - ./data:/app/Data
      - ./appsettings.Production.json:/app/appsettings.Production.json:ro
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=http://+:80
    healthcheck:
      test: ["CMD", "curl", "-fsS", "http://localhost:80/health"]
      interval: 30s
      timeout: 5s
      retries: 3
    networks:
      - web

  caddy:
    image: caddy:2-alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy-data:/data
    depends_on:
      - crudapi
    networks:
      - web

networks:
  web:

volumes:
  caddy-data:
```

---

## 🔒 Reverse proxy with auto-HTTPS (Caddy)

`Caddyfile`:

```caddy
api.example.com {
    reverse_proxy crudapi:80

    header {
        X-Content-Type-Options nosniff
        X-Frame-Options SAMEORIGIN
        Strict-Transport-Security max-age=31536000
    }

    encode gzip
}
```

Caddy auto-provisions Let's Encrypt certs and renews them.

---

## 🔐 Production checklist

- [ ] Change default admin password
- [ ] Set secrets via env vars (not appsettings.json)
- [ ] Mount `Data/` on encrypted storage
- [ ] Pin Docker image version (`:v1.0.0`, not `:latest`)
- [ ] Enable HTTPS in front of the container
- [ ] Set `jwt.require_https = true` at `/admin/settings`
- [ ] Configure CORS allow-list per database
- [ ] Test the audit log is writing
- [ ] Configure backups for `Data/`

---

## 💾 Backups

```bash
# Crontab — daily 2 AM
0 2 * * * tar czf /backups/crudapi-$(date +\%Y\%m\%d).tar.gz /opt/crudapi/data
```

To restore:
1. Stop the container
2. Replace `data/` with the backup
3. Start the container

---

## 🧪 Health monitoring

```bash
curl https://api.example.com/health
# {"status":"healthy"}
```

Integrate with Uptime Robot, Better Stack, Datadog, or Prometheus.

---

## 🔄 Updates

```bash
docker pull ghcr.io/websvcin/crudapi:v1.0.1
docker compose up -d
curl https://api.example.com/health
```

Always read [CHANGELOG.md](CHANGELOG.md) before upgrading.

---

## 🆘 Roll back

```bash
sed -i 's/v1.0.1/v1.0.0/' docker-compose.yml
docker compose up -d
```

If the new version corrupted Data/:

```bash
docker compose down
rm -rf data
tar xzf /backups/crudapi-YYYYMMDD.tar.gz -C .
docker compose up -d
```
