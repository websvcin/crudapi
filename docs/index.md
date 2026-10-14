---
title: Home
layout: home
nav_order: 1
description: "CrudApi documentation — turn your databases into secure, multi-tenant CRUD APIs."
permalink: /
---

# CrudApi documentation
{: .fs-9 }

Turn your databases into secure, multi-tenant CRUD APIs in about a minute — no backend code to write.
{: .fs-6 .fw-300 }

[Quickstart](getting-started/quickstart.html){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[API reference](api-reference/){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## What CrudApi is

CrudApi is a **multi-tenant, schema-driven CRUD API platform**. Register a database you already run, and
every table in it becomes a generic, secure REST endpoint — with authentication, per-row access policies,
filtering, bulk import, rate limiting, an auto-generated admin dashboard, and a full audit trail — without
writing a line of backend code.

It's built around two separate consoles and one data API:

- **The [admin console](admin-console/)** — for the platform operator. Create tenants, register databases,
  issue platform-wide credentials, set access guardrails, and see everything that happens across every
  tenant.
- **The [tenant console](tenant-console/)** — for each customer/workspace. Register their own databases,
  create API keys and manage who holds them, build roles, watch usage, and configure their own sign-in.
- **The [data API](api-reference/)** — what your application actually calls at runtime: plain REST endpoints
  over your tables, secured by whichever credential type fits your use case.

## Where to start

| If you want to… | Go to |
|---|---|
| Get a live API running in a few minutes | [Quickstart](getting-started/quickstart.html) |
| Understand how multi-tenancy, auth, and permissions fit together | [Concepts](concepts/) |
| Look up an endpoint, its parameters, and its response shape | [API reference](api-reference/) |
| Understand a specific admin console screen | [Admin console](admin-console/) |
| Understand a specific tenant console screen | [Tenant console](tenant-console/) |
| Learn about a specific capability (bulk import, rate limits, row policies, …) | [Features](features/) |
| Deploy this yourself | [Deployment](deployment/) |

## How your application actually talks to CrudApi

This is the part most developers care about first, so it's worth saying plainly here as well as in the
[API reference](api-reference/): once a database is registered and you have a credential, every table is
reachable at

```
GET/POST/PUT/DELETE  /api/v1/{tenant}/{database}/crud/{table}
```

with the credential sent as a header — which header depends on the credential type:

| Credential | Header | Typical use |
|---|---|---|
| API client key (`drk_...`) | `X-API-Key: drk_...` | Server-to-server, the common case |
| Basic Auth | `Authorization: Basic <base64(user:pass)>` | Simple server-to-server, legacy tooling |
| Issuance-key JWT | `Authorization: Bearer <token>` | Short-lived, per-end-user tokens your backend issues |
| LoginLink token | `Authorization: Bearer <token>` | A signed-in person, via your LoginLink-connected sign-in |

Full detail, every endpoint, every query parameter, and exactly how each credential type is created: see
[API reference](api-reference/) and [Concepts → Authentication & credentials](concepts/authentication-and-credentials.html).
