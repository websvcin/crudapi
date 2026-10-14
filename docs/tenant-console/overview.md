---
title: Overview
parent: Tenant console
nav_order: 1
---

# Overview
{: .no_toc }

The first tab after signing in — everything for your workspace, in one place.
{: .fs-6 .fw-300 }

---

## Stat cards

Quick counts for the workspace: databases, keys, users, and similar top-line numbers.

## Health

Are your registered databases actually reachable right now, and does each have a generated dashboard
configuration? This is the same signal the admin console's [Dashboard](../admin-console/dashboard.html)
shows for this tenant in its own tenant table, surfaced here from the tenant's own side.

## Needs attention

Things worth doing soon — an empty database with no config generated yet, a key approaching expiry, a
pending managed-database request, and similar nudges, each linking straight to where you'd act on it.

## Recent activity

The latest entries from this workspace's own [Activity](activity.html) log, with everything filterable
there also visible here at a glance.

{: .note }
> Everything on this page is scoped to **this workspace only** — a platform administrator viewing this via
> [Support view](../admin-console/tenants.html#support-view) sees the exact same page, with a banner at the
> top noting they're a platform admin looking in, read-only, for 30 minutes.
