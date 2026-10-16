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
[Persistent volume: ./data → /app/App_Data]
   ↓
[Your MySQL databases]
```

Terminate HTTPS at the reverse proxy, not inside the container — CrudApi listens on plain HTTP
internally and expects a proxy in front of it in any real deployment.

---

## 📦 docker-compose.yml (production)

```yaml
services:
  crudapi:
    image: ghcr.io/websvcin/crudapi:v1.0.97   # Pin the version! Check the latest at
                                               # https://github.com/websvcin/crudapi/releases
    container_name: crudapi
    restart: unless-stopped
    expose:
      - "80"
    volumes:
      - ./data:/app/App_Data
      - ./appsettings.Production.json:/app/appsettings.Production.json:ro
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=http://+:80
    healthcheck:
      test: ["CMD", "curl", "-fsS", "http://localhost:80/"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 20s
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

> Everything lives under the single `/app/App_Data` volume now — the admin registry, Data
> Protection keys, rate-limit state and Config Editor sessions together. There's no need for
> separate `RateLimitAdmin__DataDirectory`/`ConfigVisualizer__DataDirectory` environment variables
> an older guide might mention; those packages already default into subfolders of the same
> consolidated folder.

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

Caddy auto-provisions Let's Encrypt certs and renews them. nginx or Traefik work the same way —
the only requirement is that **something** terminates TLS in front of the container and forwards
plain HTTP to it.

> Whatever proxy you use, set `ReverseProxy__TrustedProxyIPs` to that proxy's address. Left unset,
> the app trusts `X-Forwarded-*` headers from any source — fine behind a proxy you fully control, a
> real gap otherwise.

---

## 🔐 Production checklist

- [ ] Change the first admin's password from whatever you set it to during setup
- [ ] Set secrets (GitHub token, AI key) via the admin console's Host Settings, not `appsettings.json`
- [ ] Mount `App_Data/` on encrypted storage
- [ ] Pin the Docker image to an exact version — never `:latest`
- [ ] Terminate HTTPS in front of the container
- [ ] Turn on `jwt.require_https` in the admin console's Security settings
- [ ] Set `ReverseProxy__TrustedProxyIPs` explicitly
- [ ] Configure CORS allow-lists per database (from a database's own **Web origins · CORS** action)
- [ ] Confirm the audit log is actually writing
- [ ] Download the recovery kit from the admin console's Storage & data page
- [ ] Configure backups for `App_Data/`

---

## 💾 Backups

```bash
# Crontab — daily 2 AM
0 2 * * * tar czf /backups/crudapi-$(date +\%Y\%m\%d).tar.gz /opt/crudapi/data
```

This captures the whole `App_Data` folder in one file — the admin registry, Data Protection keys,
rate-limit state, and Config Editor sessions together.

To restore:
1. Stop the container
2. Replace `data/` with the backup
3. Start the container

---

## 🧪 Health monitoring

There's no dedicated `/health` endpoint — monitor that the app responds at all:

```bash
curl -fsS https://api.example.com/
```

Integrate with Uptime Robot, Better Stack, Datadog, or Prometheus by checking for a `200`.

---

## 🔄 Updates

```bash
docker pull ghcr.io/websvcin/crudapi:<new-version>   # pin the exact tag, don't float on :latest
docker compose up -d
curl -fsS https://api.example.com/
```

Always read [CHANGELOG.md](CHANGELOG.md) before upgrading, especially for any breaking change to
the data-API URL shape or credential behavior.

---

## 🆘 Roll back

Edit the image tag in `docker-compose.yml` back to your last known-good version (see
[Releases](https://github.com/websvcin/crudapi/releases) for the list), then:

```bash
docker compose up -d
```

If the new version wrote something incompatible into `App_Data`:

```bash
docker compose down
rm -rf data
tar xzf /backups/crudapi-YYYYMMDD.tar.gz -C .
docker compose up -d
```
