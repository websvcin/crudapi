---
title: Help & guides
parent: Admin console
nav_order: 14
---

# Help & guides
{: .no_toc }

`/admin/help` — a built-in, in-product guide to every admin console screen, access, settings, and
troubleshooting. Worth knowing this exists before reaching for this public documentation site.
{: .fs-6 .fw-300 }

---

This screen is a self-contained reference living inside the product itself, covering: a tour of every
admin screen, what a tenant sees on their side, how access/roles/tiers work, what each error means,
copyable example requests, passwords and sign-in, the audit log, where each integration setting's value
comes from, when a security setting takes effect, database engines, where data lives and how to back it
up, console usage tips, and a glossary. **Open every page guide** expands every collapsed section at once.

## The first hour, in order

The guide's own recommended sequence for a brand-new install:

1. **Finish first-time setup** — see [Deployment → Setup wizard](../deployment/setup-wizard.html).
2. **Download the recovery kit** from [Storage & data](storage-databases.html) — without it, losing the
   data file also loses the signing keys, and every token stops working.
3. **Fill in Host Settings** if you use GitHub/AI (for Generate Config) or LoginLink (for tenant sign-in) —
   see [Settings → Host Settings](settings.html#host-settings).
4. **Check Security Settings**, especially password rules — see [Settings → Security Settings](settings.html#security-settings).
   The [Dashboard](dashboard.html)'s "Needs attention" panel surfaces real gaps automatically.
5. **Create a tenant** on [Tenants](tenants.html), register or create its database, and give it a key.
6. **Add the rest of your platform team** on [Admin users](admins.html).

This public site covers the same ground in more depth and with full written detail per screen — use
whichever is more convenient in the moment; the in-product guide is handy for a quick reminder while
you're already looking at the screen in question.
