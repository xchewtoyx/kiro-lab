# Synthetic lab catalog

**Everything here is made up.** It's a fictional online shop. The
organizations (`example-*`), teams, services, URLs (`*.example.com`) and
UUIDs are invented for a public lab repo and refer to no real system,
team or person.

The *shape* (entity kinds, `{kind}:{slug}` IDs, relationship types,
ownership domains) is modelled on a private internal catalog skeleton. No
content from that catalog has been copied.

This is the input data for the service readiness report described in
[`../intent.md`](../intent.md).

## Layout

```text
catalog/
  schema/
    README.md                 # ID rules and the owner/runbook lab extension
    entity.schema.json        # v1 entity shape
    relationship-types.yaml   # allowed source/target pairs
  ownership/
    domains.yaml              # repository org -> ownership domain
  entities/
    services/  systems/  capabilities/  resources/
    repositories/  projects/  objectives/
```

There is one entity per YAML file, each directly under its kind's directory.

## Where ownership and runbook evidence lives

**Owner must be explicit** (`intent.md` D10). Only the optional Service
fields `owner` and `runbook` count as evidence. They're a lab extension
(see [`schema/README.md`](schema/README.md)) and sit right after
`lifecycle` in each service file.

The base shape also lets you *derive* an ownership domain: Service →
`implemented_by` → Repository → `github:<org>/…` →
`ownership/domains.yaml`. That path is kept so the catalogue shape is
faithful, but it is **not owner evidence**. A domain names an area, not an
accountable team. The "Derived domain" column below shows where the two
would disagree.

## Service fixtures

Each service exercises a case from `intent.md`. "Expected" applies the
D2 rule (present and non-empty means **recorded**) to the declared fields
only (D10). Absent, `""`, whitespace only, `null` and `[]` all count as
**missing** (D11).

| Service | `owner` | `runbook` | Derived domain | Case exercised | Expected |
|---|---|---|---|---|---|
| `catalogue-api` | `team:catalogue` | local path | storefront | both recorded (ex. 1, 5a) | owner recorded, runbook recorded |
| `basket` | `team:checkout` | local path | storefront | both recorded, experimental | owner recorded, runbook recorded |
| `search-indexer` | `team:search` | URL | storefront | runbook as a URL | owner recorded, runbook recorded |
| `checkout` | `team:checkout` | *key absent* | storefront | runbook missing (ex. 3) | owner recorded, runbook missing |
| `payments-gateway` | *key absent* | local path | platform | owner missing even though a domain is derivable (D10: not evidence) | owner missing, runbook recorded |
| `feature-flags` | *key absent* | URL | **unmapped org** | owner missing, no derived domain either | owner missing, runbook recorded |
| `inventory-sync` | `""` | `"   "` | platform | empty string / whitespace only (D11) | owner missing, runbook missing |
| `image-resizer` | `[]` | `null` | storefront | empty list / null (D11) | owner missing, runbook missing |
| `notifications` | `TBD` | `TODO` | platform | placeholders count as recorded (ex. 3a) | owner recorded, runbook recorded |
| `recommendations` | two teams (list) | local path | labs | more than one owner (T3) | owner recorded, runbook recorded |
| `email-relay` | `team:messaging` | `runbooks/does-not-exist.md` | platform | value not checked (D2) | owner recorded, runbook recorded |
| `session-store` | *key absent* | *key absent* | **no repository** | deprecated, nothing recorded | owner missing, runbook missing |
| `ledger-export` | — | — | — | **file is not valid YAML** (ex. 4) | unknown |

Notes:

* `ledger-export.yaml` is broken on purpose: it has an unclosed quote and
  bad indentation. It's the only file that fails to parse; every other
  entity is schema-valid and only references entities that exist.
* None of the local runbook paths point to real files. Under D2 that
  doesn't matter: only presence is checked.
* The project `project:checkout-retry` `changes` `checkout` and
  `payments-gateway`. It's a ready-made "change I'm about to prepare"
  scenario: both services have a gap.
