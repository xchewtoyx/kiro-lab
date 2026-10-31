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

In this lab, two sources could count as owner evidence:

1. **Declared:** the optional Service fields `owner` and `runbook`
   (a lab extension; see [`schema/README.md`](schema/README.md)). In
   each service file they sit right after `lifecycle`.
2. **Derived (from the base shape):** Service → `implemented_by` →
   Repository → `github:<org>/…` → `ownership/domains.yaml`. This gives
   a *domain*, not a team. There's no derived path for runbooks.

Which of these the report treats as owner evidence is an open decision in
`intent.md` (D10).

## Service fixtures

Each service exercises a case from `intent.md`. "Expected" applies the
D2 rule (present and non-empty means **recorded**) to the declared fields
only.

| Service | `owner` | `runbook` | Derived domain | Case exercised | Expected (declared) |
|---|---|---|---|---|---|
| `catalogue-api` | `team:catalogue` | local path | storefront | both recorded (ex. 1, 5a) | owner recorded, runbook recorded |
| `basket` | `team:checkout` | local path | storefront | both recorded, experimental | owner recorded, runbook recorded |
| `search-indexer` | `team:search` | URL | storefront | runbook as a URL | owner recorded, runbook recorded |
| `checkout` | `team:checkout` | *key absent* | storefront | runbook missing (ex. 3) | owner recorded, runbook missing |
| `payments-gateway` | *key absent* | local path | platform | owner missing but a domain is derivable (D10) | owner missing, runbook recorded |
| `feature-flags` | *key absent* | URL | **unmapped org** | owner missing, no derived domain either | owner missing, runbook recorded |
| `inventory-sync` | `""` | `"   "` | platform | empty string / whitespace only (D11) | owner missing, runbook missing *(provisional: D11)* |
| `image-resizer` | `[]` | `null` | storefront | empty list / null (D11) | owner missing, runbook missing *(provisional: D11)* |
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
