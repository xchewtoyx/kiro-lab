# Intent: Service readiness report

Status: DRAFT. Decisions D1–D10 are made. D11 is open; it came from
investigating the synthetic catalogue in `catalog/`. Testable assumptions
(A1–A8) and technical unknown T4 remain open. This is intent only;
there is no design or implementation yet.

> Naming note: the name "service readiness report" was kept on purpose (D4).
> "Readiness" here means *readiness of the catalogue records to support
> preparing a change*. It does not mean the service is ready, healthy or
> approved to deploy. This definition lives only in this doc; the report
> output carries no disclaimer (D8). The name pulls against the no-claims
> constraint. The title is exempt from the no-readiness-wording rule (D9),
> and the risk is tracked as assumption A7.

## Problem

Before preparing a change, an engineer needs to know whether the affected
service has basic operational evidence on record: a named owner and a runbook.
Right now they have to look this up by hand, and a gap can stay hidden until
it matters, for example during an incident caused by the change, when nobody
knows who owns the service or how to roll it back.

## User

An engineer who is about to prepare a change to one or more services. They
are checking for gaps before they start. They aren't reviewing, approving or
operating the service.

## Desired outcome

The engineer runs a local report that **sweeps every service in the synthetic
catalogue** (decided: D1) and can see straight away **which services are
missing ownership or runbook evidence**, including the ones their change
touches. They can then fill those gaps or flag them before the change goes
ahead.

