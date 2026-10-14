---
title: Multi-tenancy
parent: Concepts
nav_order: 2
---

# Multi-tenancy
{: .no_toc }

What a tenant actually is, and exactly where the walls between tenants sit.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What a tenant is

A **tenant** is one customer or workspace. Everything a tenant owns — its databases, its API keys, its
people, its roles, its own LoginLink connection, its own usage history — belongs to that tenant alone and is
invisible to every other tenant. A platform can host any number of tenants on one running instance; nothing
about the data API, the tenant console, or billing-relevant usage tracking assumes there's only one.

Each tenant has:

- An **id** — internal, stable, never shown as part of a URL.
- A **slug** — the human-facing short name that appears in its sign-in address (`/{slug}/login`) and, for
  LoginLink, its own sub-path organisation.
- A **name** — the display label shown in the admin console's tenant list.

## Where the wall actually is

The tenant boundary is enforced at the same point every single request already resolves its target: the
`{tenant}` segment of the URL. A credential is always issued *for* one tenant — there is no data-API
credential type that spans tenants. Concretely:

- An API client key, Basic Auth login, or issuance key created in tenant A's workspace is checked only
  against tenant A's databases. Presenting it against `/api/v1/tenant-b/...` fails authentication outright —
  it isn't merely denied by role, the credential doesn't resolve at all under a different tenant.
- A LoginLink token is scoped to whichever LoginLink organisation that tenant is connected to; a token minted
  for one tenant's organisation doesn't resolve against another tenant's connection.
- Console sign-in is per tenant too — signing into `/acme/login` gives you acme's workspace, never another
  tenant's, regardless of which account or key you used to get in.

The **only** credential type that legitimately crosses tenant boundaries is a **Platform Developer Key**,
and only against the *management* API (creating tenants, listing every tenant's databases for support,
etc.) — never against any tenant's actual data. See
[Concepts → Authentication & credentials](authentication-and-credentials.html#developer-keys-the-management-api).

## Two databases, same key, different tenants — not a collision

A database is identified by the pair `{tenant}/{database}`, not by `{database}` alone. Two unrelated tenants
can each register a database with the key `reports` and never conflict — their rows live in physically
different connection targets, resolved independently per tenant. There is no global database registry a
`database` key is checked against; it only has to be unique *within* its own tenant.

## What a tenant can configure about itself

Everything in the [tenant console](../tenant-console/) — registering databases, creating keys, building
roles and row policies, inviting people, connecting LoginLink, watching usage — is self-service, with no
platform-administrator involvement required for the day-to-day case. The platform administrator's role is
mostly **initial provisioning and guardrails**: creating the tenant in the first place, and deciding whether
that tenant is allowed to hand out the more powerful built-in roles (Database admin, Owner) or build its own
custom roles — see [Admin console → Access model](../admin-console/access-model.html).

## What the platform administrator can see across tenants

The admin console can see and manage every tenant's databases, keys, and settings when it needs to — that's
not a boundary violation, it's the operator's own surface, separate from (and audited separately from) each
tenant's own console. Every such cross-tenant action a platform administrator takes is written to the audit
log exactly like a tenant's own actions are, so "who touched tenant X, and was it X or the platform operator"
is always answerable afterward.
