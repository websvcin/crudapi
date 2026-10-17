---
title: LoginLink integration
parent: Features
nav_order: 7
---

# LoginLink integration
{: .no_toc }

CrudApi integrates with LoginLink (a separate sign-in product) in **four distinct places** — easy to
conflate, so each is listed explicitly below with where it's configured and what it actually controls.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The four integration points

| # | What it's for | Configured at | Who it affects |
|---|---|---|---|
| 1 | Your **platform team** signing into `/admin` | [Admin console → Admin users](../admin-console/admins.html) | Only your own platform operators. |
| 2 | A **tenant's own team** signing into their console | [Tenant console → Access](../tenant-console/access.html) | Only that tenant's team members — never data. |
| 3 | A **tenant's end-users** signing into the tenant's own app, and that app reading data as them | [Tenant console → Keys](../tenant-console/keys.html) (data) + [Settings](../tenant-console/settings.html) (management credential) | The tenant's real application users. |
| 4 | **Auto-provisioning** a brand-new LoginLink organisation whenever a tenant is created | [Admin console → Tenants](../admin-console/tenants.html#new-tenant-provisioning-with-loginlink) | New tenants only, platform-wide switch. |

These are independent — connecting #1 has no effect on #2 or #3, and vice versa. A tenant can be fully set
up on #3 (their app's users sign in via LoginLink) while your own platform team (#1) still signs in with
plain passwords.

## Organisation vs. platform root

A LoginLink connection resolves to either a specific **organisation slug** (a sub-path under your LoginLink
server, e.g. `https://loginlink.example.com/acme`) or the **platform root** organisation itself (no
sub-path at all). For a tenant workspace's own team/data sign-in (points #2/#3), only an organisation slug
is allowed now — connecting to the platform root is refused with a clear message, since the root is meant
for your own platform team (point #1) and for auto-provisioning's own bookkeeping, not for a real tenant's
people. If a tenant workspace somehow ended up on the root organisation before this restriction existed,
it keeps working; it just can't be *reconnected* there going forward.

Relatedly, **one LoginLink organisation can only ever serve one tenant workspace.** Connecting an
organisation another workspace already uses is refused, naming that workspace. This is what makes it safe
for CrudApi to resolve a workspace purely from which organisation a token names (see
[CrudApi as a registered API](#crudapi-as-a-registered-api) below) — without it, a token addressed to
CrudApi itself couldn't be traced back to exactly one workspace.

## Auto-provisioning at tenant creation

When platform-wide provisioning is on (configured via the 3-step wizard on
[Tenants](../admin-console/tenants.html#new-tenant-provisioning-with-loginlink)), creating a new tenant
also creates a brand-new LoginLink organisation for it automatically, using a platform-held management
credential with only the `CreateTenants` permission. The new organisation's slug is whatever LoginLink
itself assigns — this is stored and used verbatim as the tenant's actual LoginLink issuer going forward
(a real bug fixed in this product: an earlier version discarded this real slug and always pointed the
tenant back at the platform root instead).

## Connecting an existing organisation (manual)

Either a platform administrator connects one on a tenant's behalf (from the Tenants page's LoginLink
modal), or — if allowed — the tenant connects its own from its own [Access](../tenant-console/access.html)
or [Settings](../tenant-console/settings.html) tab. Either path needs:

1. The organisation's **slug** — read from the address bar while signed into that LoginLink organisation
   (it can't be derived from a credential, so it must be typed correctly).
2. A **management credential** — created in that organisation's own API Access page, with **Manage apps**
   ticked.

## CrudApi as a registered API

Point #3's original design had a real limit: the data API only ever trusted tokens addressed to **one**
specific LoginLink app per workspace (the one CrudAdmin itself registered, or a separate "data app" created
from Keys) — so if your tenant's end-users actually sign in through *their own* app (a separate product
you've built, not CrudAdmin's), its tokens were never accepted, because they were never addressed to that
one trusted app.

CrudApi now also registers itself in LoginLink as its own **API** (a resource server, independent of any
one app's identity) inside each connected workspace's organisation — done automatically by CrudAdmin when
the workspace connects, and again whenever CrudApi's own identifier changes. Any app LoginLink has granted
access to that API can then request a token **addressed to CrudApi itself** (LoginLink's `resource`
parameter at `/token`), instead of needing to be the one specific app CrudAdmin already knows about. The
data API accepts either shape: a token for the one connected app (unchanged), or a token for CrudApi's own
identifier, resolved to the right workspace by which organisation it names.

- **CrudApi's identifier** — what an app asks for with `resource=`. Configurable at
  [Admin console → Settings → LoginLink server](../admin-console/settings.html#loginlink-server); left
  blank, it defaults to the host's own configured public address (`Public:BaseUrl`) — a fixed value, shown
  on that page as the value actually in effect, rather than silently varying by request. A calling app
  should read it from `GET /api/tenant/loginlink/data` (field `apiIdentifier`) rather than hardcode it —
  changing the identifier re-registers it everywhere already connected, and anything hardcoded would
  silently stop working.
- **Allowed apps (optional)** — a workspace can restrict which specific apps may call this way, beyond
  whatever LoginLink itself has granted. Unset (the default) means any app LoginLink granted the API.
- **"Register again"** — if registration ever fails (a transient LoginLink error, a credential that expired
  between attempts), retry it from [Tenant console → Access](../tenant-console/access.html) without
  reconnecting LoginLink itself. A failed attempt no longer goes unnoticed: the Access page shows a banner
  with the exact reason until a later attempt succeeds, and the same text comes back from
  `GET /api/tenant/loginlink/data` (field `apiRegisterError`) for a calling app to show too.

## Who connects it — platform vs. tenant control

For a tenant's own team/data sign-in (points #2/#3 above), a platform administrator first decides *who* is
in control: either the platform connects and manages it on the tenant's behalf, or the tenant is allowed to
connect and manage its own — see
[Admin console → Tenants → Connecting LoginLink per tenant](../admin-console/tenants.html#connecting-loginlink-per-tenant).
Whichever side is "in control" is the only side that can disconnect or reconfigure it; the other side sees
a read-only status.

## Known rough edge

"Check it still works" (on [Tenant console → Settings](../tenant-console/settings.html#loginlink-management-credential))
can report "No management credential is stored" even when a credential genuinely is stored, because the
check also requires a LoginLink **app** object to exist for the connection, and conflates these two
distinct missing-prerequisite states into one message. If you've definitely saved a credential and still
see this, the app side of the setup most likely still needs completing, not the credential itself.
