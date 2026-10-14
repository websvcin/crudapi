---
title: Setup wizard
parent: Deployment
nav_order: 3
---

# Setup wizard
{: .no_toc }

`/setup` — the one-time flow every fresh install redirects to, until a real first admin account exists.
Most steps can be skipped and finished later.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

The wizard tracks progress across page reloads and browser closes — nothing above is ever lost mid-way;
reopening `/setup` picks up exactly where you left off.

## 1. Bootstrap

Decides where `bootstrap.db` — a small settings file remembering where the platform's data actually lives —
comes from: **Fresh install** (creates a new, empty one right away) or pointing at an existing one from a
prior install. This step only resolves *that one file*; the real storage choice comes in step 3.

## 2. Auto-provision (optional)

Configure a provisioning account (MySQL server, Plesk, or cPanel — see
[Admin console → Settings → Provisioning account](../admin-console/settings.html#provisioning-account)) so
CrudAdmin can later create real databases on tenants' behalf. Entirely skippable — tenants can always bring
their own existing databases instead.

## 3. Storage

Choose SQLite (default, a local file) or MySQL for the platform's **own** admin registry — separate from
any tenant's data database. Choosing MySQL here can create the database and migrate the schema inline as
part of this step, without requiring a separate restart, when the target server is reachable and the auto-
provisioning account from step 2 can reach it.

## 4. Admin

Create the real first platform administrator: name (optional), email (this is the sign-in — there's no
separate username), and password. This account has full super-admin access. There is no default account
anywhere in the product — this step is mandatory.

## 5. Connections (optional)

A checklist pointing you at [Host Settings](../admin-console/settings.html#host-settings) for GitHub
(publishing generated configs), an AI provider (generating them), and LoginLink (tenant/admin sign-in) —
all optional, and all editable later from that same screen. Your current public address is confirmed
automatically from the request that's loading this very page.

## 6. Tenant (optional, skippable)

Create your first tenant right here (name + slug) or skip and do it later from
[Admin console → Tenants](../admin-console/tenants.html) — functionally identical either way.

## 7. Health check

Runs the platform's own health checks and shows the result — warnings don't block you from finishing. If
the [recovery kit](../admin-console/storage-databases.html#recovery-kit) hasn't been downloaded yet, this
is flagged here with a direct link to do it before you consider the install truly done.

**Finish and open dashboard** completes setup and takes you into [`/admin`](../admin-console/dashboard.html)
for the first time.

## Upgrading from before setup existed

An install that predates the setup wizard is automatically grandfathered in — it's never forced through
this flow retroactively.
