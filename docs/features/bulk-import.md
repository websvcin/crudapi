---
title: Bulk import
parent: Features
nav_order: 1
---

# Bulk import
{: .no_toc }

Loading many rows at once, from a CSV file, a filtered copy of another table, or an external API.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

{: .note }
> Bulk import is a **data-API capability**, not a dedicated screen in the admin or tenant console. You call
> it directly (see [API reference → Bulk endpoints](../api-reference/bulk-endpoints.html)), or it's
> surfaced as buttons in whatever frontend you build against the
> [generated dashboard config](generate-config-dashboards.html) — the config's `bulkActions` feature wires
> each table to whichever of the three sources below you asked [Generate Config](generate-config-dashboards.html)
> to include for it.

## Three sources

| Source | Good for |
|---|---|
| **CSV file upload** | A one-off import from a spreadsheet export — includes a downloadable template with the right headers, foreign-key and enum hints baked in. |
| **Filtered copy from another table** | Cloning/archiving rows already in your database into a different table, optionally stamping fixed override values on every copied row. |
| **External API** | Pulling from a third-party API and mapping its fields onto your table's columns. |

Full endpoint shapes, request/response examples, and execution-mode behavior (realtime vs. background,
based on row count): [API reference → Bulk endpoints](../api-reference/bulk-endpoints.html).

## Realtime vs. background

A small import runs synchronously and you get the final result immediately; a large one is queued and you
get a job id back instead. The threshold is configurable platform-wide (`bulk.realtime_row_threshold`, see
[Admin console → Settings → Host Settings](../admin-console/settings.html#host-settings)), or you can force
either mode explicitly per request.

## Row policies apply during import too

If the caller's role carries a [row policy](row-policies.html), rows that would violate it are rejected
rather than written — for the filter and API-source sources, this is checked before anything is inserted; a
batch where every row gets rejected comes back as a downloadable CSV of the rejected rows and why, instead
of a misleadingly empty success response.

## Partial failures

A batch with some bad rows still inserts everything that's valid — the response (or, for a realtime run
with failures, a downloadable CSV) tells you exactly which rows failed and why, so you can fix and re-run
just those instead of starting over.
