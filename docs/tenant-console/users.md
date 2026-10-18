---
title: Users
parent: Tenant console
nav_order: 4
---

# Users
{: .no_toc }

The people and services that can connect to your app through this workspace's LoginLink or issuance-key
flows.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

{: .note }
> This is about **your app's own end-users** — the people a LoginLink token or an issuance-key JWT is
> issued for — not your team's console sign-in (that's [Access](access.html)), and not credentials your
> servers use directly (that's [Keys](keys.html)).

## Only people you add can connect

A toggle deciding how new users are admitted:

- **Off** (default) — anyone who successfully signs in through your configured flow is added
  automatically, the first time they're seen.
- **On** — only users you've explicitly added with **+ Add user** may connect; anyone else is refused.

## Activity by person

A usage strip above the user list (24h / 7 days / 30 days) summarizing request volume per person, so you
can spot a sudden spike or a user who's gone quiet without digging into [Usage](usage.html) directly.

## The users table

Search by name, email, or external id; filter by **All** / **Active** / **Disabled**. Each row: the
person, their request count, how many requests were turned away (denied), when they were last seen, their
status, and row actions (disable/enable, remove).

**Who a row shows as the person's name:** their display name if one's been set (via **Edit**), else their
email if LoginLink has sent one on their token, else their raw subject id as a last resort — a LoginLink
sign-in's email is picked up automatically the first time this workspace sees a request from them after it's
known, with no action needed here.

**+ Add user** registers someone ahead of their first sign-in — required if "Only people you add can
connect" is on; optional otherwise, useful for pre-provisioning.
