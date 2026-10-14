---
title: Filtering (OData)
parent: API reference
nav_order: 4
---

# Filtering (OData)
{: .no_toc }

The `filter` query parameter every list endpoint accepts, and the one rule that trips people up.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The one rule: every column is qualified

```
?filter=widgets.status eq 'active'
```

**Always `table.column`, never `column` alone.** This is deliberate, not an oversight: `filter` is also
where a [row policy](../features/row-policies.html) gets merged in server-side, and `join` can bring
columns from more than one table into scope at once — qualifying every column keeps "which table does
`status` belong to" unambiguous in every case, not just the simple ones. An unqualified column name is
refused with a `400` rather than guessed at.

```bash
# Right
?filter=widgets.status eq 'active'

# Refused
?filter=status eq 'active'
```

## Operators

| Operator | Meaning | Example |
|---|---|---|
| `eq` | equals | `widgets.status eq 'active'` |
| `ne` | not equals | `widgets.status ne 'archived'` |
| `gt` / `ge` | greater than / or equal | `widgets.price gt 100` |
| `lt` / `le` | less than / or equal | `widgets.price le 50` |
| `contains` | substring match | `contains(widgets.name,'bolt')` |
| `startswith` | prefix match | `startswith(widgets.sku,'A-')` |

Combine conditions with `and` / `or`, and group with parentheses:

```
?filter=(widgets.status eq 'active' or widgets.status eq 'pending') and widgets.price gt 10
```

String values are single-quoted; numbers and booleans are bare.

## Combining with other query parameters

`filter` composes with `join`, `display`, `limit`, `orderBy`, and `groupBy` in the same request — see
[CRUD endpoints](crud-endpoints.html#list-rows--get-crudtable) for the full parameter table. When `join`
brings in a related table, that table's columns become filterable the same qualified way
(`customers.region eq 'APAC'`), since the join target is just another `table.column` the filter can now see.

## Row policies ride along in the same filter

If the caller's role carries a row policy, its condition is combined with whatever `filter` you send — you
never need to (and can't) manually add your own role's restriction; it's already there before your request
reaches the database. See [Features → Row policies](../features/row-policies.html) for how that
restriction is actually defined and enforced.

## Where else filtering shows up

- **Bulk → filter source** ([Bulk endpoints](bulk-endpoints.html#source-2--copy-rows-from-another-table-with-a-filter))
  uses the exact same `filter` syntax to select which rows of a *source* table get copied.
- **A built-in or custom role's row policy** ([Features → Row policies](../features/row-policies.html)) is
  authored using this same condition language.

## Common mistake

```
400  server_error  — "status" is not a recognized column
```

This is almost always a missing `table.` prefix. Re-check that every column reference in your `filter`
string — including inside `contains(...)`/`startswith(...)` — is qualified.
