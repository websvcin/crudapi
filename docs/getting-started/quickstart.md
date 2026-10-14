---
title: Quickstart
parent: Getting started
nav_order: 1
---

# Quickstart
{: .no_toc }

From a running CrudApi instance to your first live API call, in order.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 0. Before you start

You need a running CrudApi instance. If you don't have one yet, see [Deployment](../deployment/) — this
page assumes the app is already up and reachable (locally, that's usually `http://localhost:5291`, but any
address works the same way).

The very first time anyone opens the app, it redirects to a one-time **setup wizard** (`/setup`) that
creates the first platform administrator account. If you land there, see
[Setup wizard](../deployment/setup-wizard.html) first, then come back here.

## 1. Sign in as the platform administrator

Open `/admin` and sign in with the account the setup wizard created.

## 2. Create a tenant

A **tenant** is one customer/workspace — its data, keys, and people are completely separate from every
other tenant's.

1. Go to **Tenants** in the admin sidebar.
2. Click **+ Create Tenant**.
3. Give it an id, a name, and a slug (the slug is what appears in its sign-in address).

See [Admin console → Tenants](../admin-console/tenants.html) for every field and option on this screen,
including optional LoginLink auto-provisioning.

## 3. Sign in to the tenant's own workspace

Each tenant has its own console, separate from the admin console, reached at `/{slug}/login`. The simplest
way in for now is a **console key** — a credential that opens the console only, and can never touch data:

1. Back in the admin console, on the Tenants page, open the tenant you just created and click
   **🔑 Console key**.
2. Copy the key it shows you (shown once — if you lose it, generate a new one).
3. Go to `/{your-slug}/login`, choose the **API Key** tab, and paste it in.

You're now in the tenant's own workspace — see [Tenant console → Overview](../tenant-console/overview.html)
for a tour of what's here.

## 4. Register a database

In the tenant workspace, go to **Databases → + Register a database**, and give it the connection details
for a MySQL database you already run. CrudApi tests the connection before saving it — nothing is stored
until the test passes.

Full detail on every field, plus the self-service "create a managed database for me" option if your
platform administrator has turned it on: [Tenant console → Databases](../tenant-console/databases.html).

## 5. Create an API key

Still in the tenant workspace, go to **Keys → + Create API key**:

1. Pick which database(s) the key may reach.
2. Pick a role — this decides what the key may *do* (read-only, read/write, full access, …). See
   [Concepts → Authentication & credentials](../concepts/authentication-and-credentials.html#roles) for what
   each built-in role actually allows.
3. Create it, and copy the key it shows you — **this is the only time you'll see it.**

## 6. Call the API

Every table in the database you registered is now a secure REST endpoint:

```bash
curl -H "X-API-Key: drk_your_key_here" \
  "https://your-host/api/v1/your-tenant-slug/your-database-key/crud/orders?limit=10"
```

That's the whole shape of a request: `/api/v1/{tenant}/{database}/crud/{table}`, plus the credential header.
Everything else — filtering, sorting, joins, pagination, writes, bulk import, row-level restriction — builds
on exactly this. See the full [API reference](../api-reference/) next.

{: .note }
> **A table name is the resource.** Permissions, row policies, and rate limits are all expressed in terms of
> the table (`widgets.read`, `widgets.create`, …), not a separate "endpoint" concept — if your key's role can
> read a table, `GET .../crud/that-table` works; if it can't, you get a clear `forbidden_action` error
> instead of a 404.

## What's next

- [API reference](../api-reference/) — every endpoint, every parameter, every credential type, in full.
- [Concepts](../concepts/) — how multi-tenancy, roles, and row policies actually fit together, if you're
  about to design access control for real customers.
- [Features → Bulk import](../features/bulk-import.html) — loading more than a handful of rows at once.
- [Features → Row policies](../features/row-policies.html) — restricting a key or a role to *some* rows of a
  table, not just some tables.
