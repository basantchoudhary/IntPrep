# Ops Gate — Implementation Design

How the [170-item acceptance checklist](ops-acceptance-checklist.md) becomes a running gate,
given that evidence is not standardised, most items cannot be automated on day one, and the
PoC has no mandate.

This document decides four things: **the ladder** (how an unautomatable item still counts),
**the contracts** (input, process, output), **where AI sits**, and **the PoC build order**.

---

## 1 · Decisions taken

Recorded as decisions rather than options, so the reasoning survives the next argument about them.

| # | Decision | Rationale | Reversible? |
|---|---|---|---|
| D1 | **Every item enters at Level 0 and climbs independently.** No item is dropped for being hard to verify. | Otherwise the bar silently becomes "what our tooling can see". | No — this is the spine |
| D2 | **Normalise what the adapter emits, never the input.** Rules are written against a fixed field vocabulary per domain. | One rule covers Azure DevOps, GitHub Actions and Jenkins. Skipping this gives 170 bespoke rules and collapse within a year. | No |
| D3 | **Three fixed tier profiles.** No per-project profiles; project-specific relief comes only through waivers, which are visible and expire. | A negotiable bar is not a bar. Three profiles means three conversations, ever. | Yes |
| D4 | **Item text is global; severity is per profile.** `OWN-04` is Blocker at tier 1, Major at tier 2, absent at tier 3. | Fixes the 89-blocker problem without touching the checklist. An overlay is just a map of `id → severity`. | Yes |
| D5 | **The PoC wires three evidence platforms**: repo scan, CI results, cloud resource + tag state. | These need only a repository and one cloud account — plausibly obtainable without a programme. | Yes |
| D6 | **Ask for attachment, not veto.** The evidence bundle is attached to the acceptance record; it does not block the pipeline. | Veto is political and gets negotiated. Attachment is administrative and does nearly the same work. | Yes |
| D7 | **`declared` is a distinct status from `pass`.** Level 0 satisfies an item without claiming it was verified. | The bundle must never imply a machine checked something a human asserted. | No |

**Assumptions still to confirm** — D5 assumes an Azure-centric estate with Databricks and Azure
Monitor. If GSK's actual stack differs, adapter sizing changes but nothing else in this design does.

---

## 2 · The verification ladder

The core mechanism. `level` is a property of each check, and the engine's behaviour switches on it.

| Level | Name | What the project provides | What the gate stores | Status it can produce |
|---|---|---|---|---|
| **0** | Declared | An assertion, with a name and a date | The assertion itself | `declared` · `missing` |
| **1** | Evidenced | A pointer to an artefact | The artefact, hashed into the bundle | `evidenced` · `missing` · `error` |
| **2** | Verified | Nothing new — the adapter fetches it | Observed values + rule verdict | `pass` · `fail` · `error` |
| **3** | Continuous | Nothing new | Time series after handover | `pass` · `fail` · `error` |

Three properties follow, and they are the whole reason for the design:

- **The gate ships complete on day one.** All 170 items are live at Level 0. That version already
  beats a document review, because every item carries a name, a date, and a waiver register with no
  blanks.
- **Promotion never rewrites a rule.** Moving `RES-06` from Level 0 to Level 2 adds an adapter and a
  rule. No other check changes, no consumer changes.
- **The maturity number is real.** Share of applicable items at Level 2+ starts low and rises. A
  number that starts at 15% and climbs is a better story than one claiming 80% on day one.

### Level 0 is not a checkbox — the knob that stops it becoming one

The obvious objection is that Level 0 is self-certification with extra steps. Two things prevent that:

1. **`declared` is not `pass`.** It appears differently in the report and the bundle, and it never
   counts toward the maturity number.
2. **A profile can require a minimum level per severity.** Tier 1 sets
   `verdict.requires_level: {blocker: 2}` — a blocker that is only *declared* does not pass at tier 1.
   Tier 3 leaves it at 0. This is where the tiering earns its keep.

---

## 3 · Input contract

Four files. Each has exactly one job, and only the first is a genuinely new artefact.

