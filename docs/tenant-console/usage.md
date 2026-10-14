---
title: Usage
parent: Tenant console
nav_order: 6
---

# Usage
{: .no_toc }

Who called this workspace, how often, and from where. Detail is kept for 30 days.
{: .fs-6 .fw-300 }

---

A period switch (24 hours / 7 days / 30 days) drives every section below.

## Stat tiles

Top-line request counts for the selected period.

## By credential type

A breakdown of requests per credential type (API key, Basic auth, issuance-key JWT, LoginLink token) —
click one to filter everything below it to just that type.

## By user

Per-user request counts (for JWT/LoginLink traffic specifically, since that's the credential type tied to
an actual person) — click a user to see only their calls in the sections below.

## Endpoints

Which routes are actually being hit, under whatever filter is currently active (credential type and/or
user) — the fastest way to answer "is anyone still using this old integration" before deciding to revoke a
key.

## Recent calls

The raw recent-call log for the current filter — timestamp, credential, endpoint, and outcome.

{: .note }
> This is distinct from [Admin console → Audit logs](../admin-console/audit-logs.html), which is the
> security/action trail (who changed what). Usage here is specifically about data-API **call volume** —
> rate-limit-relevant traffic, not configuration changes.
