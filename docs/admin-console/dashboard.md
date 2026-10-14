---
title: Dashboard
parent: Admin console
nav_order: 1
---

# Dashboard
{: .no_toc }

`/admin` — platform health at a glance. The first thing you see after signing in.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The four summary tiles

| Tile | Shows |
|---|---|
| Tenants | Total tenant count, plus a "· N suspended" note if any are suspended. |
| Tenant databases | Total databases registered across every tenant. |
| Failed sign-ins, 24h | Failed authentication attempts in the last day. |
| Need attention | Count of items in the **Needs attention** panel below — highlighted when non-zero. |

## Needs attention

A live panel of things worth a human looking at — each item has a severity dot (warning / info / failure),
a title, an optional detail line, and a direct link to go fix it. When there's nothing to flag, it simply
says "✓ Nothing needs your attention" rather than showing an empty list.

## Get CrudAdmin ready

A checklist of setup steps, with a progress bar (`N of Total` required items done). Items you haven't done
yet show a **Do it** link straight to the right screen; optional items are marked as such; completed items
collapse into a "N done" expandable section so the checklist doesn't clutter the dashboard once you're past
initial setup.

## Recent activity

The latest entries from the audit log — time, event type, severity badge, actor, resource, and
success/failure outcome — with a **View all** link straight to [Audit logs](audit-logs.html). If
nothing has happened yet, it says so plainly instead of showing an empty table.

## Tenants table

Every tenant, at a glance: name and slug, sign-in address, database count, managed-database count, a health
badge, and when it last had any activity.

| Health badge | Meaning |
|---|---|
| ok | Normal. |
| No database | Created, but hasn't registered a database yet. |
| No config | Has a database, but hasn't generated a dashboard config yet. |
| Suspended | Deliberately disabled. |

A **Manage tenants** link goes straight to the [Tenants](tenants.html) screen. If there are no tenants yet,
the table is replaced with a direct **Create a tenant** prompt.

## Generate Test JWT

A button in the page header for minting a real, signed issuance-key-style JWT on the spot, for testing —
without needing to wire up your own backend's token-issuing call first.

1. **Subject (User ID)** — required; any string identifying the test user.
2. **Tenant ID** — optional, scopes the token to one tenant.
3. **Roles** — comma-separated (e.g. `Reader, Writer`).
4. **Databases** — multi-select from every registered database; leaving all unchecked issues a token with
   no database scope.
5. **Expires in (minutes)** — 1 to 1440, defaults to 60.

The result shows the signed token (with a **Copy** button), a ready-to-run `curl` example, and the decoded
token payload — so you can both use it immediately and see exactly what claims it carries.

{: .note }
> This is a genuinely valid, signed token against the platform's real signing key — treat it the same way
> you'd treat any other issuance-key-minted JWT. It's a convenience for testing your own integration, not a
> separate "fake" token type.
