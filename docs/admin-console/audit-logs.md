---
title: Audit logs
parent: Admin console
nav_order: 7
---

# Audit logs
{: .no_toc }

`/admin/auditlogs` — the complete security event trail across every tenant.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Filters

| Filter | Does |
|---|---|
| **Tenant** | Type-ahead search; leave blank for every tenant. |
| **When** | Any time, last hour, 24 hours, 7 days, 30 days, or a custom date range. |
| **Who** | Free text against admin email or key name. |
| **Event type** | A long fixed list — `admin.login`, `admin_user.created`, `database.created`, `role.created`, `permission.created`, `tenant.access_guardrails`, `host_setting.updated`, `host_setting.secret_updated`, `jwt.admin_issued`, `row_policy.created`, `signing_key.rotated`, `signing_key.force_retired`, `issuance_key.created`, `token.issued`, `basic_auth.created`, and more. |
| **Severity** | Info / Warning / Critical. |
| **Outcome** | Success / Denied / Failure / Error. |
| **Show** | Page size — 50/100/250/500. |

**Clear filters** resets everything back to "Any time, every tenant." **Auto Refresh** keeps the table
live-updating without a manual reload — useful when actively watching for an event as it happens (e.g.
testing a credential change in another tab).

## The table

Each row: timestamp, event type, severity badge, actor (who did it), resource (what it touched), outcome
badge, and a details column for anything event-specific. Less critical columns (IP, action detail, actor,
severity) progressively hide on narrower screens so the table stays usable without horizontal scrolling on
a laptop.

## What gets logged

Every authenticated action across the platform writes an entry here — both successes and refusals. This
includes admin sign-ins, every tenant/database/role/permission/row-policy change, host-setting changes
(secrets are logged as changed, never with their value), signing-key rotation, issuance-key and token
issuance, and every data-API authorization decision (see
[Concepts → Architecture](../concepts/architecture.html#request-pipeline-in-order) — the request pipeline
writes to this same log at the authorization step, on both success and denial).

{: .note }
> A tenant sees its **own** activity from its own console (see
> [Tenant console → Activity](../tenant-console/activity.html)) — this platform-wide view additionally
> covers every tenant plus platform-level events (admin sign-ins, host settings, signing keys) a tenant's
> own view never shows.

## Reading the audit log programmatically

The **Audit** capability on a [Platform Developer Key](developer-api.html) exposes `GET
/api/admin/audit-logs` (and a purge endpoint) for scripted access — the same filters available here are
available as query parameters on that endpoint.