### 3.1 `manifest.yaml` — pointers, not evidence

The project declares **where** things live, not what they contain. Keep it small: if it reads as a
form, it never gets filled in. Adapters probe conventional locations first, so this is mostly
overrides.

```yaml
project: orders-curation
tier: 2

owners:
  production: jane.doe@gsk.com
  technical:  raj.patel@gsk.com
  business:   finance-ops-lead@gsk.com

evidence:
  repo:   https://dev.azure.com/gsk/data/_git/orders-curation
  ci:     { system: azure-devops, pipeline: orders-curation-ci }
  cloud:  { subscription: sub-1234, resource_group: rg-orders-prod, tag: "project=orders-curation" }
  jobs:   { system: databricks, workspace: adb-prod-eu, job_prefix: orders_ }
  observability: { system: azure-monitor, workspace: law-data-prod }
  data:
    primary: curated/orders.csv
    source:  landing/orders_source.csv
    target:  curated/orders.csv
    measure: order_value
```

**The manifest does more work than the automation does.** A project that cannot say where its
alerting configuration lives has failed MON-01 before any tool ran. Filling it in *is* the readiness
conversation, held months earlier and in an hour rather than a workshop.

### 3.2 `declarations.yaml` — Level 0 assertions

```yaml
declarations:
  RES-06:
    stated: true
    by: raj.patel@gsk.com
    date: 2026-05-14
    note: "Restore tested into rg-orders-dr on 14 May, completed in 42 minutes."
  ACC-03:
    stated: true
    by: ops-lead@gsk.com
    date: 2026-06-02
    note: "Ops executed ops/runbooks/late-source.md end to end; no corrections needed."
```

A declaration without `by` and `date` is incomplete and reads as `missing`. The `note` is free text
and is what a reviewer actually reads.

### 3.3 `attestations.yaml` and `waivers.yaml`

Both already exist in the implementation. Unchanged, with the existing rules: attestations carry role,
signature, date and expiry; waivers carry reason, approver and expiry, and a blank is not a waiver.

### 3.4 Profile overlays

Three files, each tiny — they carry only what differs from the base.

```yaml
# profiles/tier-2.yaml
profile: tier-2
extends: base

verdict:
  requires_level: { blocker: 1 }      # tier 1 sets 2; tier 3 omits this

severity:                              # only the overrides
  OWN-04: major
  RES-07: minor
  PRF-04: minor

exclude: [REG-02, REG-07, SEC-11]      # not applicable at this tier
```

---

## 4 · Process

Seven stages. The ordering matters, and stage 2 is where most assurance tools quietly lie.

| # | Stage | What happens | Failure behaviour |
|---|---|---|---|
| 1 | **Resolve** | Tier → profile → the applicable subset of the 170, with per-item severity and level | Unknown tier is an error, not a default |
| 2 | **Collect** | Per item, by level: L0 reads `declarations.yaml`; L1 fetches and hashes the artefact; L2/L3 call the adapter | **Unavailable is a first-class result.** Never a pass |
| 3 | **Evaluate** | Deterministic rules, L2+ only. Rules are parsed to an AST and walked against a whitelist — a spec file cannot execute anything | A rule that cannot be evaluated is `error` |
| 4 | **Assemble** | Category-B items only: gather and cross-reference the material, and state the specific question a human must answer | Status is `needs_review` regardless of what comes back |
| 5 | **Verdict** | Pure rules over statuses, plus the profile's `requires_level` floor | No model involvement, ever |
| 6 | **Remediate** | Where a base agent can produce the missing artefact, produce it into `.opsgate/remediation/` | Proposals only. Nothing is applied |
| 7 | **Seal** | Hash the bundle, stamp the spec version and profile, take the signature | A failing gate cannot be signed as accepted |

### Status vocabulary

Extends what the implementation already has, with the two ladder statuses:

`pass` · `fail` · `declared` *(new)* · `evidenced` *(new)* · `needs_review` · `attested` · `missing` ·
`waived` · `error`

