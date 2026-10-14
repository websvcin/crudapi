---
title: Access model
parent: Admin console
nav_order: 4
---

# Access model
{: .no_toc }

`/admin/permissions` — labeled **Permissions** in the sidebar. The rules every tenant works within; nothing
on this page gives anyone access by itself, it only sets what tenants are *allowed* to hand out.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The three-layer model, in one minute

A **permission** is one thing that can be done (e.g. reading data). A **role** is a named bundle of
permissions. A **grant** gives a role to a key or a person **at a scope**: one database, all of a tenant's
databases, or the whole platform. Permissions and roles say *what*; scope says *where* — so the same role,
even Owner, is a safe everyday choice on one tenant's own databases and a very different decision on the
whole platform.

1. **Which databases?** — set on each key, checked on every request; a tenant's key can never reach another
   tenant's database.
2. **Which actions?** — the role's permissions (this page).
3. **Which rows?** — an optional row policy narrowing a role further, per tenant (see
   [Features → Row policies](../features/row-policies.html)).

The page has five tabs: **Permissions and tiers**, **Built-in roles**, **Tenant guardrails**, **Wide
grants**, and **Advanced**.

## Tab 1 — Permissions and tiers

Every permission the platform knows about, in one table: what it lets someone do, its **tier**, who may
hand it out, what could go wrong, and which roles currently hold it (computed live from what's actually
enforced, so this table can never disagree with reality).

| Tier | Meaning | Who may grant |
|---|---|---|
| 1–2 | Everyday / removal actions (read, create, update, delete) | Tenants, through whichever roles you allow |
| 3 | Sensitive (e.g. seeing structure, generating configuration, running scripts) | Needs your approval, per tenant |
| 4 | `*.*` — everything, including anything added later | Needs your approval; a platform-scoped grant needs platform-only approval |

A standing banner on this tab calls out `*.*` specifically: it grants total access — every action on every
table, plus every database action, including anything added in the future. For everyday full access, the
page recommends **Editor** instead; reserve `*.*` (the Owner role) for a tenant you trust, on that tenant's
own databases, or on the platform for your own operators.

Permissions created by hand for a specific table (rather than the five standard actions) are flagged and
summarized here, with the full list on the Advanced tab.

## Tab 2 — Built-in roles

The same five roles described in
[Concepts → Authentication & credentials](../concepts/authentication-and-credentials.html#roles)
(Reader, Writer, Editor, Database admin, Owner), shown here as an exact matrix — ten columns: Read, Add,
Change, Delete, See structure, Config 🛑, Starter data 🛑, Scripts 🛑, Everything 🛑, Tier — plus how many
keys and people currently hold each one. Built-in roles are locked; you can't edit what they hold, only
whether a tenant is allowed to hand one out (Tab 3).

Below the matrix, each role gets its own card with a plain-language sentence of exactly what it allows —
this is the same wording a tenant sees when deciding which role to give a key. If any tenant has built its
own custom role (Tab 3 → "Roles a tenant may build itself"), those appear in a second table underneath,
each labeled with its owning tenant.

## Tab 3 — Tenant guardrails

This is where you actually decide what tenants may do with their own workspace, and it's built around one
**default** that applies to every tenant, plus **exceptions** for the few that need something different.

**Default for every tenant** — a form with three independent choices:

1. **Roles a tenant may hand out** — a checkbox per built-in role. Reader/Writer/Editor are on by default;
   Database admin and Owner are off (each marked "needs your approval" until you turn them on), and Owner
   additionally shows a standing warning explaining it's full control of that tenant's own databases.
2. **Where a tenant may grant access** — either "Chosen databases only" (default — every key must list its
   databases) or "Also all of its own databases" (adds an "All my databases" choice when a tenant creates a
   key; such a key automatically reaches every database that tenant owns, including ones added later, and
   shows up on the Wide grants tab).
3. **Roles a tenant may build itself** — Off (default) or On. When on, the tenant's own Roles screen gets a
   "New role" button that can combine read/add/change/delete/see-structure in any mix — never configuration,
   scripts, starter data, or `*.*`, regardless of this setting.

A live preview below the form spells out, in plain language, exactly what a tenant will see and be able to
do under the choices as currently checked — before you save anything.

**Exceptions** — tenants that don't follow the default. Search for a tenant by name or id to give it its
own settings (this doesn't show every tenant, by design — it stays short regardless of how many tenants you
have). Each exception can be edited or reverted to "Use the default" at any time.

{: .note }
> **Changing a guardrail never revokes access a key already has.** If you tighten a default or an
> exception, the save response lists exactly which existing keys now hold more than the tenant is currently
> allowed to grant — they keep working as-is, but you can see them and decide whether to act.

## Tab 4 — Wide grants

Every credential that reaches **more than one database of a single tenant** — either all of that tenant's
databases, or (for a platform key) the whole platform. This is the review list: narrower, single-database
keys never appear here, since they're not what a guardrail audit needs to focus on.

Each row shows who holds the grant, which tenant (blank for a platform-wide key), what it can do, its
reach (tenant-wide or platform), how many times it's been used, and when it was last used — "never" means
exactly that, not missing data. A **Review** link opens the right keys page to narrow or revoke it. A
platform key marked for the whole platform with no tenant attached is flagged as having no clear owner.

If nothing qualifies, the tab simply says so — "No key reaches more than one database of a tenant."

## Tab 5 — Advanced

The raw permission list — every `{resource}.{action}` string that exists, its description, and which roles
hold it. `*` and `database` permissions are locked (can't be deleted from here); anything created by hand
can be removed, with a warning that any role holding it loses it immediately, and keys using that role may
start getting `403`s.

Two creation tools, used rarely:

- **Add permission** — a single `resource` + `action` (+ optional description).
- **Bulk create** — one `resource` (a real table name) and a checklist of the four standard actions
  (create/read/update/delete), previewing the exact permission strings before creating them.

{: .known-issue }
> Table-specific permissions created here are matched by exact spelling against your database's real table
> names — a typo (`emplyee` instead of `employee`) silently creates a permission that grants nothing,
> because no table will ever match it. Prefer letting a role's table-scoped access be chosen from each
> database's own table list elsewhere in the console, rather than typing names here, for exactly this
> reason.
