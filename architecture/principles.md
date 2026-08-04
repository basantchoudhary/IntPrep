# Driving Principles — rationale, sequencing and talk-track

Slide: [principles.html](principles.html). This file holds what the slide deliberately
leaves off — why the grouping is this way, what each principle is defending against,
and how it maps to the assessment themes.

## Why this sequence

**Think → Compose → Execute → Operate & Assure.**

It follows the delivery lifecycle, so it reads as a way of working rather than a list
of slogans. It also lands *Deterministic First* in position 3 — after platform and
composition have earned it, and immediately before the assurance payoff it enables.
Leading with determinism sounds like scepticism about AI; arriving at it in sequence
makes it a design conclusion.

## 1. Think — altitude before build

| Principle | Defends against | Say this |
|---|---|---|
| System thinking | Local optimisation that moves cost or risk somewhere else | "A faster pipeline that triples the ops burden is not a win." |
| Platform thinking | One-off solutions per engagement | "I design for the tenth team, not the first." |
| Data & AI is software | Notebooks in production, untested pipelines, no versioning | "There is no data exemption from engineering discipline." |
| Simplicity as a constraint | Architecture-as-résumé; unnecessary distributed complexity | "Complexity has to be earned." |

## 2. Compose — build once, reuse everywhere

| Principle | Defends against | Say this |
|---|---|---|
| Lego blocks | Monolithic platforms nobody can adopt partially | "Capabilities, small and replaceable." |
| Contracts over coupling | "Reusable" components that need bespoke wiring every time | "Interfaces, not integrations — this is what actually makes reuse real." |
| Golden path | Standards that exist as documents nobody follows | "The compliant path has to be the easy path, or it won't be taken." |
| Self-serve | The platform team as a permanent bottleneck | "If teams have to queue for us, we haven't built a platform." |

## 3. Execute — deterministic first

The differentiating column. Most candidates demo an agent that *decides*.

| Principle | Defends against | Say this |
|---|---|---|
| Deterministic by default | Using a model where a rule is correct, cheaper and auditable | "If the answer is knowable, it should be a rule." |
| Agentic by exception | AI applied for its own sake | "The model earns its place only where judgement is genuinely required." |
| Reproducible & idempotent | Non-repeatable runs — fatal in a regulated estate | "Same input, same result — and I can prove it a year later." |
| Human accountable at the boundary | Automated approval of regulated change | "The agent assembles the evidence. A named person signs." |

## 4. Operate & Assure — trusted in production

| Principle | Defends against | Say this |
|---|---|---|
| Cross-cutting is first class | Security/cost/quality bolted on after go-live | "These are design inputs, not a hardening phase." |
| Design for failure | Architectures that assume the happy path | "I design the failure modes, not just the flow." |
| Fail fast, fail visibly | Defects discovered by users in production | "Break at the gate, never in production." |
| Self-explanatory | Systems only their author can operate | "Readable by the next engineer — and now by agents too." |
| Evidence by construction | A scramble for audit evidence before go-live | "Audit is an output of the system, not a phase before it." |

---

## Mapping to the assessment themes

*Internal only — keep this off the slide.*

| Theme | Principles that carry it |
|---|---|
| **T1** Architecture & integration | System thinking, Platform thinking, Contracts over coupling, Design for failure, Simplicity |
| **T2** Assurance & regulatory | Deterministic by default, Reproducible, Human accountable at the boundary, Evidence by construction, Fail fast |
| **T3** Stakeholders & reuse | Lego blocks, Golden path, Self-serve, Self-explanatory, Platform thinking |

## Changes made to the original list

- **Merged** *Transparent system* + *Self-explanatory to humans and agents* — one principle, two audiences.
- **Sharpened** *Software principles* → *"Data & AI is software"*, a defensible claim rather than a category.
- **Sharpened** *Complex vs simple* → *"Simplicity as a constraint — complexity has to be earned"*.
- **Split** *Fail fast* into build-time (*fail fast, fail visibly*) and runtime (*design for failure*) — T1 names failure modes and resilience explicitly, which the original list did not cover.
- **Added** Contracts over coupling · Human accountable at the boundary · Evidence by construction · Reproducible & idempotent.
