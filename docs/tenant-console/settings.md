---
title: Settings
parent: Tenant console
nav_order: 10
---

# Settings
{: .no_toc }

Basic workspace details, and the raw LoginLink management credential.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Workspace details

- **Display name** — shown across both consoles.
- **Webhook URL** — where CrudAdmin sends outbound event notifications for this workspace, if configured.
- A reference line showing the token/issuance address relevant to this workspace.

## LoginLink management credential

The raw connection details behind the [Access](access.html) tab's "Sign in with LoginLink" option:
organisation slug (or blank for the platform root organisation — not an error, just "no separate sub-path"),
Client ID, and Client secret (write-only; never shown again after saving).

| Action | Does |
|---|---|
| **Check it still works** | Verifies the stored credential is still accepted by LoginLink. |
| **Replace** | Save a new Client ID/secret, overwriting the stored one. |

{: .known-issue }
> "Check it still works" can report **"No management credential is stored"** even when one genuinely is —
> the check also requires a LoginLink **app** to exist for this connection, and conflates "no credential"
> with "no app yet" into the same message. If you've definitely saved a credential and still see this,
> the LoginLink app side of the setup likely still needs completing (see
> [Features → LoginLink integration](../features/loginlink-integration.html)), not the credential itself.

{: .note }
> This is the **management** credential — it lets CrudAdmin administer the LoginLink side of the
> connection (creating the sign-in app, syncing roles). It's distinct from an individual person's own
> LoginLink login, and distinct from the data-API "LoginLink token" credential type described on the
> [Keys](keys.html) tab.
