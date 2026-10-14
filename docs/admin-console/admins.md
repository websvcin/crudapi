---
title: Admin users
parent: Admin console
nav_order: 9
---

# Admin users
{: .no_toc }

`/admin/adminusers` — your platform team's own login accounts, and how they sign in.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

This is strictly about **your own platform team's** accounts at `/admin/login` — entirely separate from
tenant accounts, tenant users, or any data-API credential.

## Managing admin accounts

**+ Create Admin** asks for a username, password, and an optional **Super Admin** checkbox. Super Admins
have a few abilities regular admins don't (e.g. opening the [SQLite browser](storage-databases.html) and
downloading the [recovery kit](storage-databases.html#recovery-kit)), and — the one hard rule on this page —
**a Super Admin account cannot be deleted**. A regular admin's row has a **Delete** option; editing lets you
change display name, email, and active/inactive status.

## Sign-in methods

A small table showing which ways your team may sign in: **Password** (always available) and **LoginLink**
(shows its current adoption and an on/off pill). Configuring LoginLink affects **only your own platform
team's sign-in** — it has no effect on tenants or their users, which have their own entirely separate
LoginLink connections (see [Tenants → Connecting LoginLink per tenant](tenants.html#connecting-loginlink-per-tenant)).

## Configuring LoginLink sign-in

**Configure LoginLink sign-in** opens a 6-step wizard the first time:

1. **Your organisation** — the LoginLink organisation your team signs into, typed exactly as it appears in
   your browser's address bar while signed into LoginLink (it can't be read back from a credential, so this
   has to be typed correctly — a wrong name only surfaces later, at the connection test in step 5).
2. **Management credential** — a LoginLink credential CrudAdmin uses to manage this sign-in app.
3–4. Further app/organisation detail steps.
5. **Test the connection** — a live check against the address built from step 1.
6. **Who can sign in** — which LoginLink users/roles are allowed to actually reach `/admin/login` this way.

Once configured, the same button instead opens a management view with:

| Action | Does |
|---|---|
| **🔁 Test again** | Re-checks the saved app against LoginLink. Changes nothing. |
| **🔑 Change management credential** | Creates a new app with a new credential and replaces the saved values. |
| **🏢 Change organisation** | Points sign-in at a different organisation entirely — creates a new app there. |
| **👥 Change who can sign in** | Updates the existing app's access list on LoginLink. No new app created. |
| **Turn on/off** | Toggles whether LoginLink sign-in is currently accepted, without discarding the setup. |
| **⟲ Start over** | Clears this setup in CrudAdmin and turns LoginLink sign-in off. **Nothing is deleted on LoginLink itself** — only the local connection. |

{: .note }
> If this screen reads *"This workspace is connected directly to the platform's own LoginLink organisation,
> with no separate slug"*, that's describing a connection to the **platform root** organisation rather than
> a sub-path one — not an error. Use **Start over** then reconnect with a specific organisation slug if you
> want this team on its own separate LoginLink organisation instead.
