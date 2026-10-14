---
title: Permissions model
parent: API reference
nav_order: 5
---

# Permissions model
{: .no_toc }

How `{resource}.{action}` permissions actually work, for every data-API endpoint.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The shape of a permission

Every data-API permission is a string: `{resource}.{action}`. `{resource}` is either a real table name or
the special pseudo-resource `database` (for schema-level actions); `{action}` is one of `read`, `create`,
`update`, `delete` (for tables) or `read`, `config`, `bootstrap`, `script` (for `database`). `*` is a
wildcard for "every table" — `*.read` means read access to every table in the database(s) the credential
reaches.

A role is just a set of these strings. The five built-in roles are fixed combinations; a tenant can also
define its own custom roles (if its platform administrator has allowed it) from the same vocabulary, minus
`database.config`/`database.bootstrap`/`database.script` and the `*` wildcard, which stay reserved for the
built-in roles — see [Tenant console → Roles](../tenant-console/roles.html).

## Which route needs which permission

| Route | Permission needed |
|---|---|
| `GET crud/{table}` , `GET crud/{table}/{col}/{val}` | `{table}.read` |
| `POST crud/{table}` | `{table}.create` |
| `PUT crud/{table}/{col}/{val}` | `{table}.update` |
| `DELETE crud/{table}/{col}/{val}` | `{table}.delete` |
| `GET bulk/template/{table}/csv` | `{table}.read` |
| `POST bulk/upload/{table}` | `{table}.create` |
| `POST bulk/filter/{table}` | `{table}.create` (plus `{sourceTable}.read` to read the source) |
| `POST bulk/api-source/{table}` | `{table}.create` |
| `GET database/info` , `tables` , `relationships` | `database.read` |
| `GET database/generate-config` | `database.config` |
| `POST database/bootstrap-db/{database}` | `database.bootstrap` |
| `POST database/script` | `database.script` |

## How a request is actually checked

1. The credential resolves to a tenant, a set of reachable databases, and a role (see
   [Concepts → Authentication & credentials](../concepts/authentication-and-credentials.html)).
2. The route maps to a `{resource}.{action}` pair, from the table above.
3. The role's permission set is checked for that exact pair, or the matching `*.{action}` wildcard.
4. If it's missing, the response is `403 forbidden_action` — naming the action, not a generic
   "access denied" — and the attempt is written to the audit log either way.
5. If it's present and the action is a read/write on a table, any row policy attached to the role is also
   enforced at this point (see [Features → Row policies](../features/row-policies.html)) — holding the
   table-level permission is necessary but not sufficient if a row policy narrows which rows you may
   actually reach.

## Built-in roles, as permission sets

| Role | Effective permissions |
|---|---|
| Reader | `*.read` |
| Writer | `*.read`, `*.create`, `*.update` |
| Editor | `*.read`, `*.create`, `*.update`, `*.delete` |
| Database admin | Editor's set, plus `database.read`, `database.config`, `database.script` |
| Owner | Everything, including `database.bootstrap` |

Database admin and Owner additionally require platform-level approval before a tenant may hand either out —
see [Admin console → Access model](../admin-console/access-model.html).

## Designing a custom role

A custom role picks any subset of `{table}.{action}` permissions, scoped to specific tables rather than the
`*` wildcard if you want finer control (e.g. `orders.read` + `orders.update` and nothing else, for a key
that should only ever touch one table). See [Tenant console → Roles](../tenant-console/roles.html) for the
builder UI and exactly which actions are off-limits to a custom role.