`error` and `missing` always fail closed. `declared` and `evidenced` satisfy an item but are excluded
from the maturity number and are subject to `requires_level`.

---

## 5 · The adapter contract — the anti-sprawl mechanism

**This is the decision that determines whether the system survives contact with a second platform.**

An adapter's only job is to emit a fixed field vocabulary for its domain, whatever the source. Rules
are written against the vocabulary, never against the tool.

| Domain | Fields the adapter must emit | Status |
|---|---|---|
| `test` | `suites · suites_failed · tests · coverage_pct · critical_path_coverage_pct` | built |
| `dq` | `defined · evaluated · passed · failed · pass_rate · failures[]` | built |
| `recon` | `source_rows · target_rows · row_variance · source_total · target_total · variance_pct` | built |
| `iac` | `declared_count · deployed_count · missing[] · unmanaged[] · changed[] · resources_differing` | built |
| `observability` | `slo_defined · slo_instrumented · alert_rules · failure_points · failure_points_alerted_pct · unalerted[]` | built |
| `finops` | `tagged · budget_set · monthly_budget · forecast · forecast_pct_variance` | built |
| `repo` | `files_scanned · secrets · findings[]` | built |
| `sandbox` | `executed · succeeded · exit_code · duration_s · timed_out` | built |
| `release` | `tagged_release · pipeline_exists · tests_gated · approvals_required` | **new** |
| `deps` | `pinned · total · eol_count · eol[]` | **new** |
| `access` | `excess_permissions · privileged_accounts` | built |
| `catalog` | `coverage_pct · unmasked_sensitive` | built |

Adding Jenkins alongside Azure DevOps means writing one adapter that emits the `test` and `release`
vocabularies. **Zero checks change.** That property is the reuse argument, and it is worth defending
against the first "we just need one custom field for this project".

---

## 6 · Classifying the 170 — five reasons, not one

"Cannot be automated" is five different problems, and conflating them is why teams either
over-automate or give up. Each maps to a different treatment.

| Reason it resists | Example items | Treatment | Can AI help? |
|---|---|---|---|
| **Needs judgement** | PRF-02, RES-11, DQ-05 | Category B: agent assembles, frames the question, human decides | **Yes — highest value** |
| **Needs accountability** | REG-01/02/03, ACC-09 | Attested. Automate the **envelope** — signed, right role, in date — never the content | Envelope only |
| **Needs an act to have happened** | RES-06, ACC-02, ACC-03, ACC-04, PRF-03 | **No amount of reading proves an event occurred.** Make the act self-evidencing: instrument the drill, capture the exit code | **No** |
| **Needs reading prose** | DOC-01, DOC-04, RES-02, MON-03 | Agent cross-references the document against recovered facts | **Yes** |
| **Needs org context** | OWN-05, ACC-05, SEC-12 | Directory / IAM lookup — automatable, just not by a model | Not an AI problem |

Row three deserves the most attention, because it contains every item I expect to be contested. The
honest answer is not to infer these but to make performing them cheap and self-evidencing — the
sandbox rollback already works this way, and the same trick generalises: **turn the act into its own
evidence rather than trying to detect it afterwards.**

---

## 7 · Where AI sits

**Earns its seat:** normalising messy evidence into the declared vocabulary; reading prose and
cross-referencing it against recovered facts; explaining *why* a check failed; drafting the missing
artefact; framing the specific question a human must answer.

**Must not:** compute the verdict, sign anything, decide whether a gap is acceptable, or be the
reason an item counts as verified.

That last clause is the load-bearing one. **If an item's status depends on a model's opinion, it is
category B and stays there.** No exceptions for convenience, because the exception is what gets
asked about in an audit.

Concrete assembly tasks worth building, in value order:

1. **Runbook coverage** — do the runbooks cover every failure mode in the RES register? (MON-03, DOC-03)
2. **Architecture currency** — does the architecture document match the recovered topology? (DOC-01)
3. **Undocumented failure points** — already built, and the template for the rest (RES-02)
4. **SLO realism** — declared target against sandbox-observed behaviour (PRF-02)
5. **Manifest drafting** — read the repo and propose the manifest, so the project team edits rather than authors

