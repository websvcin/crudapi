---
title: Database & schema endpoints
parent: API reference
nav_order: 3
---

# Database & schema endpoints
{: .no_toc }

Introspecting a database's schema, generating a dashboard config, loading a starter bootstrap config, and
running schema-changing scripts.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Base address

```
/api/v1/{tenant}/{database}/database/...
```

These are database-level actions — they operate on the database's structure, not on rows of a specific
table — so each needs a permission on the special `database` resource rather than a `{table}.{action}`
permission.

## Introspection (read-only)

| Endpoint | Returns |
|---|---|
| `GET /database/info` | Basic information about the connected database. |
| `GET /database/tables` | Every table CrudApi can see. |
| `GET /database/tables/{table}/columns` | A table's columns, with type, key, and foreign-key metadata. |
| `GET /database/relationships` | Foreign-key relationships across the whole database. |

Permission needed: `database.read` — granted to **Reader and above**, since none of these expose anything
beyond what the CRUD endpoints already imply.

```bash
curl -H "X-API-Key: drk_..." "https://your-host/api/v1/acme/reports/database/tables"
```

## Generate a dashboard config — `GET /database/generate-config`

Produces a structured configuration describing the database's tables, fields, suggested relationships, and
(optionally, via `includeBulkUploadFromFile`/`...FromFilter`/`...FromAPI` query parameters naming tables)
which bulk-import sources to wire up per table — the input the
[Config Visualizer](../features/generate-config-dashboards.html) and dashboard tooling consume. If the
platform has GitHub integration configured, the result is also pushed to the connected repository.

```bash
curl -H "X-API-Key: drk_..." \
  "https://your-host/api/v1/acme/reports/database/generate-config?includeBulkUploadFromFile=widgets,orders"
```

Permission needed: `database.config` — granted to **Database admin and Owner**.

Generating the config and publishing it to GitHub are reported **separately**: the config itself is always
generated and saved, but if the GitHub push fails (bad token, no network, not configured), the response's
HTTP status is **502**, with the body still showing the generated config plus the specific push error — so
a caller never mistakes "generated but not published" for either a clean success or a total failure.

## Load a starter bootstrap config — `POST /database/bootstrap-db/{database}`

Reads `bootstrap-db/{database}.json` from the platform's connected GitHub repository and inserts every
record it contains into the matching tables — the counterpart to **Generate a dashboard config**, for
getting a freshly registered, empty database into a demo-ready state from a config someone already
committed.

Permission needed: `database.bootstrap` — granted to **Owner only**. Unlike the other three actions here,
this one writes rows across potentially every table in the database at once, without going through each
table's own per-row checks, so it's reserved for the most trusted role.

## Run a script — `POST /database/script`

Runs a schema-changing SQL script — the only one of the four database actions that can alter structure, so
it goes through a dedicated safety check before anything touches the real database.

Permission needed: `database.script` — granted to **Database admin and Owner**.

### Step 1 — dry run

Send the script with `"dryRun": true` first. Nothing is executed; you get back what the script *would* do:

```bash
curl -X POST -H "X-API-Key: drk_..." -H "Content-Type: application/json" \
  -d '{"script":"ALTER TABLE widgets ADD COLUMN notes TEXT; DROP TABLE old_cache;","dryRun":true}' \
  "https://your-host/api/v1/acme/reports/database/script"
```

```json
{
  "allowed": true,
  "destructive": 1,
  "fingerprint": "a93f...",
  "statements": [
    { "index": 1, "text": "ALTER TABLE widgets ADD COLUMN notes TEXT", "kind": "ALTER TABLE", "level": "safe", "explanation": "..." },
    { "index": 2, "text": "DROP TABLE old_cache", "kind": "DROP TABLE", "level": "destructive", "explanation": "..." }
  ]
}
```

Every statement is checked individually and classified **safe**, **destructive** (drops or deletes
something), or **refused**. Only structural statements are ever allowed at all — `CREATE`/`ALTER`/`DROP`/
`RENAME` on tables, views, and indexes. Everything else is refused outright, including: any data statement
(`INSERT`/`UPDATE`/`DELETE`/`SELECT ... INTO`), `CREATE TABLE ... SELECT` (it copies rows, which is a data
change), `GRANT`/`REVOKE`, `SET`, creating/altering/dropping users or roles, creating/dropping/altering a
*different* database, and `... INTO OUTFILE`. If even one statement is refused, the whole script is blocked
— `allowed: false` — and none of it runs.

### Step 2 — run it for real

Send the same script again without `dryRun`, this time including:

- `fingerprint` — exactly what the dry run returned. If the script text has changed at all since, the
  fingerprint won't match and the run is blocked (`FingerprintMismatch`) — an approval never silently
  carries over to an edited script.
- `confirmedDestructive` — the index of **every** statement the dry run marked `destructive`, one by one.
  Confirming one destructive statement never implicitly confirms another.
- `acknowledgeBroadRights` — only needed if the database's own connection user turns out to have rights
  beyond this one database (checked live against the real grants at run time, never assumed from how the
  connection was set up). If so, the platform can't guarantee the script's blast radius is limited to this
  database, and running it anyway requires this explicit acknowledgement.

```bash
curl -X POST -H "X-API-Key: drk_..." -H "Content-Type: application/json" \
  -d '{"script":"ALTER TABLE widgets ADD COLUMN notes TEXT; DROP TABLE old_cache;",
       "fingerprint":"a93f...","confirmedDestructive":[2],"acknowledgeBroadRights":false}' \
  "https://your-host/api/v1/acme/reports/database/script"
```

| Block reason | HTTP status | Meaning |
|---|---|---|
| `EngineNotSupported` | 400 | Scripts only run against MySQL today. |
| `Refused` | 400 | At least one statement isn't on the allowed list — see above. |
| `FingerprintMismatch` | 409 | The script changed since the dry run; dry-run it again. |
| `NeedsDestructiveConfirmation` | 409 | One or more destructive statements aren't in `confirmedDestructive`. |
| `NeedsBroadRightsAcknowledgement` | 409 | The connection user's rights reach beyond this database; set `acknowledgeBroadRights: true` to proceed anyway. |

**Statements are not wrapped in one all-or-nothing transaction.** They run in order; if one fails partway
through, the statements before it have already been applied and stay applied — the response tells you
exactly which statement index failed so you know what state the database is actually in.

Every script run (or refusal) is written to the audit log with the fingerprint, statement count, and
outcome, whether it succeeds, fails partway, or is blocked before it starts.

## Errors

Same JSON error shape as [CRUD endpoints](crud-endpoints.html#errors) — `error` + `errorDescription` (or,
for script-specific blocks, `error` + `message` as shown above). A role missing `database.config`,
`database.bootstrap`, or `database.script` specifically — even if it holds every other database
permission — gets a plain `forbidden_action` naming the exact missing permission. See
[Admin console → Access model](../admin-console/access-model.html) for how a platform administrator decides
which tenants may hand out Database admin or Owner at all.
