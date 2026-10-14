---
title: Access
parent: Tenant console
nav_order: 3
---

# Access
{: .no_toc }

How your **team** signs in to this console — not how your app reads your data (that's the
[Keys](keys.html) tab).
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## How your team signs in

Two options, side by side:

### Option 1 (recommended) — Sign in with LoginLink

Each teammate signs in with their own LoginLink account; you approve who gets in. Status badges:

- **Waiting for your administrator** — your platform administrator hasn't turned on LoginLink connection
  for this workspace yet. An **Ask to enable this** button sends that request — it shows up for them next
  time they open [Tenants](../admin-console/tenants.html) in the admin console.
- **Not connected** (connection is allowed, but not yet done) — a **Start connecting** button walks a
  wizard: create a Management API credential in LoginLink (its API Access page, with **Manage apps**
  ticked), then connect it here.
- **Connected** — shows how many people have linked their LoginLink account to this workspace vs. are still
  waiting, and lets you manage who's allowed in.

A **Start over…** control clears this workspace's LoginLink connection on the CrudAdmin side only —
**nothing in LoginLink itself is changed or deleted**; you'd remove objects there yourself if desired.

### Option 2 — Console key

For teammates without a LoginLink account: you create a key (see
[Admin console → Tenants → Console key](../admin-console/tenants.html#console-key) — a platform admin can
also issue these on your behalf), and they sign in with it at your workspace's own sign-in address, shown
right on this card. A console key opens **this console only** — it can never read or write actual data.

## Signed in now

Every active session into this console right now, combining both sign-in methods in one list — who, how
(Console key or LoginLink), where from, and when it started. Filter by method, search by who/how, and end
any session immediately with one click.
