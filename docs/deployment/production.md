---
title: Production deployment
parent: Deployment
nav_order: 5
---

# Production deployment
{: .no_toc }

Persistence, a reverse proxy with automatic HTTPS, backups, and rollback — everything beyond the
single `docker run` in [Docker](docker.html).
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Recommended architecture

```
Internet
   ↓
[Reverse proxy: Caddy / nginx / Traefik]    ← TLS terminates here
   ↓ (plain HTTP)
[CrudApi container on port 80, not exposed directly]
   ↓
[Persistent volume: ./data → /app/App_Data]
   ↓
[Your MySQL databases]
```

Terminate HTTPS at the reverse proxy, not inside the container — CrudApi listens on plain HTTP
internally and expects a proxy in front of it in any real deployment.

## Production `docker-compose.yml`

```yaml
services:
  crudapi:
    image: ghcr.io/websvcin/crudapi:v1.0.0   # pin a real version — never run :latest in production;
                                              # check https://github.com/websvcin/crudapi/releases
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

{: .note }
> Everything lives under the single `/app/App_Data` volume now — see
> [Docker → What's inside App_Data](docker.html#whats-inside-app_data). There's no need for the
> separate `RateLimitAdmin__DataDirectory`/`ConfigVisualizer__DataDirectory` environment variables
> older guides mention; those packages already default into subfolders of the same consolidated
> folder.

## Reverse proxy with automatic HTTPS (Caddy)

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

Caddy auto-provisions and renews Let's Encrypt certificates — no manual certbot setup. nginx or
Traefik work the same way; the only requirement is that **something** terminates TLS in front of
the container and forwards plain HTTP to it.

{: .warning }
> Whatever proxy you use, set `ReverseProxy__TrustedProxyIPs` (see
> [Environment variables → Reverse proxy trust](environment-variables.html#reverse-proxy-trust)) to
> that proxy's address. Left unset, the app trusts `X-Forwarded-*` headers from any source — fine
> behind a proxy you fully control, a real gap otherwise.

## Production checklist

- [ ] Change the first admin's password from whatever you set it to during setup, if it was a placeholder
- [ ] Set secrets (GitHub token, AI key) via Host Settings, not `appsettings.json`
- [ ] Mount `App_Data/` on encrypted storage
- [ ] Pin the Docker image to an exact version, never `:latest`
- [ ] Terminate HTTPS in front of the container
- [ ] Turn on `jwt.require_https` in [Security Settings](../admin-console/settings.html#security-settings)
- [ ] Set `ReverseProxy__TrustedProxyIPs` explicitly
- [ ] Configure CORS allow-lists per database
- [ ] Confirm the [audit log](../admin-console/audit-logs.html) is actually writing
- [ ] Download the [recovery kit](../admin-console/storage-databases.html#recovery-kit)
- [ ] Set up the backup schedule below

## Backups

```bash
# Crontab — daily at 2 AM
0 2 * * * tar czf /backups/crudapi-$(date +\%Y\%m\%d).tar.gz /opt/crudapi/data
```

This captures the whole `App_Data` folder in one file — the admin registry, Data Protection keys,
rate-limit state, and config-editor sessions together, consistent with
[Storage & data → Backing this up in practice](../admin-console/storage-databases.html#backing-this-up-in-practice).

To restore: stop the container, replace the `data/` folder with the backup's contents, start the
container again.

## Health monitoring

There's no dedicated `/health` endpoint — monitor that the app responds at all:

```bash
curl -fsS https://api.example.com/
```

Wire a `200` into Uptime Robot, Better Stack, Datadog, Prometheus, or whatever you already use.

## Updating

```bash
docker pull ghcr.io/websvcin/crudapi:<new-version>   # pin the exact tag, don't float on :latest
docker compose up -d
curl -fsS https://api.example.com/
```

Check [Reference → Changelog](../reference/changelog.html) before upgrading, especially for any
breaking change to the data-API URL shape or credential behavior.

## Rolling back

Edit the image tag in `docker-compose.yml` back to your last known-good version, then:

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
