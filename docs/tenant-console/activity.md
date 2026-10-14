---
title: Activity
parent: Tenant console
nav_order: 8
---

# Activity
{: .no_toc }

Everything that happened in your workspace. Other workspaces never appear here.
{: .fs-6 .fw-300 }

---

## Filters

| Filter | Options |
|---|---|
| **Database** | All databases, or one specific one. |
| **Action** | All, or one of: Sign-ins, API keys, Basic logins, Issuance keys, Roles, Databases, Allowed origins (CORS), Row policies, Platform support view. |
| **When** | Any time, last 24 hours, 7 days (default), 30 days, or a custom date range. |
| **Result** | Any, Worked, Failed. |
| **Search** | Free text against who or what. |

{: .note }
> **"Database" finds events tied to one specific database** — registering it, its allowed web origins, and
> row policies scoped to it. Actions that apply to the whole workspace (signing in, building a role) aren't
> tied to any single database and won't appear when this filter is narrowed.

## The table

When, who, what, and the result for every logged event.

## Platform support view entries

When a platform administrator opens a [Support view](../admin-console/tenants.html#support-view) of your
workspace, it's logged right here, tagged **platform**, together with the reason they gave — support
access into your workspace is never invisible to you.

## Exporting

**Export CSV** downloads the currently-filtered result set — useful for retaining history beyond the
UI's own retention window, or for feeding into your own compliance tooling.
