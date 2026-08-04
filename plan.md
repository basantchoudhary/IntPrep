# Interview Prep — Overall Plan

Senior Solution Engineer, EAI @ GSK. Criteria in [evaluation-criteria.md](evaluation-criteria.md).

## The narrative spine

One story, told top-down, that answers all three themes:

> A **North Star Data & AI Platform** (T1 — the reference architecture) with a
> **deterministic-first control plane** whose gates and assurance checks are
> productised as an **Ops Gate** capability (T2 — the assurance bar), built from a
> small set of **reusable base agents** that other engagements compose for
> themselves (T3 — pattern reuse and technical authority).

Each artefact below is deliberately positioned as *reusable*, not bespoke — that is
the word the grade bar repeats in all three themes.

---

## Component 1 — North Star Architecture (anchors T1)

- Reference: [architecture/north-star-platform.md](architecture/north-star-platform.md)
- Diagram: `architecture/north-star-platform.png` *(to be added)*
- **Role in the story:** the domain reference design. Everything else hangs off the
  Horizontal Platform Services row — that row *is* the reuse surface.
- **Still to prepare:** the failure-mode / resilience view. T1 explicitly names
  "designing for resilience, failure modes" and the current diagram is a capability
  map, not a failure analysis. Needs a companion view or talk-track.

## Component 2 — CostAgent (proof it is real — T1 + T2)

- Already built and running on her laptop.
- **Deliverables:** a short slide set (problem → where it sits on the North Star →
  deterministic vs agentic split → results) followed by a live demo.
- **Role in the story:** converts the architecture from a diagram into shipped,
  demonstrable engineering. Maps to the **Cost & FinOps Service** block.
- **Still to prepare:** slides, demo script with a safe fallback (recorded run or
  captured output) in case live execution fails in the room, and the measurable
  number — cost identified/avoided, or time saved vs manual analysis.

## Component 3 — Suite of Agents (anchors T2 + T3)

Two tiers. The tiering is itself the T3 argument: base agents are the reusable
patterns; the composite agents are the proof that other capabilities were built
from them rather than from scratch.

### Base agents (the reusable primitives)

| Agent | Purpose |
|---|---|
| **Sandbox Creator** | Stand up an isolated, governed environment to evaluate or reproduce a workload |
| **Test Framework Generator** | Generate the test scaffold — unit, integration, data, regression — for a given project |
| **Data Expectation Services Agent** | Derive and enforce data quality rules, profiles and expectations |
| **Reverse Engineer Infra-as-Code** | Recover IaC / an accurate infra model from a deployed or handed-over estate |

### Composite agents (built by composing the base agents)

| Agent | Purpose | Composed from |
|---|---|---|
| **Ops Gate Agent** | Ops-team evaluation of a project *before* accepting it into production | Sandbox Creator, Test Framework Generator, Data Expectation Services, Reverse Engineer IaC |
| **Ops Self-Monitoring Agent** | Continuous readiness / drift monitoring post-handover | Data Expectation Services, observability signals |
| **Ops Self-Healing Agent** | Diagnose → remediate → validate, with a human gate on action class | Self-Monitoring + Sandbox Creator (validate before apply) |
| **Databricks Spark Code Modernization Agent** | Modernize / optimize legacy Spark & Databricks workloads | Test Framework Generator (safety net), Sandbox Creator (verify) |

### Ops Gate Agent — the T2 centrepiece

Built as a **skill**, so it is genuinely portable and adoptable by other teams —
that portability is the "reused by other engagements" evidence the T2 bar asks for.

The key design decision to be able to defend in the room is the
**deterministic / agentic boundary**:

- **Deterministic** — the checklist itself, policy and security gates, pass/fail
  thresholds, evidence capture, the approval decision. Anything that must be
  auditable, repeatable, and regulator-explainable.
- **Agentic** — interpreting an unfamiliar codebase or estate, explaining *why* a
  check failed, proposing remediation, summarising risk for a human approver.
- **Never agentic** — the final production-acceptance verdict. A model does not
  approve a GxP-relevant deployment; it assembles the evidence a human signs.

This boundary is also the regulatory-alignment answer for T2, and it is the
strongest single point she can make: most people demo an agent that decides;
this one is explicitly designed so the agent *doesn't*.

---

## Coverage check against the criteria

| Theme | Covered by | Gap to close |
|---|---|---|
| **T1** Architecture & integration | North Star architecture + CostAgent as built proof | Resilience / failure-mode view; a real past engagement where her design was reused |
| **T2** Assurance & regulatory | Ops Gate Agent as the assurance bar; deterministic control plane | The checklist artefact itself; measurable reduction in production risk / rework |
| **T3** Stakeholders & reuse | Base → composite agent tiering; Ops Gate shipped as a skill | Named adopters / audiences; mentoring evidence |

## Decks

Each capability has two decks: **architecture** for the room, **design** for the drill-down.

| Capability | Architecture (2 pages) | Design (+ backups) |
|---|---|---|
| Ops Gate | [ops-gate-architecture.html](agents/ops-gate/ops-gate-architecture.html) | [ops-gate-design.html](agents/ops-gate/ops-gate-design.html) · [checks.sample.yaml](agents/ops-gate/checks.sample.yaml) |
| Ops Self-Monitoring & Self-Healing | [ops-self-healing-architecture.html](agents/ops-self-healing/ops-self-healing-architecture.html) | [ops-self-healing-design.html](agents/ops-self-healing/ops-self-healing-design.html) · [policy.sample.yaml](agents/ops-self-healing/policy.sample.yaml) |
| Platform | [north-star-simplified.html](architecture/north-star-simplified.html) | [north-star-platform.md](architecture/north-star-platform.md) |
| Principles | [principles.html](architecture/principles.html) | [principles.md](architecture/principles.md) |

Present the architecture decks. Open a design deck only if the panel drills.

## Build repository

Code lives in a separate repo: **DataPlatformSuite** — https://github.com/basantchoudhary/DataPlatformSuite

One repo, not several. The base agents and Ops Gate share one contract (the check
spec), so splitting them buys version skew and nothing else — and the thesis is
composability, which five repositories would contradict. Each agent is a package
depending only on `contracts/`, so the monorepo is a convenience rather than a
coupling: any agent could be extracted without unpicking imports. **Repo strategy
is not the same thing as module boundaries** — worth saying explicitly if asked.

Working today: `python3 -m opsgate check --target examples/sample_project` runs a
real 19-check assessment with zero dependencies, produces a signed evidence
bundle, drafts the missing artefacts via the base agents, and emits day-2
monitors. 21 tests pass.

## Open items

1. ~~Add the North Star PNG to `architecture/`.~~ done
2. CostAgent slides + demo script + fallback recording.
3. ~~Ops Gate Agent design doc.~~ done — `agents/ops-gate/ops-gate-design.html`
4. ~~Minimal Ops Gate implementation.~~ done — DataPlatformSuite
5. Design docs for the four base agents (implementations exist; the *design*
   narrative for each still needs writing).
6. **Evidence bank**: for each theme, real GSK engagements with named adopters and a number.
7. Confirm interview format — how long, who is on the panel, is live demo permitted.
