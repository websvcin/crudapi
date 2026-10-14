---
title: Row policies
parent: Features
nav_order: 2
---

# Row policies
{: .no_toc }

Limiting a role to *some rows* of a table, not just some tables — e.g. "Reader on `invoices`, but only
rows where `owner_id` is the signed-in caller."
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Created from a database's **Row policy** button on the [Databases](../tenant-console/databases.html) tab,
through a three-step builder.

## Step 1 — who, where, what

- **Which role** — the policy only affects keys/tokens using this *exact* role; every other role is
  unaffected. Pick from built-in or custom roles already defined on the
  [Roles](../tenant-console/roles.html) tab — a new role can't be created from inside this dialog.
- **Database** — fixed to whichever database's button you opened this from (so the table/column pickers
  can read its real schema). An **"Also apply this to every database in your workspace"** checkbox extends
  the same rule to every other database — but **only where a table and column of the exact same name
  exist**; it silently does nothing on a database that doesn't match, without any warning at save time.
- **Table** — read live from that database's real schema.
- **Action** — See rows / Add rows / Change rows / Delete rows. A role can be freely readable but still
  restricted on deletes (or vice versa) — each action needs its own policy if you want more than one
  constrained.

## Step 2 — which rows

Build one or more conditions (column + comparison + value), joined by **AND** (default) or **OR** — click
the small tag between two conditions to flip it. Conditions are read strictly in the order you built them.

{: .warning }
> **"Whoever is signed in right now" only means something for an app-login token (issuance-key JWT or
> LoginLink token).** An API key or Basic Auth login *is* the identity itself — there's no separate person
> behind it to match against, so this particular condition type has no effect for those credential types.
> An ordinary value condition (e.g. "status is Shipped") still applies to every credential type equally.

## Step 3 — review

A plain-language summary of exactly what the rule will do, with an optional "show the technical JSON"
toggle for the literal filter expression it compiles to, an optional note for your own team's reference,
and a required "I've read this and confirm it's correct" checkbox before saving.

## How it's enforced

The policy's condition is merged directly into the `filter` every matching request already builds — at the
exact same point [filtering](../api-reference/filtering-odata.html) is applied — so it can't be bypassed by
calling a different endpoint or omitting your own filter. On write actions (create/update/delete), a row
outside the policy's scope is refused with `403 row_policy_forbidden`, distinct from a plain `404`, so it's
clear the row exists but is off-limits rather than missing.

See [API reference → CRUD endpoints](../api-reference/crud-endpoints.html#errors) for exact error shapes,
and [Concepts → Authentication & credentials](../concepts/authentication-and-credentials.html#row-policies-restricting-which-rows-not-just-which-tables)
for how this fits into the broader permission model.
