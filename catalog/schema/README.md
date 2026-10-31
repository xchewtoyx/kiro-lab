# Catalog schema v1 (synthetic lab)

`entity.schema.json` defines the entity shape. `relationship-types.yaml`
defines the allowed relationship pairs. Every entity file and
`relationship-types.yaml` carries `schema_version: 1`.

## Stable entity IDs

IDs are `{kind}:{slug}`, unique across the whole catalog, and stable once
published.

| Kind | Prefix | Directory |
|---|---|---|
| Service | `service:` | `entities/services/` |
| System | `system:` | `entities/systems/` |
| Capability | `capability:` | `entities/capabilities/` |
| Resource | `resource:` | `entities/resources/` |
| Repository | `repository:` | `entities/repositories/` |
| Project | `project:` | `entities/projects/` |
| Objective | `objective:` | `entities/objectives/` |

* The prefix matches `kind` (lowercased).
* A slug is lowercase, starts and ends with `[a-z0-9]`, and may contain
  `.`, `_`, `/` or `-` in between. Repository slugs are `owner/repo`.
* Reference kinds store a stable pointer in `external_ref`:
  Repository → `github:owner/repo`, Project → `linear:project:<uuid>`,
  Objective → `linear:initiative:<uuid>`. All UUIDs here are made up.

## Lab extension: `owner` and `runbook`

The base shape has no per-service owner or runbook. Ownership is derived
through `implemented_by` → Repository → org → `ownership/domains.yaml`,
and runbooks don't exist at all. This lab adds two **optional,
Service-only** fields:

| Field | Type | Meaning |
|---|---|---|
| `owner` | string, list of strings, or null | Declared owning team(s) |
| `runbook` | string or null | Path or URL of a runbook |

Under the v1 policy this is a compatible change, because the fields are
optional. The schema deliberately does **not** reject empty values (`""`,
whitespace, `[]`, `null`): the fixtures use them to give the readiness
report gaps to find.
