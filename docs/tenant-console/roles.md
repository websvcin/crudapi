---
title: Roles
parent: Tenant console
nav_order: 7
---

# Roles
{: .no_toc }

Roles are bundles of permissions. The platform defines the permissions; you decide who gets which role.
{: .fs-6 .fw-300 }

---

## Ready-made roles

The five [built-in roles](../concepts/authentication-and-credentials.html#roles) — Reader, Writer, Editor,
and (if your platform administrator has allowed it) Database admin and Owner. Shown here for reference;
they can't be edited.

## Your own roles

If your platform administrator has turned on custom roles for this tenant (see
[Admin console → Access model → Tab 3](../admin-console/access-model.html#tab-3--tenant-guardrails)),
**+ New role** appears and lets you combine read/add/change/delete/see-structure permissions into a role
scoped only to *this* tenant. A custom role can never include configuration generation, running scripts,
loading starter data, or the `*.*` wildcard — those stay reserved for the built-in Database admin/Owner
roles regardless of this setting.

## How a role actually applies

A role only says **what** may be done. **Which database(s)** it applies to is chosen separately, on the
[Keys](keys.html) tab, when a specific key or token issuance key is created — the exact same role can be
given to one key limited to a single database and to another that reaches every database in the workspace.

With LoginLink, a signed-in person's roles arrive encoded directly in their token, using these same role
names — there's no separate role-assignment step on the data-API side for LoginLink users.

Row policies for a role are configured per-database from the [Databases](databases.html) tab, not here —
see [Features → Row policies](../features/row-policies.html).
