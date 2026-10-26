---
title: Databases
parent: Tenant console
nav_order: 2
---

# Databases
{: .no_toc }

Every database registered here belongs to this workspace only.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Registering a database

**+ Register a database** asks for a connection (host, port, database name, user, password) and tests it
before saving — nothing is stored until the test connection succeeds. If a platform administrator has
turned on managed databases for this tenant (see
[Admin console → Tenants → Managed databases](../admin-console/tenants.html#managed-databases)), a line at
the top of this tab also offers "ask for one" or "create one yourself," depending on the mode chosen there.

## The databases table

Each row: name/key, engine and host, connection status, configuration status, and an **Actions** menu.

### Actions menu

| Action | Does |
|---|---|
| **Generate Config** | Produces the dashboard configuration from the live schema — see [Features → Generate Config & dashboards](../features/generate-config-dashboards.html). Reports success, or "generated but not published" with the real reason if the GitHub push failed, never a silent either/or. Replaces a file already there by default, same as always; the [developer API's `ifExists` parameter](developer-api.html) can ask first instead, for a caller that wants to check before replacing. |
| **Config Column** (buttons) | **Open editor** launches the [Config Visualizer](../features/generate-config-dashboards.html#the-config-visualizer) via a signed handoff; **Regenerate** re-runs Generate Config for this exact database. |
| **Run a script** | Opens the script runner — see [Features → Run scripts](../features/run-scripts.html) for the full dry-run/confirm/execute flow, including the "execute here" credential picker. |
| **Row policy** | Add or edit a row-level restriction for a role on this database — see [Features → Row policies](../features/row-policies.html). Can optionally be copied to every other database in the workspace whose tables/columns match. |
| **Edit** | Change the stored connection details. |
| **Unregister** | Removes the database from this workspace's own registry only — its CORS origins, row policies, rate limits and cached config state go with it, but the underlying database itself (and its data) is never touched. The platform didn't create it and has no way to drop it. You can register it again later with the same connection details. |

## Row policies

A row policy limits a role to specific rows of a table (e.g. "Support can only see orders where `region`
is APAC") rather than whole tables. Add one from a specific database's own **Row policy** button — from
there you can also apply the identical rule to every other database in the workspace, if their schemas
match closely enough. Full detail, including exactly how it's enforced and how it interacts with `filter`:
[Features → Row policies](../features/row-policies.html).

## Managed databases

If self-service or request-based managed databases are enabled for this tenant, this tab also shows any
pending requests and their status, and — for self-service mode — lets you create a new managed database
directly, up to whatever limit the platform administrator has set.
