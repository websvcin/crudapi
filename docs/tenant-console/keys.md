---
title: Keys
parent: Tenant console
nav_order: 5
---

# Keys
{: .no_toc }

Three ways into your data. Pick the one that fits who's asking.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Each credential type is a small tile that opens its own full list — the tile stays compact no matter how
many keys you actually have. A **📘 API reference** link opens the published docs directly.

## 1 · Your servers

A server you control calls the data API directly, always acting as itself at one fixed role.

| Type | Header | Notes |
|---|---|---|
| **API key** | `X-API-Key` | The everyday case. |
| **Basic auth** | `Authorization: Basic` | For tools that can't send a custom header — use only when nothing else will work. |

Each list shows what the key can do, which database(s) it reaches, when it was last used, and its expiry.
**+ Add** creates a new one — shown once; copy it immediately.

## 2 · Your app's users

Your app's own end-users read data as themselves, via a short-lived JWT. Two ways to get them one — which
to use is the only real difference:

| Type | Who issues it | Why pick this one |
|---|---|---|
| **Token issuance key** | Your own backend, after your existing sign-in succeeds | Works today with whatever sign-in system your app already has — your users don't need a LoginLink account at all. The key itself never reads data; it can only mint tokens with the roles and database access you've configured. |
| **LoginLink token** | LoginLink, when the user signs in there | Your app's users sign in directly at LoginLink; the data API checks their LoginLink token, confirms they're on your list, and which databases/roles they may reach. |

Each **token issuance key**'s list shows what it's locked to, which roles it may grant, its longest allowed
login duration, and expiry. A **🧪 Try it** action on any issuance key mints a real, short-lived test token
on the spot — pick a subject id, roles, database(s), and expiry minutes; the result shows the token, a
ready `curl` example, and the decoded payload. The key's own limits silently narrow anything you request
beyond what it's actually allowed to issue — what comes back is always the honest, final answer.

**LoginLink token** setup is managed from this same tile — see
[Features → LoginLink integration](../features/loginlink-integration.html) for the full connect flow.

{: .note }
> Need someone to sign into the **console itself** (not your data)? That's a console key, on the
> [Access](access.html) tab — a different, much narrower credential than anything on this page.
