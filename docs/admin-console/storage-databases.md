---
title: Storage & data
parent: Admin console
nav_order: 5
---

# Storage & data
{: .no_toc }

`/admin/storage` — where CrudAdmin keeps its own data, and the one thing to do about it.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

This page is about the platform's **own** storage — tenants, keys, roles, settings, audit history — not
your tenants' registered MySQL databases (those live under [Tenants](tenants.html) and each tenant's own
[Databases](../tenant-console/databases.html) tab).

## Your data lives here

The current engine is the first thing on the page, not buried among secondary cards: an engine badge
(**SQLite** or **MySQL**), the file name or host, the engine version, and a plain summary of what it holds
(admin count, tenant count, registered-database count, or "set up and empty so far"). Below that, one note
on what backing it up actually means:

- **SQLite** — the data is a single file in the data folder (listed in the collapsed sections below); back
  that file up along with the recovery kit.
- **MySQL** — the data is in a database on a server; back it up with that server's own backup tooling. The
  recovery kit covers only the *means to open it* (the password + keys), not the data itself.

### Switching to MySQL

While the store is SQLite, this card's one action is **Switch to MySQL →**, which opens a popup — not a
separate page — walking 5 steps: test connection, create schema, copy data, update env vars, restart. It
copies every row into the target MySQL server but **does not restart the app or switch it over
automatically** — you're given the exact environment variables to set
(`CRUDADMIN_DB_ENGINE`/`CRUDADMIN_MYSQL_CONNECTION_STRING`, see
[Deployment → Environment variables](../deployment/environment-variables.html)), then you restart the app
yourself — a half-migrated in-place swap would be worse than a clean restart onto a fully-copied store.

Once the store is already MySQL, this same card simply shows **✓ Running on MySQL — nothing to migrate**
instead of the button — there's no dead-end form to fill in only to be refused.

{: .known-issue }
> Moving back from MySQL to SQLite isn't supported today — this is a one-direction migration.

## Also on this page

Three secondary sections, collapsed by default except Recovery kit (which stays open whenever it genuinely
needs your attention):

- **Recovery kit** — CrudAdmin keeps the password to its own data in `bootstrap.db`, locked together with
  the `keys` folder. **Both are needed to open the data at all**, even though the actual data sits safely
  in your database. If either is lost, CrudAdmin can't open its own data, and that cannot be undone. This
  section either prompts you to **Download recovery kit** (super-admin-only, bundling both pieces plus a
  plain-language guide) or, once downloaded or explicitly acknowledged as "I already keep these another
  way," confirms that and when. Download a fresh kit any time you change where CrudAdmin keeps its data, or
  re-run auto-provisioning.

  {: .warning }
  > The kit contains real secrets. Store it like a password — a password manager's file storage, or an
  > encrypted disk. Never email it, and never commit it to a code repository.

- **SQLite databases** — a browsable list of every single-file SQLite database found on the server, each
  with its size, last-changed time, and status (**Registered**, **Found, not registered**, or **Missing**).
  A super-admin-only **🗄 Open SQLite browser** button opens a dedicated in-browser SQL browser/editor via a
  signed handoff — changes made there write straight to the file, so back it up first. New files are
  registered from inside that browser itself, not from this page.

- **Everything CrudAdmin keeps** — a full table of every file CrudAdmin's data folder contains, grouped
  into **Keep and back these up**, **Can be rebuilt or is safe to lose**, and **Leftovers and things not
  recognised** (removable directly from this table, with confirmation). Each row shows the file/folder
  name and size, which part of the product owns it, why it exists, what happens if it's lost, and whether
  it belongs in your backup. A separate "Kept next to the program, not in this folder" group lists anything
  a bundled package insists on placing elsewhere, if any package currently does.

## Backing this up in practice

- **SQLite store**: back up the data folder (per the file table above) plus the recovery kit, with your own
  filesystem backup tooling — that combination is a complete copy of everything the store holds.
- **MySQL store**: back up via that MySQL server's own backup/replication tooling, plus the recovery kit —
  the kit is still required even on MySQL, since it's what makes the connection to that server openable in
  the first place.
