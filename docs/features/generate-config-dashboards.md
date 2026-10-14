---
title: Generate Config & dashboards
parent: Features
nav_order: 5
---

# Generate Config & dashboards
{: .no_toc }

Turning a raw database schema into the single JSON document that drives an auto-built admin UI — sidebar,
tables, forms, and a dashboard — with no hand-written frontend code.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Triggering it

From either console's **Actions → Generate Config** button on a database (see
[Admin console → Tenants](../admin-console/tenants.html) or
[Tenant console → Databases](../tenant-console/databases.html)), or directly via
`GET .../database/generate-config` — see
[API reference → Database & schema endpoints](../api-reference/database-endpoints.html#generate-a-dashboard-config--get-databasegenerate-config).

## What it produces, per table

For every table in the database, an entity object with: searchable fields, display/create/edit field lists
(each mapped to a sensible UI type — text, number, date, boolean, enum dropdown, or a foreign-key
`relation`), a suggested icon, pagination defaults, a default sort, and — if you asked for it via the
`includeBulkUploadFrom*` query parameters — [bulk import](bulk-import.html) entry points wired to that
table.

Tables are also grouped into logical sections with a name, description, and icon, using an AI call (your
configured [AI provider](../admin-console/settings.html#ai)) to produce sensible groupings and
descriptions from column names and relationships — this is a real, metered cost per call (see
[known limitations](../reference/known-issues.html#cost-and-reliability-of-generate-config) below).

## The four dashboard blocks

Independent of the AI step — pure heuristic matching against column names and types, generated fresh every
time:

| Block | Appears when |
|---|---|
| **Summary** | Always — groups each table by its best-guess status/category column (exact match on `status`, `category`, `stage`, `type`, `state`, `role`, `priority`, `level`, or `group`; falls back to a foreign key or the first column). |
| **Table** | Per table, if it has any plausible date column (`created_at`, or anything typed date/timestamp) — a row count, a "recent records" widget, and an entity view. |
| **Kanban** | Only if at least one table has a status/stage column. Column values are matched against the fixed labels "To Do," "In Progress," "Done" — this only looks right if your schema's real values happen to match those exact strings. |
| **Calendar** | Only if at least one table has both a plausible start-date and end-date column. |

## The Config Visualizer

A companion visual tree/field editor over the generated config, reached via **Actions → Config Column →
Open editor** on a database, through the same signed single-sign-on handoff pattern as
[rate limiting](rate-limiting.html). Lets you adjust the generated structure by hand without editing raw
JSON directly.

## Publishing

If [GitHub integration](../admin-console/settings.html#github) is configured, the generated config is also
pushed to your repository at `{prefix}/{tenant-slug}/{db-key}.json`. Generation and publishing are reported
**separately** — a failed GitHub push doesn't discard the generated config; the response's HTTP status is
`502` with the generated config still included, plus the specific push failure reason, so "generated but
not published" is never mistaken for a total failure.

{: .known-issue }
> Entities generated today carry hardcoded role names (`"root"`) in their `accessRoles`/permission fields,
> rather than this platform's actual role names (Reader/Writer/Editor/Database admin/Owner, or your own
> custom roles). If your frontend compares a signed-in person's real role against these fields, every
> button currently shows for everyone until this is addressed on the generation side. See
> [Reference → Known limitations](../reference/known-issues.html) for status.
