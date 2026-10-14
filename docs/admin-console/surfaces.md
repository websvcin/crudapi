---
title: Who can do what
parent: Admin console
nav_order: 11
---

# Who can do what
{: .no_toc }

`/admin/surfaces` — a live capability matrix comparing the platform admin console against each tenant's own
console.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The two surfaces

| | Platform admin | Tenant |
|---|---|---|
| Address | `/admin` | `/{slug}/login` |
| Who | Whoever runs this CrudAdmin instance; signs in with email + password (or LoginLink, see [Admin users](admins.html)). | A customer's own administrators; sign in with a console key, data-access key, or LoginLink. |
| Reach | Every tenant, plus everything about running the platform itself. | Only its own tenant — can never see another's. |

The top of the page shows live counts for each surface: how many capabilities it has *today*, and how many
are *planned* but not yet built.

## The capability table

Grouped by functional area (Tenants, Databases, Access, Keys, Audit, etc.), each row is one capability with
two columns — **Platform admin** and **Tenant** — each marked:

| Mark | Meaning |
|---|---|
| ✓ Today | Already live; links to the actual page(s) or tenant tab where it lives. |
| ◔ Planned | Not built yet, with a short note on the plan where available. |
| — | Not applicable to that surface at all. |

This is the fastest way to answer "can a tenant already do X themselves, or does that still require a
platform admin?" without digging through every individual screen.

## Where today's pages end up

A historical note some admin pages still carry: this console originally combined platform-level and
single-tenant concerns on the same set of pages, with a stated plan to separate them once a proper tenant
console existed. That tenant console [now exists](../tenant-console/) — the capability table above reflects
the current, already-separated state; this section is kept as a record of which legacy page absorbed which
responsibility, useful mainly if you're troubleshooting a page that still mentions the older combined
layout.
