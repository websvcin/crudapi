# 🚀 Production Deployment

This guide covers running CrudApi in production with proper persistence, reverse proxy, HTTPS, and backups.

---

## 🏗 Architecture (recommended)


Internet
↓
[Reverse Proxy: nginx / Caddy / Traefik]    ← TLS terminates here
↓ (HTTP)
[CrudApi container on port 8080]
↓
[Persistent volume: ./data → /app/Data]
↓
[Your MySQL databases]