Item 5 is the one that most reduces adoption friction, and it is not obvious. Worth building early.

---

## 8 · What should be a skill, and what should not

| Thing | Skill? | Why |
|---|---|---|
| **The gate itself** | **Yes** — exists | Operator-facing, portable, adoptable without adopting the tooling |
| **A "prepare for the gate" self-check** | **Yes — build this** | Project-side. They run it because it helps *them*. Adoption without politics |
| Evidence adapters | **No** | Connectors behind a contract. One skill per adapter is over-decomposition and buys version skew |
| The assembly tasks in §7 | **Three or four, not thirty** | Prompt-heavy and judgement-adjacent, so genuinely skill-shaped |

The project-side self-check is an **adoption strategy, not a feature**. An Ops-side gate is compliance
and gets resisted; the same spec handed to a project team as "see what you'd fail, and I'll draft your
manifest" is help and gets adopted. Same file, opposite politics.

---

## 9 · PoC scope

### 9.1 Where the existing 19 checks land

The implemented spec already covers roughly 25 of the 170 items — the honest starting maturity number
is **~15%**.

| Implemented check | New checklist ID(s) |
|---|---|
| FUN-01 test suite passes | FUN-01 |
| FUN-02 critical-path coverage | FUN-02 |
| FUN-03 DQ expectations | DQ-01 + DQ-02 |
| FUN-04 reconciliation | FUN-07 + DQ-04 |
| NFR-01 SLO defined | PRF-01 |
| NFR-02 SLO realistic | PRF-02 |
| NFR-03 failure points alerted | MON-01 |
| NFR-04 cost tagged/budgeted | FIN-01 + FIN-02 + FIN-05 |
| NFR-05 failure modes documented | RES-01 + RES-02 |
| NFR-06 rollback demonstrated | REL-05 |
| CFG-01 no drift | CFG-05 + CFG-06 |
| CFG-02 no hardcoded secrets | CFG-01 |
| CFG-03 infra reproducible | REL-04 |
| SEC-01 least privilege | SEC-01 |
| SEC-02 classified and masked | SEC-05 + SEC-06 |
| REG-01 GxP assessment | REG-01 + REG-02 |
| REG-02 DPIA | REG-03 |
| ORG-01 owner and rota | OWN-01 + OWN-04 |
| ORG-02 vendor dependency | DEP-06 |

> ⚠ **Migration hazard.** The two ID schemes collide with different meanings — implemented `CFG-01` is
> drift, checklist `CFG-01` is secrets. The migration must be a **rename with a mapping table**, not an
> overlay. A silent mis-map here would be very hard to spot and would invalidate every bundle produced
> before it was found.

### 9.2 New items to reach Level 2 in the PoC

Chosen because they need only a repository and one cloud account — no programme, no integration board:

| Item | Adapter | Rule sketch |
|---|---|---|
| REL-01 tagged release | `release` | `tagged_release` |
| REL-02 pipeline exists | `release` | `pipeline_exists` |
| REL-03 tests gate the pipeline | `release` | `tests_gated` |
| REL-06 approval not the deployer | `release` | `approvals_required >= 1` |
| MNT-02 dependencies pinned | `deps` | `pinned == total` |
| MNT-03 no EOL dependency | `deps` | `eol_count == 0` |
| DQ-09 freshness declared | `dq` | `freshness_sla_declared` |
| DQ-10 volume expectation declared | `dq` | `volume_bounds_declared` |

That takes the PoC to roughly **33 of 170 verified — about 19%**, from two new adapters.

### 9.3 Build order

1. **Ladder in the engine** — `level` on the check, `declared`/`evidenced` statuses, `requires_level`
   in the verdict. Everything else depends on this.
