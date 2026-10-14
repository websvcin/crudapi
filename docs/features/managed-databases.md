---
title: Managed databases
parent: Features
nav_order: 4
---

# Managed databases
{: .no_toc }

Letting CrudAdmin create real databases on a tenant's behalf, instead of the tenant only registering ones
it already has.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The provisioning account

A platform administrator configures one credential the whole platform uses to actually create databases,
in [Host Settings](../admin-console/settings.html#provisioning-account) — one of three kinds:

| Kind | What it needs |
|---|---|
| A MySQL server you run | An admin login that can `CREATE`/`CREATE USER`/`GRANT`. |
| Plesk server | Panel address, target subscription, and an admin login or secret key. |
| cPanel account | Panel address, account username, and an API token. |

## Per-tenant mode

Configured per tenant from [Admin console → Tenants → Databases](../admin-console/tenants.html#managed-databases):

| Mode | Behavior |
|---|---|
| **Off** | The tenant only registers databases it already has elsewhere. |
| **Ask first** | The tenant requests a database by name; a platform administrator approves or rejects each request. |
| **Self-service** | The tenant creates its own managed databases directly, up to a limit the platform administrator sets. |

Both the "ask first" and "self-service" modes also set a **name prefix** — how the database is actually
named on the underlying server, which is never exposed in the tenant's own API calls (those always address
a database by its own `db_key`, independent of the real server-side name).

## Requesting or creating one

From the tenant side, this surfaces on the [Databases](../tenant-console/databases.html) tab — either a
request form (ask-first mode) or a direct "create a managed database" action (self-service mode), both
counted against whatever overall database limit the platform administrator has set for that tenant.

## What happens on approval

Approving a request (or a self-service creation) genuinely calls out to the real provisioning account —
Plesk, cPanel, or a MySQL server — which can take a few seconds, not instant. A request that was approved
but failed to actually provision (a name collision, an unreachable account) stays visible with its error
and a retry option, rather than silently disappearing.
