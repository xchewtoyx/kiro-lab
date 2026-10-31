# Requirements: Service readiness report

Status: DRAFT for review. This is phase 1 of a requirements-first spec.
Design and tasks come only after these requirements are approved.

Source of intent: [`intent.md`](../../../intent.md). Decision IDs (D1–D11)
and success examples (ex. N) refer to that document. Input data:
[`catalog/`](../../../catalog/README.md).

## Introduction

The service readiness report is a local, read-only report over the
synthetic catalogue. For every service it states whether the catalogue
records an **owner** and a **runbook**. Its job is to show an engineer
preparing a change where that evidence is missing. It makes no claim about
a service's health, and none about whether a change or deployment is
approved.

## Glossary

- **Catalogue:** the directory tree described in `catalog/README.md`.
- **Service file:** any file directly under `catalog/entities/services/`
  with a `.yaml` extension.
- **Owner field / runbook field:** the optional top-level `owner` and
  `runbook` keys of a service entity.
- **Recorded:** the field is present and its value is non-empty.
- **Missing:** the key is absent, or its value is `""`, whitespace only,
  `null` or `[]` (D11).
- **Unknown:** the report could not read the service file, so it can't
  say whether the field is recorded or missing.
- **Field state:** exactly one of recorded, missing or unknown.

## Requirements

### Requirement 1: Sweep every service

**User story:** As an engineer preparing a change, I want every service in
the catalogue listed, so that I can find the services my change touches
and trust that none were left out.

Traces to: D1, D6, ex. 5.

#### Acceptance criteria

1. WHEN the engineer runs the report against a catalogue, THE report SHALL
   produce exactly one entry for each service file.
2. THE report SHALL include services whose owner and runbook are both
   recorded. It SHALL NOT list only services with gaps.
3. THE report SHALL identify each entry by the service's `id` when it can
   be read, and otherwise by the service file's path.
4. WHEN the report runs twice against an unchanged catalogue, THE report
   SHALL produce identical output.
5. THE report SHALL NOT produce entries for files outside the services
   directory. (How a non-Service entity inside that directory is handled
   is open question Q1.)

### Requirement 2: Owner state

**User story:** As an engineer preparing a change, I want to see whether
each service declares an owner, so that I know who to involve before I
change it.

Traces to: D2, D10, D11, ex. 1, 2, 2a, 3a.

#### Acceptance criteria

1. WHEN the owner field is a string that has at least one non-whitespace
   character, THE report SHALL show the owner state as recorded.
2. WHEN the owner field is a list with at least one entry, THE report
   SHALL show the owner state as recorded. *(See open question Q2.)*
3. WHEN the owner key is absent, or its value is `""`, whitespace only,
   `null` or `[]`, THE report SHALL show the owner state as missing.
4. THE report SHALL treat placeholder values such as `TBD` as recorded.
   It SHALL NOT judge whether a recorded value is real or valid.
5. THE report SHALL decide the owner state from the service's own `owner`
   field only. It SHALL NOT use `implemented_by`, the repository, its
   organization, `ownership/domains.yaml` or related services to supply
   or infer an owner, even as a fallback.

### Requirement 3: Runbook state

**User story:** As an engineer preparing a change, I want to see whether
each service declares a runbook, so that I know whether there's guidance
for operating it if my change goes wrong.

Traces to: D2, D11, ex. 1, 3.

#### Acceptance criteria

1. WHEN the runbook field is a string that has at least one
   non-whitespace character, THE report SHALL show the runbook state as
   recorded.
2. WHEN the runbook key is absent, or its value is `""`, whitespace only
   or `null`, THE report SHALL show the runbook state as missing.
3. THE report SHALL NOT check whether a recorded runbook path exists or
   whether a recorded URL resolves.

### Requirement 4: Unreadable service files

**User story:** As an engineer preparing a change, I want unreadable
records shown as unknown, so that a broken record is never mistaken for
one with no gaps.

Traces to: constraints ("unknown, never no gaps"), ex. 4, T4.

#### Acceptance criteria

1. IF a service file cannot be parsed, THEN THE report SHALL list it with
   owner state unknown and runbook state unknown, identified by its file
   path.