The report states facts about catalogue records only: a field is **recorded**
(present and non-empty), **missing** (absent or empty), or **unknown** (the
record can't be read). It never says anything about the service itself, and
it never says a recorded value is correct.

## Non-goals

- Does not say a service is healthy, available or performing well.
- Does not approve, block or gate any deployment or change, and does not
  imply that it does (no "ready", "pass", "approved" or "safe to deploy"
  wording, and no green-light summary).
- Does not check whether a recorded value is valid, well-formed or real.
  Placeholders such as `TBD` count as recorded (decided: D2). This includes
  not checking whether runbook links resolve, whether owners exist, or
  whether runbook content is correct or current.
- Does not infer an owner from anything else: not the implementing
  repository, its organization, the ownership domain, or related services.
  Only an `owner` value written on the service counts (decided: D10).
- Does not read live systems, production catalogues, monitoring or
  deployment tooling.
- Does not check any field other than owner and runbook: no on-call rota,
  escalation contact, SLOs, dependencies or similar (decided: D3). The set
  of fields is fixed, not configurable.
- Does not fix gaps or edit the catalogue.
- No scoring or ranking of services (a score reads as a health or readiness
  rating).

## Constraints

- **Local only:** runs on the engineer's machine against local files, with
  no network calls.
- **Synthetic data only:** no real service, team or person data.
- **No health or approval claims**, in wording, structure or summary
  (for example, no totals or proportions; see D7). The one exception
  is the title "service readiness report" (D9).
- An absent record or unreadable input must be reported as an *unknown*,
  never as "no gaps".
- **Status per field only:** because every service is listed (D6), a row
  must never carry an overall verdict (such as "OK", "pass", "ready" or a
  tick) that rolls owner and runbook into one judgement. Each field reports
  only its own state.

## Success examples

1. Service has a non-empty owner and runbook value. The report shows both
   as recorded and says nothing more, in particular not "ready", "valid"
   or "verified".
2. Service has no owner field. The report says: owner missing.
3. Service has a runbook field that is empty. The report says: runbook
   missing, not "recorded".
2a. Service has no `owner` field, but its repository's org maps to an
   ownership domain (for example, `payments-gateway`). The report still
   says: owner missing. The domain is not shown as an owner or as a
   fallback.
3a. Service has `owner: TBD`. The report shows owner as recorded. This is
   the accepted limit of D2, and the wording "recorded" must not suggest
   the value was checked.
4. A catalogue entry can't be read (for example, a malformed record). The
   sweep lists it as unknown, rather than skipping it or counting it as
   having no gaps.
5. Every service in the catalogue appears in the sweep output, including
   services with no gaps; none are silently dropped. An engineer can find
   the service their change touches and see its owner and runbook state.
5a. A service with both fields recorded shows "owner: recorded, runbook:
   recorded" and nothing that combines the two into an overall verdict.
6. The output has no summary, totals or proportions, only the per-service
   list (D7).
7. A reviewer reads any report output and can't find wording, other than
   the title "service readiness report" (exempt under D9), that implies
   health, readiness or approval.

## Decision owner

**The requester owns every decision in this doc** (decided: D5), including
the evidence rule (D2). No separate owner for catalogue data standards was
named. Splitting ownership with a platform or SRE lead was considered and
not chosen.

## Unresolved questions

### Decisions made

- **D1. Scope:** full catalogue sweep. A single-service lookup was
  considered and not chosen.
- **D2. Evidence bar:** present and non-empty. "Well-formed" and "resolves"
  were considered and not chosen. Accepted consequence: placeholders count
  as recorded.
- **D3. Evidence types:** owner and runbook only. Adding on-call or
  escalation contact, and making the fields configurable, were considered
  and not chosen.
- **D4. Name:** "service readiness report." "Service evidence gap report"
  and "catalogue ownership and runbook record check" were considered and not
  chosen. Accepted risk: the name invites readers to take clean rows as
  "ready" (see A7).
- **D5. Decision owner:** the requester, for everything including the
  evidence rule. A split with a platform or SRE lead was considered and not
  chosen.
- **D6. Output shape:** every service listed with its status, clean or not.
  "Gaps only" and "gaps first, then clean" were considered and not chosen.
  Accepted risk: clean rows may read as a "ready" list (see the constraint
  on per-field status).
- **D7. No aggregate summary;** the output is the per-service list only.
  "Gap counts only" and "recorded and missing counts" were considered and not
  chosen. Accepted consequence: there is no "checked N services" line, so
  the output itself doesn't show that nothing was dropped (see A8).
- **D10. Owner must be explicit:** only a declared `owner` value on the
  service counts as owner evidence. Ownership derived through
  `implemented_by` → Repository → org → `ownership/domains.yaml` does not
  count, even as a fallback. "Derived only" and "either" were considered
  and not chosen. Accepted consequence: `payments-gateway` reports owner
  missing although a domain is derivable. A domain identifies an area,
  not an accountable team.
- **D8. No "what this is not" statement in the output.** A required
  disclaimer line, and a disclaimer only on all-clean reports, were
  considered and not chosen. The per-field wording is the only safeguard
  inside the output itself. Accepted consequence: the name (D4), every row
  listed (D6) and no disclaimer (D8) all lean the same way, so A7 now
  carries the no-readiness-claim intent on its own. Testing A7 early is
  recommended.
- **D9. Name vs the no-claims rule:** the title is exempt; all other output
  wording is held to the rule. "Literal report heading" and "relax the rule
  to explicit claims only" were considered and not chosen. Accepted
  consequence: the most prominent word in the output is the one the rule
  otherwise forbids.

### Human decisions still open

This came out of investigating the catalogue shape (T1).

- **D11. Exactly what counts as empty.** D2 says "non-empty", but the data
  contains cases where that's a judgement call: `""`, whitespace-only
  `"   "`, `null` and `[]`. Do all four count as missing? The catalogue
  README treats them as missing *provisionally*.

### Testable assumptions (can be checked with users or data)

- **A1.** Engineers check for these gaps *before* preparing a change, rather
  than during review or after an incident.
- **A2.** Missing owner and runbook are the gaps that matter most before a
  change.
- **A3.** The synthetic catalogue has the same shape as the real one it
  stands in for, so lessons carry over. *Partly disproved:* the kinds, IDs
  and relationships match, but `owner` and `runbook` are lab additions
  the real shape doesn't have.
- **A4.** Engineers will read "missing" as a prompt to act, not as a block
  or an approval.
- **A5.** An engineer preparing a change can quickly find the services they
  care about in a full sweep. (Catalogue maintainers are a likely second
  audience; check whether the report is really for them.)
- **A6.** Placeholder values are rare enough in the catalogue that "present
  and non-empty" still gives a useful signal. *Can't be tested on the
  synthetic catalogue,* because its placeholder rate was chosen by hand.
  It needs real data.
- **A7.** Engineers reading a report titled "service readiness report" won't
  take a row with both fields recorded as meaning the service is ready or
  approved. Test this by showing sample output to a few engineers and asking
  what a clean row means.
- **A8.** With no summary line (D7), engineers will trust that the list is
  the complete catalogue. Check this by comparing output against the
  synthetic catalogue in testing, and by asking engineers whether they
  missed a count.

### Technical unknowns (need investigation once inputs exist)

- **T1. Resolved.** A synthetic catalogue now lives in
  [`catalog/`](catalog/README.md). It's one YAML file per entity, with
  `{kind}:{slug}` IDs and 13 service fixtures (12 parseable, 1
  deliberately malformed). Findings from the base shape: there was **no
  runbook concept anywhere** and **no per-service owner** (see D10).
  Both fields were added as optional Service-only lab fields; per D10,
  the declared `owner` field is the only owner evidence.
- **T2. Answered by the data, now a decision (D11).** The fixtures contain
  an absent key, `""`, whitespace only, `null` and `[]`.
- **T3. Answered by the data.** `owner` may be a string or a list
  (`recommendations` has two teams). An empty list is the only
  list-specific edge case, and it's folded into D11.
- **T4. New:** the base catalogue requires every file to be valid, and the
  source repo's validator rejects the whole catalogue if any file is bad.
  The lab deliberately includes a malformed file (`ledger-export`) to
  exercise "unknown" (success example 4). How the report coexists with a
  strict validator is for design, not intent.
