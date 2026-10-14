---
title: Tenants
parent: Admin console
nav_order: 2
---

# Tenants
{: .no_toc }

`/admin/tenants` — create, inspect, and manage every tenant on the platform.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The tenant list

Each tenant is a card showing its name and slug, an Active/Inactive status badge, and a row of counts:
databases, roles, app users, keys (every API key/Basic login/issuance key the tenant has), app tokens
currently in use (backend-issued tokens for end-users, not team sign-ins), and calls kept (usage-log detail
retained for 30 days). Two more badges appear conditionally: a LoginLink connection badge, and a managed-
database mode badge (see below). Clicking the tenant's name opens its active console/app sessions, with a
**Revoke** option per session.

Tenants waiting on a LoginLink self-service request you haven't approved yet sort to the top of the list,
with a banner counting how many are waiting, so none get missed below the fold.

### Per-tenant actions

| Button | Does |
|---|---|
| **Edit** | Change the tenant's display name, or flip Active/Inactive. |
| **🔗 LoginLink** | Open the LoginLink connection modal (below). |
| **👁 Support view** | Open a read-only, 30-minute view of the tenant's own console, logged with a reason. |
| **🔑 Console key** | Issue a console-only sign-in credential for this tenant's administrator. |
| **🗄 Databases** | Configure managed-database limits and provisioning mode. |
| **🔑 Access** | Jump to [Access model](access-model.html), pre-filtered to this tenant's guardrail exception. |
| **Delete** | Permanently remove the tenant record (only once it holds nothing — see below). |

## Creating a tenant

**+ Create Tenant** asks for an id (unique, used internally — never shown in URLs), a name, and a slug
(auto-suggested from the name, editable; this becomes part of its sign-in address). If
[new-tenant provisioning](#new-tenant-provisioning-with-loginlink) is on, the form also asks for the first
admin's email (required — becomes their LoginLink sign-in) and name (optional).

If LoginLink provisioning is on and succeeds, you're shown the new organisation's admin password **once** —
the Close button on that dialog stays disabled until you've copied it, since it is never shown again. If
provisioning fails, the tenant is still created; a **Retry** control appears on its card.

## New-tenant provisioning with LoginLink

A platform-wide switch, shown as a status card at the top of this page: when on, every **+ Create Tenant**
also creates that tenant's own sign-in organisation on LoginLink automatically. The card shows how many
tenants exist in total, how many were created via LoginLink, and how many were connected to LoginLink by
hand instead.

Turning it on walks a three-step wizard:

1. **LoginLink server** — read-only here; set once in [Host Settings](settings.html#host-settings).
2. **Credential** — paste a Client ID and Secret from a LoginLink credential that has only the
   `CreateTenants` permission ticked. The secret is stored encrypted and never shown again; closing the
   wizard without finishing clears whatever you'd typed.
3. **Check and turn on** — a live check call to LoginLink confirming the credential can actually create
   tenants. Nothing is saved until this check passes.

## Connecting LoginLink per tenant

The **🔗 LoginLink** button opens a modal whose content depends on whether that tenant is already
connected:

- **Already connected** — shows the issuer and client id it's connected as, with a **Disconnect** option.
  While you manage the connection, the tenant cannot connect its own.
- **Not connected** — first asks *who* should be in control:
  - **The tenant connects it** — they link their own LoginLink organisation from their own Access tab; you
    just turn this on and approve who gets in. A toggle further lets you allow this per tenant.
  - **You connect it** — handled right here, with two paths:
    - **⚡ Create a new one** — CrudAdmin creates a brand-new LoginLink organisation using the platform-wide
      provisioning credential (needs an admin email + optional name for that org's first user). Disabled
      with an explanation if platform-wide provisioning isn't on.
    - **🔗 Connect an existing one** — a four-step guide for finding the tenant slug (from the LoginLink
      console's own URL), creating an API-access credential there with **Manage apps** ticked, and pasting
      its Client ID/Secret here. An **Advanced** section lets you override with a full issuer URL for a
      LoginLink instance on its own custom domain.

## Console key

Issues a credential that signs one person into this tenant's own console (`/{slug}/login`) **and nothing
else** — no database, no role, so it can never read or write data. Give it a label (e.g. "Alice, tenant
admin") and an expiry (7/30/90/365 days, or never — not recommended). Shown once; copy it before closing.

## Support view

Opens the tenant's own console exactly as they'd see it, **read-only**, for 30 minutes. You must give a
reason (up to 300 characters) before opening it — the tenant can see this reason in their own activity log,
so support access is never silent.

## Managed databases

Decides whether CrudAdmin may provision real databases on this tenant's behalf (vs. the tenant only
registering databases it already has elsewhere), and how many it may have in total:

- **How many databases can it register?** — Unlimited (default) or a specific number, counting every
  database the tenant has registered, managed or not. Lowering the limit below what it already has doesn't
  remove anything — it only blocks registering more.
- **Should CrudAdmin create databases for it?**
  - **Off** — the tenant only registers databases it already has.
  - **Ask first** — the tenant requests one, and you approve or reject each request from this same modal.
  - **Self-service** — the tenant creates its own managed databases directly, up to a limit you set.
  - For either of the last two, you also set a **name prefix** — what the database is actually called on
    the underlying server (never used in the tenant's own API calls, which always address it by its own
    `db_key`).

**Pending requests** (in "Ask first" mode) list each requested name and reason with **Approve**/**Reject**
buttons. Approving genuinely calls out to the real provisioning account (Plesk, cPanel, or a MySQL server
you control) — it can take a few seconds, and the button shows a spinner rather than appearing frozen. A
request that was approved but failed to actually provision (a name collision, an unreachable account) stays
visible with the error and a **Retry** button, rather than silently vanishing into the audit log.

## Deleting a tenant

Deletion first runs a live check of everything the tenant still holds — app users, keys of every type,
databases, roles, active app tokens, current team sign-ins. **If it holds anything at all, nothing is
deleted** — you get a count of exactly what's in the way, and need to clear it first. Only once a tenant
holds nothing does the delete button activate, and even then you must type the tenant's exact id to confirm.
This cannot be undone. If the tenant was connected to LoginLink, deleting it here does **not** remove its
LoginLink organisation — that stays and needs removing there separately, if desired.
