---
title: Engine roadmap
parent: Admin console
nav_order: 6
---

# Engine roadmap
{: .no_toc }

`/admin/engineroadmap` — curate which database engines show your team as "🔜 Planned," ahead of a real
SqlConnector release.
{: .fs-6 .fw-300 }

---

## Active engines

A read-only banner listing every engine currently **active** — meaning it's actually registered by your
installed SqlConnector package (`AddSqlConnector()` in your host project). Today that's MySQL only. This
list can't be edited here: to add a real active engine, you upgrade the underlying NuGet package, not this
page.

## Planned engines (curated)

A simple table you maintain by hand, purely for internal communication — "here's what's coming" — with no
functional effect on the product:

| Field | Meaning |
|---|---|
| Engine key | Unique identifier (e.g. `oracle`) — can't be changed after creation. |
| Display name | What your team sees (e.g. "Oracle"). |
| Package hint | A free-text note, e.g. "SqlConnector v2.0." |
| Sort order | Controls display order in the list. |
| Description | Optional free text. |
| Visible | Checked = shown as "🔜 Planned"; unchecked lets you stage an entry before announcing it. |

Each entry can be edited or deleted. This is purely informational bookkeeping — it never adds real engine
support; see [Concepts → Architecture](../concepts/architecture.html#multi-engine-by-design-mysql-today) for
how real engine support actually gets added.
