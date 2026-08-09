# Ops Acceptance Checklist — Data Engineering Projects

**Plain English, phase 1.** This is the bar a data engineering project must clear before the
Ops team accepts it into production. It is written entirely from the *receiving* side.

Read every item as: **"Has the project team done this, and is the evidence in our hands?"**
Not as work for Ops to do. Ops is not fixing the gaps here — Ops is establishing whether the
gaps exist, and refusing to inherit a system whose gaps are unknown.

Deliberately no tooling in this document. How each item gets verified — automatically, by an
agent, or by a named signature — is [phase 2](#phase-2--turning-this-into-a-gate).

## How to read the columns

| Column | Meaning |
|---|---|
| **#** | Stable identifier. It survives into the machine-readable spec, so it is worth not renumbering. |
| **Ops is checking that…** | The question, in the words an Ops lead would actually use. |
| **Handover evidence expected** | What must physically be handed over or pointed at. "They said so in a meeting" is not evidence. |
| **Bar** | First-pass severity. **Blocker** = no production without it. **Major** = accept only with a dated remediation plan. **Minor** = record it and move on. Argue with these; they are a proposal. |

---

## 1. Ownership & support model — `OWN`

Nothing else on this list matters if there is no name attached. This section is first on purpose:
an unowned system in production is an incident with a delay fuse.

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| OWN-01 | There is a **named production owner** — a person, not a team mailbox | Name, role, and their manager's name | Blocker |
| OWN-02 | There is a **named technical owner** who can change the code after handover | Name, and confirmation they have time allocated | Blocker |
| OWN-03 | The **business owner** who cares if the data is wrong is identified | Name and the decision the data feeds | Blocker |
| OWN-04 | An **on-call rota** covers this system, with the hours it is covered | Rota, effective dates, and who is on it this week | Blocker |
| OWN-05 | The **escalation path** is written down — who is called, in what order, after how long | Escalation matrix with contact routes | Blocker |
| OWN-06 | The **support model** is agreed: what Ops handles, what goes back to the project team | RACI or a one-page support agreement | Blocker |
| OWN-07 | A **warranty / hypercare period** is agreed, with an end date and exit criteria | Dates and what must be true to exit | Major |
| OWN-08 | The system is **registered in the service catalogue / CMDB** | Record ID | Major |
| OWN-09 | The **business criticality tier** is assigned and agreed with Ops | Tier, and the SLA that tier implies | Blocker |
| OWN-10 | Ops was **involved before handover day**, not presented with a finished system | Evidence of earlier design/ops review | Minor |

## 2. Functional correctness — `FUN`

Does it do what it is supposed to do, and can anyone demonstrate that without the original author?

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| FUN-01 | An **automated test suite exists and passes** on the release being handed over | Test run output against the release artefact | Blocker |
| FUN-02 | **Critical business logic is covered** by tests, not just the plumbing | Coverage report, with the critical paths named | Major |
| FUN-03 | **UAT was completed and signed off** by the business owner | Signed UAT record, dated, with scope | Blocker |
| FUN-04 | **Known defects are listed and agreed**, not discovered later | Open defect register with severity and owner | Blocker |
| FUN-05 | The system has **run successfully in production-like conditions** for a meaningful period | Run history from a pre-production environment | Major |
| FUN-06 | **Historical / backfill data was loaded and validated**, if the system needs it | Backfill record and its validation result | Major |
| FUN-07 | Outputs have been **reconciled against the source of truth** end to end | Reconciliation result with tolerance and variance | Blocker |
| FUN-08 | The **first production run has a defined success criterion** Ops can check | What "good" looks like on day one, in numbers | Major |

## 3. Data quality & integrity — `DQ`

The failure mode unique to data systems: everything is green and the numbers are wrong.

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| DQ-01 | **Data quality rules are declared** — and there is more than zero of them | The rule set, as a file, not a description | Blocker |
| DQ-02 | The rules have a **pass threshold** agreed with the business owner | The threshold and who agreed it | Blocker |
| DQ-03 | Rules cover **completeness, validity, uniqueness and ranges** on key fields | Rule set mapped to the critical columns | Major |
| DQ-04 | **Source-to-target reconciliation runs routinely**, not just once at UAT | The reconciliation job and its schedule | Blocker |
| DQ-05 | The behaviour on a **DQ breach is defined** — stop, quarantine, or continue and alert | Documented policy per severity | Blocker |
| DQ-06 | **Late-arriving data** is handled, and the handling is stated | Design note and a worked example | Major |
| DQ-07 | **Duplicate and out-of-order records** are handled deterministically | Design note and a worked example | Major |
| DQ-08 | **Schema changes at source** are detected rather than silently absorbed | Detection mechanism and who is notified | Blocker |
| DQ-09 | **Data freshness expectations** are stated — how stale is too stale | The freshness SLA per dataset | Blocker |
| DQ-10 | **Row-count and volume expectations** are stated, so a silent 90% drop is caught | Expected volumes and tolerance | Major |
| DQ-11 | **Nulls in business-critical fields** have an explicit expected rate | Per-field expectation | Major |
| DQ-12 | Someone is **named to triage a DQ failure** and decide whether to release the data | Name or role, and their decision authority | Blocker |

## 4. Dependencies & integration contracts — `DEP`

Most production incidents in data platforms originate outside the system that gets blamed.

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| DEP-01 | Every **upstream source is listed**, with its owner and contact | Dependency register | Blocker |
| DEP-02 | Every **downstream consumer is listed** — who breaks if this stops | Consumer register with criticality | Blocker |
| DEP-03 | There is an **agreed data contract** with each source: schema, timing, volume | The contract or the agreed equivalent | Major |
| DEP-04 | Ops will be **notified before an upstream change**, and the route is agreed | Change-notification agreement | Major |
| DEP-05 | The behaviour when a **source is late or unavailable** is defined | Documented policy — wait, skip, use prior, fail | Blocker |
| DEP-06 | **Third-party and vendor dependencies** are identified, with their support terms | Vendor list with contract and support hours | Major |
| DEP-07 | **Shared platform dependencies** are known — clusters, warehouses, gateways | List, with who else uses them | Major |
| DEP-08 | Downstream consumers **know the freshness and SLA** they are being given | Published SLA and evidence it was communicated | Major |

## 5. Jobs, scheduling & orchestration — `JOB`

The section Ops lives in at 3am. Every item here is a question about what happens when a run
goes wrong at the worst possible moment.

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| JOB-01 | Every job's **schedule and trigger** is documented, including why that window | Schedule inventory with rationale | Blocker |
| JOB-02 | The **dependency order between jobs** is explicit, not implied by timing | Dependency graph | Blocker |
| JOB-03 | Jobs are **idempotent** — re-running the same job does not duplicate or corrupt | Statement plus evidence it was tested | Blocker |
| JOB-04 | A failed job can be **safely restarted**, and the restart procedure is written | Restart runbook per job | Blocker |
| JOB-05 | **Partial failure is recoverable** — a job that dies halfway leaves a known state | Design note on transactional boundaries | Blocker |
| JOB-06 | **Backfill and replay** are supported, with the procedure documented | Backfill runbook and its guard rails | Major |
| JOB-07 | Jobs have a **timeout / maximum runtime**, and something happens when it is hit | Timeout values and the action on breach | Major |
| JOB-08 | **Concurrent runs are prevented or safe** — no accidental double execution | Locking or concurrency design | Blocker |
| JOB-09 | **Retry behaviour** is defined: how many, how far apart, and when to stop | Retry policy per job | Major |
| JOB-10 | The **expected runtime is known**, so an overrun is detectable | Baseline durations from pre-production | Major |
| JOB-11 | **Manual intervention procedures** exist for the cases automation cannot cover | Runbook entries for each known case | Major |
| JOB-12 | The **run calendar is clear** — holidays, month-end, freeze periods | Calendar and any special handling | Minor |
| JOB-13 | **Job ownership at the individual job level** is recorded, for multi-team pipelines | Owner per job | Minor |

## 6. Configuration & environments — `CFG`

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| CFG-01 | **No credentials or secrets are in code**, config files, or notebooks | Scan result over the repository | Blocker |
| CFG-02 | Secrets are held in an **approved secret store**, with rotation defined | Store location and rotation schedule | Blocker |
| CFG-03 | **Configuration is externalised** — environments differ by config, not by code | Config layout, and a diff across environments | Blocker |
| CFG-04 | **Environment parity** is real: pre-production resembles production meaningfully | Comparison of the two, with differences listed | Major |
| CFG-05 | **Infrastructure is defined as code**, and the code matches what is deployed | IaC repository and a drift report | Blocker |
| CFG-06 | There is **no undocumented manual configuration** in production | Confirmation, plus the drift report backing it | Blocker |
| CFG-07 | The **process to change config after handover** is agreed — who, how, approved by whom | Change procedure | Major |
| CFG-08 | **Environment-specific values** are inventoried: endpoints, paths, quotas, accounts | Configuration inventory | Major |
| CFG-09 | **Feature flags and toggles** are listed, with their production state and owner | Flag inventory | Minor |

## 7. CI/CD & release — `REL`

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| REL-01 | All code is in **version control**, and the handed-over version is tagged | Repository and the release tag | Blocker |
| REL-02 | A **deployment pipeline exists** — deployment is not a person following notes | Pipeline definition and a successful run | Blocker |
| REL-03 | The pipeline **runs the tests and blocks on failure** | Pipeline configuration showing the gate | Blocker |
| REL-04 | Deployment is **repeatable from scratch** into an empty environment | Evidence of a clean deploy | Major |
| REL-05 | **Rollback has been demonstrated**, not asserted | Record of an executed rollback and its duration | Blocker |
| REL-06 | The **release approval path** is defined, and it is not the deploying engineer alone | Approval process | Blocker |
| REL-07 | **Artefacts are versioned and promoted**, not rebuilt per environment | Artefact repository and promotion flow | Major |
| REL-08 | **Release notes** accompany the handover release | The notes, listing what changed | Minor |
| REL-09 | A **hotfix path** exists that is faster than the standard release but still gated | Documented emergency change procedure | Major |
| REL-10 | **Database and schema migrations** are versioned, ordered, and reversible | Migration tooling and a reversal example | Blocker |
| REL-11 | The **branching and merge strategy** is documented for whoever maintains it next | Short written convention | Minor |

## 8. Performance, capacity & SLA — `PRF`

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| PRF-01 | A **service level objective is declared** — completion time, freshness, or availability | The SLO, in numbers, with the measurement window | Blocker |
| PRF-02 | The SLO was **agreed with the business owner**, not chosen by the engineers alone | Evidence of agreement | Blocker |
| PRF-03 | The system was **tested at production data volume**, not a sample | Load or volume test result | Blocker |
| PRF-04 | **Peak conditions** were tested — month-end, year-end, campaign spikes | Peak test result | Major |
| PRF-05 | **Expected data growth** is stated, with the horizon it remains viable | Growth projection and headroom estimate | Major |
| PRF-06 | There is **headroom in the current sizing**, quantified | Current utilisation against provisioned capacity | Major |
| PRF-07 | **Behaviour under overload is known** — degrade, queue, or fail | Documented and, ideally, observed | Major |
| PRF-08 | **Performance baselines are recorded**, so future regression is detectable | Baseline runtimes and resource usage | Major |
| PRF-09 | **Known performance bottlenecks** are declared rather than discovered | Written list with mitigation status | Minor |
| PRF-10 | The **cost of meeting the SLA** is understood — the SLA is affordable | Cost implication of the SLO | Minor |

## 9. Resilience, failure & recovery — `RES`

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| RES-01 | **Failure modes are documented** — what can break, and what happens when it does | Failure mode register | Blocker |
| RES-02 | The register covers **every component in the deployed topology**, not just the obvious ones | Register cross-checked against the architecture | Blocker |
| RES-03 | Each failure mode has a **documented recovery procedure** | Runbook entry per failure mode | Blocker |
| RES-04 | **RTO and RPO are stated and agreed** for this system | The targets, agreed with the business owner | Blocker |
| RES-05 | **Backups exist**, with the retention period stated | Backup configuration | Blocker |
| RES-06 | **Restore has actually been tested**, and how long it took is known | Restore test record with duration | Blocker |
| RES-07 | **Disaster recovery is defined**, at the level the criticality tier requires | DR plan, and evidence of a test if tier demands | Major |
| RES-08 | **Dependency failure is handled** — the system degrades rather than corrupts | Design note per critical dependency | Blocker |
| RES-09 | There is a **dead-letter or quarantine path** for records that cannot be processed | The mechanism and who reviews it | Major |
| RES-10 | **Data loss scenarios** are identified, with the recovery route for each | Written analysis | Blocker |
| RES-11 | **Single points of failure** are declared and consciously accepted | List, with sign-off on each | Major |

## 10. Monitoring, alerting & observability — `MON`

The test: if this system fails tonight, does Ops find out from a monitor or from a user?

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| MON-01 | **Every failure mode in the RES register has a corresponding alert** | Coverage map: failure mode → alert | Blocker |
| MON-02 | **Alerts route to the on-call rota**, and the route has been tested end to end | A test alert that actually reached someone | Blocker |
| MON-03 | **Every alert has a runbook**, linked from the alert itself | Alert-to-runbook mapping, no gaps | Blocker |
| MON-04 | **Job failure raises an alert** — no job fails silently | Alert definitions per job | Blocker |
| MON-05 | **Job overrun raises an alert**, not just outright failure | Duration-based alerting | Major |
| MON-06 | **Data freshness is monitored** — a job that succeeds while producing stale data is caught | Freshness monitor per dataset | Blocker |
| MON-07 | **Volume anomalies are monitored** — a successful run with 10% of the rows is caught | Volume monitor with bounds | Blocker |
| MON-08 | **Data quality results are monitored continuously**, not only at the gate | DQ monitoring configuration | Blocker |
| MON-09 | **Logs are centralised, searchable, and retained** for a stated period | Log destination and retention policy | Blocker |
| MON-10 | Logs are **useful** — they identify the run, the stage and the record on failure | A sample log from a real failure | Major |
| MON-11 | A **dashboard exists that Ops can read** without knowing the internals | The dashboard, walked through with Ops | Major |
| MON-12 | **Alert severity is graded**, and severities map to response times | Severity definitions | Major |
| MON-13 | **Alert noise has been tuned** — no known alerts that fire routinely and get ignored | Alert history from pre-production | Major |
| MON-14 | **Nothing alerts to an individual's inbox** or a personal channel | Confirmation of routing targets | Blocker |
| MON-15 | **SLO breach is itself alerted**, not just component failure | SLO monitor | Major |
| MON-16 | **Upstream dependency health is visible**, so blame is fast to assign | Dependency monitoring or documented route | Minor |

## 11. Security & access — `SEC`

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| SEC-01 | Access follows **least privilege** — no broad or inherited admin rights | Access review output | Blocker |
| SEC-02 | **Service accounts are used for automation**, never a person's credentials | Service account inventory with owners | Blocker |
| SEC-03 | **Credential rotation** is defined, scheduled, and will not break the system | Rotation procedure and schedule | Blocker |
| SEC-04 | **Data is encrypted** in transit and at rest, per policy | Configuration evidence | Blocker |
| SEC-05 | **Sensitive and personal data is classified**, and the classification is complete | Classification record per dataset | Blocker |
| SEC-06 | Sensitive fields are **masked or tokenised** wherever policy requires | Masking configuration and coverage | Blocker |
| SEC-07 | **Non-production environments do not hold unmasked production data** | Confirmation and how it is enforced | Blocker |
| SEC-08 | **Access is auditable** — who read or changed what is recorded | Audit log configuration | Blocker |
| SEC-09 | A **security review or assessment was completed** for this system | The assessment, dated, with findings closed | Blocker |
| SEC-10 | **Network exposure is understood** — what is reachable from where | Network diagram or equivalent | Major |
| SEC-11 | **Vulnerability scanning covers this codebase and its dependencies** | Latest scan result | Major |
| SEC-12 | **Joiner / mover / leaver** handling for this system's access is covered | Access management process | Major |

## 12. Compliance & regulatory — `REG`

Adjust this section to the regulatory context. In a GxP environment it is usually the section
that decides the date, so it belongs early in the project, not at handover.

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| REG-01 | **Regulatory applicability is determined** — is this system GxP relevant, or not | Impact assessment, signed and dated | Blocker |
| REG-02 | Where applicable, **validation or qualification is complete** | Validation documentation | Blocker |
| REG-03 | A **data privacy impact assessment** is complete where personal data is involved | DPIA, approved by the data protection officer | Blocker |
| REG-04 | **Data retention and deletion rules** are defined and implemented | Retention policy and its implementation | Blocker |
| REG-05 | **Data lineage is documented** end to end, source to consumption | Lineage documentation or captured lineage | Major |
| REG-06 | **Audit trail requirements** are met — records of change are kept as required | Audit configuration | Blocker |
| REG-07 | **Cross-border data movement** is identified and cleared | Data flow map with jurisdictions | Major |
| REG-08 | **Change control requirements** post-handover are understood by Ops | The applicable change control procedure | Blocker |
| REG-09 | **Attestations carry a named signatory, a date and an expiry** — none are open-ended | The attestation records | Blocker |

## 13. Cost & FinOps — `FIN`

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| FIN-01 | All resources are **tagged to a cost centre and this project** | Tagging coverage report | Blocker |
| FIN-02 | A **budget is set**, and someone owns it | Budget figure and owner | Blocker |
| FIN-03 | The **run-rate is known** — what this costs per month in steady state | Cost figure from a representative period | Blocker |
| FIN-04 | The **cost per run or per unit of data** is known, so growth can be forecast | Unit economics figure | Major |
| FIN-05 | **Forecast against budget** is within an agreed tolerance | Forecast and variance | Major |
| FIN-06 | **Budget alerts** are configured and route to the budget owner | Alert configuration | Major |
| FIN-07 | **Obvious cost inefficiencies were addressed** before handover, not left for Ops | Optimisation summary or a stated backlog | Major |
| FIN-08 | **Idle and non-production resources shut down** outside working hours where possible | Schedule or a stated exception | Minor |
| FIN-09 | **Cost of the DR and backup posture** is included in the run-rate | Included in the figures | Minor |
| FIN-10 | What was **decommissioned** — the cost this replaces — is stated | Decommissioning plan and saving | Minor |

## 14. Documentation & runbooks — `DOC`

The test for this whole section: could a competent engineer who has never met the project team
operate this system from the documents alone?

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| DOC-01 | An **architecture document exists and matches what is deployed** | The document, dated, with a diagram | Blocker |
| DOC-02 | The **data model and data dictionary** are documented for the outputs | Dictionary covering the consumed tables | Major |
| DOC-03 | **Runbooks exist for every routine operational task** | Runbook set | Blocker |
| DOC-04 | **Runbooks are written for someone who did not build the system** | Ops has read them and confirmed | Blocker |
| DOC-05 | A **troubleshooting guide** covers the known failure symptoms | The guide | Major |
| DOC-06 | **Known issues and workarounds** are documented rather than remembered | Known-issues list | Major |
| DOC-07 | **Design decisions and their rationale** are recorded — why it is built this way | Decision record or design section | Minor |
| DOC-08 | Documentation lives **where Ops can find it**, not in a personal drive | Location, with Ops access confirmed | Blocker |
| DOC-09 | Documentation has an **owner and a review date** | Owner and next review | Minor |
| DOC-10 | A **contact list** for every dependency is included | The list, with routes not just names | Major |

## 15. Maintainability & day-2 change — `MNT`

Handover is not the end of the system's life. This section is about the next two years.

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| MNT-01 | The code follows a **stated standard**, so a new engineer can work in it | The standard, and evidence it is applied | Minor |
| MNT-02 | **Dependencies are pinned and inventoried** | Dependency manifest | Major |
| MNT-03 | **No dependency is already end-of-life or out of support** | Version audit against support dates | Blocker |
| MNT-04 | The **upgrade path** for platform and runtime versions is known | Statement of the next forced upgrade and when | Major |
| MNT-05 | **Technical debt taken to hit the date** is declared, not hidden | Debt register with impact | Major |
| MNT-06 | The **process for a schema change** after handover is agreed | Change procedure covering consumers | Major |
| MNT-07 | The **test suite is maintainable** — it can be run and extended by someone new | Ops or a third party has run it themselves | Major |
| MNT-08 | **Development environment setup** is documented and reproducible | Setup instructions, followed successfully once | Major |
| MNT-09 | **Reference and lookup data** has a maintenance owner and update procedure | Owner and procedure | Major |
| MNT-10 | **Housekeeping is automated** — log rotation, temp cleanup, partition management | The jobs, scheduled | Major |
| MNT-11 | Anything **replaced by this system is decommissioned**, with a date | Decommissioning plan | Minor |

## 16. Knowledge transfer & formal acceptance — `ACC`

| # | Ops is checking that… | Handover evidence expected | Bar |
|---|---|---|---|
| ACC-01 | **Walkthrough sessions were held** with the Ops team who will run it | Session record and attendees | Blocker |
| ACC-02 | **Ops has operated the system themselves** at least once, supervised | Record of the exercise | Blocker |
| ACC-03 | Ops has **executed a runbook end to end** and it worked as written | Which runbook, and the outcome | Blocker |
| ACC-04 | A **failure was simulated and Ops handled it** — the alert fired, the runbook worked | Record of the drill | Major |
| ACC-05 | Ops has **all required access** in production, verified before acceptance | Access confirmed by Ops, not asserted | Blocker |
| ACC-06 | **Open items are agreed with dates and owners**, not left ambiguous | Agreed remediation plan | Blocker |
| ACC-07 | Every **exception or waiver has a reason, an approver and an expiry** | Waiver register — no blanks, no open-ended entries | Blocker |
| ACC-08 | The **hypercare exit criteria** are agreed in advance | Written criteria and the date | Major |
| ACC-09 | **Acceptance is signed** by a named person on the Ops side | The signed acceptance record | Blocker |
| ACC-10 | The signature is **against a specific version** of the system | Version or release tag on the acceptance record | Major |

---

## Working notes

**On rejecting a handover.** The point of a written bar is that "no" stops being a personality
trait and becomes a reading of a list. Every blocker above should be defensible as *"this is
what we would need at 3am"* — if an item cannot be defended that way, it is a Major, not a
Blocker.

**On items that will be contested.** Expect pushback on RES-06 (restore actually tested),
REL-05 (rollback demonstrated), ACC-02 and ACC-03 (Ops runs it themselves), and PRF-03 (tested
at production volume). Each is contested for the same reason: they require doing the thing
rather than describing it. That is precisely why they are on the list.

**On the shape of the list.** 170 items across 16 sections — far too many to walk through in a
meeting, and that is fine, because it is not a meeting agenda. It is the superset from which a
per-project profile is drawn: a tier-1 regulated pipeline uses nearly all of it, an internal
reporting job uses a fraction. The profile is selected by the criticality tier in OWN-09.

**On the severity split — this needs a second pass.** As graded, it is 89 Blocker · 65 Major ·
16 Minor. Over half the list blocking is a tier-1 reading, and if every profile inherits it the
word stops meaning anything. The next edit should be a deliberate down-grading pass per tier,
with a target of roughly 30–40 blockers for a mid-tier system. Grading severity *per profile*
rather than globally is probably the right end state.

## Phase 2 — turning this into a gate

Not started. When it is, each item above gets classified by how it can honestly be verified:

| Class | Meaning |
|---|---|
| **Auto** | A rule over evidence a machine can collect. The verdict is deterministic. |
| **Assembled** | An agent gathers and cross-references the material; a human makes the call. |
| **Attested** | Cannot be automated. A named person signs, dated, with an expiry. |
| **Waived** | Explicitly not applicable, with a reason, an approver and an expiry. |

The share of items in **Auto** is the maturity number, and it is designed to rise release on
release. Nineteen checks are implemented today in
[DataPlatformSuite](https://github.com/basantchoudhary/DataPlatformSuite) — this document is
the full bar those nineteen are a first slice of.

One rule carries over from that implementation and should govern the classification work:
**nothing passes by absence.** An item that cannot be verified is not an item that passed.