2. **Manifest + declarations loaders**, and the ID migration with the mapping table above.
3. **Three tier profiles** as overlays.
4. **`release` and `deps` adapters** — the cheapest new Level 2 items.
5. **Manifest-drafting assembler** — cuts adoption friction more than any check.
6. **Runbook-coverage assembler** — the highest-value category B item not yet built.
7. **Self-check packaging** — the same engine, project-side.

Steps 1–3 are a working end-to-end gate over all 170 items. Steps 4 onward only move the number.

---

## 10 · Adoption — earning the mandate a PoC cannot be granted

The PoC's goal is not to hold the mandate. It is to make granting it cheap.

| # | Move | Permission needed | What it produces |
|---|---|---|---|
| 1 | **Retrospective run** on 2–3 already-accepted projects | **None** — artefacts exist, nobody is blocked | "Passed manual review, would have failed 6 blockers, and 2 of 3 early incidents map to them" |
| 2 | **Shadow** the next real handover | Informal | "It found things the review missed" |
| 3 | **Publish the self-check** | **None** — opt-in | Adoption nobody had to mandate |
| 4 | **Ask for attachment** | One sponsor | Formalises what already happens |

Steps 1 and 3 need nobody's permission — that is the point. On step 1, fill the manifest yourself so
the project team carries zero cost.

**Political caution on step 1:** running this against a colleague's live system is warm. Frame every
output as *what the bar would have caught*, never *what that team got wrong*, and anonymise where you
can. Run the first one on a project you were close to, so you can be blunt about your own.

**On step 4 — ask for attachment, not veto.** "The evidence bundle is attached to the acceptance
record" is administrative and gets waved through. "The gate blocks production" is political and gets
negotiated into nothing. Attachment does nearly the same work, because a failing or unsigned bundle on
the record is visible to everyone who looks at that record afterwards. The blocking behaviour then
emerges without anyone having to grant it.

---

## 11 · Measures

| Measure | Definition | Why it is the right one |
|---|---|---|
| **Maturity** | Share of applicable items at Level 2+ | The one number designed to rise release on release |
| **First-pass rate** | Projects passing at first submission | Rises as the self-check spreads — measures shift-left, not gate strictness |
| **Shift-left ratio** | Defects caught at the gate vs found in production | The outcome that justifies the whole thing |
| **Cycle time** | Handover elapsed time | Days → hours is the number the business hears |
| **Waiver rate per domain** | Waivers ÷ applicable items | A rising rate is a signal about the bar, not about the projects |

**Deliberately not measured: a readiness score.** "87% ready" is meaningless when three blockers are
open, and a percentage invites negotiation over which blocker matters. Blockers are binary. The only
percentage published is maturity.

---

## 12 · Risks

| Risk | Mitigation |
|---|---|
| **The tooling decides the bar** — hard items get quietly dropped | Level 0 exists precisely for this. An unautomatable item stays on the list, declared and signed |
| **Level 0 becomes self-certification theatre** | `declared` ≠ `pass`, excluded from maturity, and `requires_level` raises the floor per tier |
| **Field vocabulary sprawl** — one custom field per project | D2 is non-negotiable. Custom fields go in the `note`, never in the rule |
| **ID collision during migration** | Explicit mapping table, rename not overlay, and re-run one known bundle before and after to diff |
| **Manifest never gets filled** | Convention over configuration; manifest-drafting assembler; retrospective runs where the gate author fills it |
| **Retrospective run reads as blame** | "What the bar would have caught", anonymised, first one on her own project |
| **170 items reads as bureaucracy** | It is a superset, not an agenda. Tier 3 profile should be small enough to be obviously reasonable |

---

## 13 · Open items

1. Confirm GSK's actual evidence platforms — D5 assumes Azure + Databricks + Azure Monitor.
2. Define the three tier profiles concretely: which items each excludes, and the severity overrides.
3. Down-grade pass on severity — the 89 blockers are a tier-1 reading and need per-profile grading (D4
   makes this cheap, but the grading still has to be done).
4. Decide the `requires_level` floor per tier. Proposed: tier 1 = 2, tier 2 = 1, tier 3 = none.
5. Pick the retrospective projects, and confirm the incident data needed to make the step-1 argument.
