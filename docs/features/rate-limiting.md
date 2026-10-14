---
title: Rate limiting
parent: Features
nav_order: 3
---

# Rate limiting
{: .no_toc }

Per-database request limits, enforced before a request reaches any business logic.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Where it lives

Rate limiting is a companion package (a separate "RateLimitAdmin" surface) reached from a database's own
**Actions → ⏱ Rate limits** button, on both the admin console's Databases view and the
[tenant console's Databases tab](../tenant-console/databases.html). It opens in a new tab via a signed
single-sign-on handoff — there's no separate login step, and no separate account to manage.

## What it enforces

A primary requests-per-minute limit per database, plus any additional rules you add on top of it (e.g.
narrower limits for a specific credential). This sits at the same point in the
[request pipeline](../concepts/architecture.html#request-pipeline-in-order) as authentication — a request
over its limit is rejected before it reaches authorization or any table logic, so it can't rack up
database load just to be denied downstream.

## Scope

Rate limiting here is **per-database and per-credential** — it's about CRUD traffic volume, not a control
on platform/management actions like [Generate Config](generate-config-dashboards.html) (which today has no
built-in cost control at all — see [Reference → Known limitations](../reference/known-issues.html)).

## Data and retention

Rate limit configuration and usage history for this feature live in their own data directory, separate
from both the main platform store and from usage data shown on
[Tenant console → Usage](../tenant-console/usage.html). If you're self-hosting via Docker, make sure this
directory is included in your persistent volume — see
[Deployment → Docker](../deployment/docker.html#volumes) and
[Deployment → Environment variables](../deployment/environment-variables.html) for the exact variable that
controls where it's written.
