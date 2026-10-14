---
title: Run scripts
parent: Features
nav_order: 6
---

# Run scripts
{: .no_toc }

Running a schema-changing SQL script against a registered database, from inside the console — with a
mandatory dry run and explicit confirmation for anything destructive.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Opened from a database's **Actions → Run a script** button, on either console. Requires the
**Database admin** or **Owner** role (`database.script`) — see
[Access model](../admin-console/access-model.html).

## 1. Check it (dry run)

Paste the SQL and check it first. Every statement is classified **safe**, **destructive** (drops or
deletes something), or **refused** — only structural changes are ever allowed (`CREATE`/`ALTER`/`DROP`/
`RENAME` on tables, views, indexes). Data statements, privilege changes, and cross-database statements are
always refused; see the full list in
[API reference → Database & schema endpoints](../api-reference/database-endpoints.html#run-a-script--post-databasescript).
If anything is refused, the whole script is blocked — nothing runs, not even the safe parts.

For each destructive statement, you confirm it **individually** — a single "confirm all" checkbox doesn't
exist by design, so approving a `DROP TABLE orders` can never accidentally also approve a
`DROP TABLE customers` three lines later.

## 2. Run it

Running requires the exact script text that was just checked (an edit invalidates the check — you'd need
to check again) and, if the database's own connection user turns out to have rights beyond this one
database, an explicit acknowledgement of that broader blast radius before it's allowed to proceed.

Statements run **in order, not as one all-or-nothing transaction** — if one fails partway through, the
ones before it have already been applied and stay applied. The result tells you exactly which statement
failed.

## "Execute here" — picking a credential on the spot

Rather than only showing you the script to copy and run elsewhere, the console lets you execute it directly
using a credential you provide in the moment:

- **🔑 API client key**
- **🔒 Basic Auth**
- **🎫 Issuance-key token**

The credential is sent once, for this one request, and is **never stored** by the page. This is a genuine
convenience for one-off schema changes without leaving the console, not a separate, lower-bar execution
path — the same dry-run/confirm/run flow and the same `database.script` permission check apply regardless
of which credential you pick here.

## Audit

Every check, refusal, and run is written to the [audit log](../admin-console/audit-logs.html) with the
script's fingerprint, statement count, and outcome — including partial failures.
