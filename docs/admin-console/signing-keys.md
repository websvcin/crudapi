---
title: Signing keys
parent: Admin console
nav_order: 10
---

# Signing keys
{: .no_toc }

`/admin/signingkeys` — rotate the JWT signing key safely, with a grace window so already-issued tokens keep
working.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

{: .note }
> **This is not about admin or tenant sign-in.** It's specifically what signs the JWTs an
> [issuance key](../concepts/authentication-and-credentials.html#issuance-key-jwt) hands a tenant's own
> end-user after their backend mints them a token. Rotating a key here never affects API client keys, Basic
> Auth logins, or console keys — only future JWT-based logins.

## The key-ring model

Only **one** key is ever active at a time. Rotating:

1. The current active key becomes **previous** — it keeps validating tokens already signed with it for a
   grace window (`jwt.key_retention_hours`, configurable in
   [Settings → Security Settings](settings.html#security-settings), 24 hours by default).
2. A brand-new key becomes **active** and signs every new token from this point on.
3. Existing tokens signed under the previous key keep working until they naturally expire or the grace
   window passes — whichever comes first.

## The key table

Every key's id, algorithm, status (**active** / **previous** / **retired**), created/rotated/retired
timestamps, and a per-row action:

| Status | Available action |
|---|---|
| active | None — the current signer. |
| previous | **Force Retire** — bypasses the grace window immediately. |
| retired | **Delete** — permanently removes the retired key record. |

## Actions

- **🔄 Rotate Key** — makes a new key active, demotes the current one to previous. Confirmed before
  running, since it's a real cutover.
- **Retire Old Previous** — retires every previous key *older than* the grace window in one action. If
  nothing yet qualifies, it tells you plainly rather than silently doing nothing.
- **Force Retire** (per previous key) — retires a specific key **immediately**, bypassing the grace window.

{: .warning }
> Force-retiring a key immediately **invalidates every still-valid token signed by it** — anyone holding
> one gets a hard authentication failure on their next request, not a graceful re-login. Use this only when
> a key is suspected compromised; for routine cleanup, use **Retire Old Previous** instead and let the grace
> window do its job.
