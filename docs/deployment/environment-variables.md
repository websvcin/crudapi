---
title: Environment variables
parent: Deployment
nav_order: 2
---

# Environment variables
{: .no_toc }

What's actually set at the environment level, vs. what's configured live through the admin console.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The important distinction

Most operational settings — AI provider/key, GitHub repo for config publishing, bulk-upload limits,
notification webhook, security policy — are stored in the admin registry itself and edited live from
[Settings](../admin-console/settings.html). **Only a small number of boot-time, infrastructure-level
choices are environment variables**, because they have to be known *before* the app can even open its own
data store.

## ASP.NET Core standard variables

| Variable | Default (in the published image) | Purpose |
|---|---|---|
| `ASPNETCORE_URLS` | `http://+:80` | Listen address/port. |
| `ASPNETCORE_HTTP_PORTS` | `80` | Alternative port binding; not needed unless you want a different port than 80 inside the container. |
| `ASPNETCORE_ENVIRONMENT` | `Production` | Standard ASP.NET Core environment name. |

## CrudAdmin bootstrap variables

| Variable | Purpose |
|---|---|
| `CRUDADMIN_DB_ENGINE` | `sqlite` (default) or `mysql` — which engine the platform's **own** admin registry uses. See [Admin console → Storage & data](../admin-console/storage-databases.html#switching-to-mysql) for switching an already-running instance. |
| `CRUDADMIN_MYSQL_CONNECTION_STRING` | Required if `CRUDADMIN_DB_ENGINE=mysql`. The connection string for the platform's own registry — **not** a tenant's data database. |
| `Storage__DataRoot` (double-underscore = ASP.NET Core's env-var nesting convention for `Storage:DataRoot`) | Relocates the entire `App_Data` folder (see [Docker](docker.html#whats-inside-app_data)) — absolute, or relative to the app folder. |

## Reverse proxy trust

| Variable | Purpose |
|---|---|
| `ReverseProxy__TrustedProxyIPs` | A list of IPs/CIDRs this instance trusts to set `X-Forwarded-*` headers. |

{: .warning }
> If left unset, the app falls back to trusting `X-Forwarded-*` headers from **any** source — fine behind
> a reverse proxy you fully control on a private network, but a real risk on anything else, since
> `generate-config` bakes the request's scheme/host into the published `apiBaseUrl`. **Set this explicitly
> in every real deployment.**

## A tenant's own database connection

A tenant's registered database connection is entered through the console (Databases → Register a
database), **not** set as an environment variable — the Plesk walkthrough's
`ASPNETCORE_ConnectionStrings__Default1=...` example is for quickly wiring up a test database reachable
from inside the container during initial verification, not the normal way a real tenant's database gets
connected.

## Where each setting actually lives

| If it's about... | It's set via |
|---|---|
| Where the app listens, which DB engine the registry uses, reverse-proxy trust, the data folder location | Environment variables (this page) |
| AI provider, GitHub publishing, bulk limits, notifications, LoginLink server address | [Admin console → Settings → Host Settings](../admin-console/settings.html#host-settings) |
| JWT policy, API key defaults, audit retention, session limits, password policy | [Admin console → Settings → Security Settings](../admin-console/settings.html#security-settings) |
| A tenant's own data database | [Tenant console → Databases](../tenant-console/databases.html) or [Admin console → Tenants](../admin-console/tenants.html) |