2. IF one or more service files cannot be parsed, THEN THE report SHALL
   still report every other service file. One bad file SHALL NOT stop the
   sweep.
3. IF the catalogue's services directory is absent or cannot be read,
   THEN THE report SHALL fail with an error stating this. It SHALL NOT
   produce an empty report.
4. *(Open question Q1: service files that parse but break the schema.)*

### Requirement 5: Wording and shape of the output

**User story:** As an engineer preparing a change, I want the output to
state record facts only, so that nobody can read it as a health check or
an approval.

Traces to: D6, D7, D8, D9, non-goals, ex. 1, 5a, 6, 7.

#### Acceptance criteria

1. THE report SHALL show each service's owner state and runbook state
   separately, using only the words recorded, missing and unknown.
2. THE report SHALL NOT show an overall verdict for a service (for
   example "OK", "pass", "ready" or a tick) that combines the two field
   states.
3. THE report SHALL NOT include a summary, totals, counts, proportions,
   scores or rankings.
4. Apart from its title, "Service readiness report", THE report SHALL NOT
   contain wording that states or implies that a service is healthy,
   ready, safe or approved (D9).
5. THE report SHALL NOT be required to include a disclaimer (D8).
6. WHEN a field is recorded, THE report SHALL NOT describe the value as
   valid, verified or checked.

### Requirement 6: Local, read-only, fixed scope

**User story:** As an engineer, I want the report to run locally with no
side effects, so that I can run it at any time without risk.

Traces to: D3, constraints, non-goals.

#### Acceptance criteria

1. THE report SHALL read only local files under the catalogue directory
   it is given.
2. THE report SHALL NOT make any network call.
3. THE report SHALL NOT create, modify or delete any file in the
   catalogue.
4. THE report SHALL evaluate only the owner and runbook fields. The set of
   fields SHALL NOT be configurable.

## Acceptance against the synthetic catalogue

Run against `catalog/` as committed, the report must give these field
states. This is the expected-result table from `catalog/README.md`,
restated as requirements.

| Service entry | Owner | Runbook | Criteria exercised |
|---|---|---|---|
| `service:catalogue-api` | recorded | recorded | 2.1, 3.1, 5.1, 5.2 |
| `service:basket` | recorded | recorded | 2.1, 3.1 |
| `service:search-indexer` | recorded | recorded | 3.1 (URL) |
| `service:checkout` | recorded | missing | 3.2 (absent key) |
| `service:payments-gateway` | missing | recorded | 2.3, 2.5 (domain derivable, not used) |
| `service:feature-flags` | missing | recorded | 2.3, 2.5 |
| `service:inventory-sync` | missing | missing | 2.3 (`""`), 3.2 (whitespace) |
| `service:image-resizer` | missing | missing | 2.3 (`[]`), 3.2 (`null`) |
| `service:notifications` | recorded | recorded | 2.4 (`TBD`), 3.1 (`TODO`) |
| `service:recommendations` | recorded | recorded | 2.2 (two owners) |
| `service:email-relay` | recorded | recorded | 3.3 (path doesn't exist) |
| `service:session-store` | missing | missing | 2.3, 3.2 (deprecated, no repo) |
| `catalog/entities/services/ledger-export.yaml` | unknown | unknown | 4.1, 4.2, 1.3 |

That's 13 entries in total (1.1). No entry may carry an overall verdict
(5.2), and the output has no counts (5.3).

## Open questions (to settle before design)

- **Q1. Parseable but invalid service files.** A file that is valid YAML
  but breaks the schema: for example `owner: 42`, `runbook: [a, b]`, a
  missing `id`, or `kind: System` in the services directory. Is that
  service shown as unknown, or are its fields judged as written? No
  fixture covers this yet.
- **Q2. Lists of blank entries.** Is `owner: ["", "  "]` recorded (the
  list has entries) or missing (no entry names anyone)? D11 covers `[]`
  but not this case.
- **Q3. Output form.** Terminal text, a Markdown file, or something
  machine-readable? This affects how the output travels, for example
  pasted into a change ticket, which is why D8 (no disclaimer) mattered.
  Only the requirement (who reads it and where) is in scope here; the
  format details are for design.
