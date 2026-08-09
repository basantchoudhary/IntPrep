# Integration Architecture in the AI Era — the thinking framework

> Topic 1 of the Q&A set. This file is the *structure of the argument*.
> The spoken answers are in [questions.md](questions.md).
>
> Anchors: [../../architecture/principles.md](../../architecture/principles.md) ·
> [../../architecture/north-star-platform.md](../../architecture/north-star-platform.md) ·
> [../../agents/ops-gate/ops-gate-design.html](../../agents/ops-gate/ops-gate-design.html)

---

## 0. The thesis — say this first, in one breath

> "Enterprise integration didn't restart in 2023. The question was always *how do
> independent systems interact safely across a boundary they don't control* — and that
> question is unchanged. What changed is the **cast**. We used to integrate software
> with software, with humans at the edges. Now there are three classes of participant —
> **software systems, agentic systems, and human systems** — and every pair of them is
> an integration surface that needs a contract, a failure mode and an owner. So my
> architecture is the pre-AI architecture, extended: same integration styles, same
> coupling discipline, same blast-radius thinking — plus one new endpoint type that is
> **non-deterministic**, and therefore has to be **contained by deterministic
> structure**, not trusted."

Then the payoff line:

> "Most people are designing agents that decide. I design systems where the agent
> **proposes** and the deterministic control plane **disposes** — and a named human
> signs anything that matters."

---

## 1. Continuity — what carries over completely untouched

This is the credibility move. Lead with what *doesn't* change; it signals you understand
integration rather than just the new vocabulary.

| Pre-AI discipline | Still true, verbatim | Why it matters *more* now |
|---|---|---|
| **Loose coupling** | Participants must not know each other's internals | An agent's call sequence is unpredictable — coupling you'd have survived deterministically now bites randomly |
| **Contract-first** | The interface is the product; the implementation is private | Agents consume contracts *literally*; a vague contract becomes a wrong call |
| **Simplicity as a constraint** | Complexity has to be earned | Agents amplify accidental complexity — they will find and traverse every path you left open |
| **Bounded blast radius** | Bulkheads, quotas, circuit breakers, least privilege | Now the primary safety control, not a resilience nicety |
| **Idempotency & reproducibility** | Same input, same result; safe to retry | Was good practice; is now **mandatory** because agents retry and re-plan |
| **Golden path over documentation** | Make the compliant path the easy path | The golden path is now what the *agent discovers*, so it must be executable, not written down |
| **Reusable assets / Lego blocks** | Build once, compose everywhere | Tools and skills are the newest Lego blocks; the composition argument is identical |
| **Observability & audit** | You cannot operate what you cannot see | Non-determinism makes tracing the *decision*, not just the call, mandatory |
| **The four integration styles** | File transfer · shared database · RPC · messaging (Hohpe & Woolf, 2003) | Agents introduce no fifth style. A tool call is RPC. A2A delegation is messaging. That's it. |

**Line to use:** *"MCP is not a new integration style. It's RPC with discovery and a
semantic contract. Recognising that is what stops you re-solving problems the industry
solved in 2003."*

And the sharpest continuity point:

> "Shared-database integration was already an anti-pattern — two systems coupled through
> a schema neither owns. Handing an agent raw SQL against a system of record is that same
> anti-pattern, except the caller is now probabilistic. If it was wrong for a
> deterministic service, it is not suddenly right for an LLM."

---

## 2. Discontinuity — the assumptions AI actually breaks

Be precise here. Six, not "everything".

| Assumption that held pre-AI | What breaks | Architectural response |
|---|---|---|
| **The caller is deterministic** — same input, same call sequence | An agent may call different tools, in a different order, for identical input | Design the *tool*, not the caller, to be safe: validate, constrain, make idempotent, cap |
| **Consumers are known at design time** | An agent composes tools you never anticipated pairing | Every tool must be independently safe — no "safe only when called after X" |
| **The contract is syntactic** (schema, types) | Schema-valid, semantically wrong calls become the dominant defect | Contracts gain a **semantic layer**: purpose, preconditions, side-effect class, cost, when *not* to use |
| **Data is inert** | Retrieved content can carry instructions — prompt injection | Trust tiers on content; never let retrieved data reach the control path unmediated |
| **Behaviour changes only when code changes** | A model version bump silently changes behaviour with zero diff | Pin model versions; treat the model as a versioned third-party dependency with an eval gate in CI |
| **Testing = deterministic assertions** | Pass/fail on one run proves nothing | Add **evaluation** alongside testing: golden sets, distributional thresholds, regression on behaviour |

**Line to use:** *"AI didn't invalidate my architecture. It invalidated six assumptions
inside it, and I can name all six."* — that sentence alone separates a senior architect
from someone reciting agent vocabulary.

---

## 3. The participant model — the core artefact

Three classes of participant. Nine integration edges. This is the diagram to draw on the
whiteboard if they hand you a pen.

- **S — Software systems.** Deterministic, contract-bound. APIs, pipelines, databases,
  ERP, systems of record.
- **A — Agentic systems.** Probabilistic, goal-bound. Plan, choose, call, reflect.
- **H — Human systems.** Accountable, approval-bound. Approvers, operators, reviewers,
  change boards.

| From ↓ / To → | **Software (S)** | **Agent (A)** | **Human (H)** |
|---|---|---|---|
| **Software (S)** | Classic EIP — events, messaging, API contracts, CDC, ETL | Bounded task envelope: async job + callback, budget, timeout | Alert, ticket, dashboard, report |
| **Agent (A)** | **Tool call via a validating adapter (MCP)** — the highest-risk edge | **A2A delegation** — task contract, agent card, provenance chain | Escalation & approval — typed request + evidence bundle + timeout policy |
| **Human (H)** | UI, API, CLI — unchanged | Intent + constraint: prompts, policy, guardrails, golden path definitions | Existing org process — RACI, change advisory board |

### The three rules that fall out of the grid

1. **A→S is where the engineering goes.** It is the only edge where a probabilistic
   caller touches a system of record. Everything else is a variation on problems we
   already know. Spend your design budget here.
2. **A→H must be typed, not conversational.** An approval is a structured object —
   proposed action, evidence, blast radius, reversibility, expiry — not a chat message.
   If a human approval can't be replayed and audited a year later, it isn't a control.
3. **A→A is the newest and least mature.** Treat cross-agent delegation as untrusted
   inter-service messaging: authenticate, carry provenance, cap depth, and never let
   delegated authority exceed the delegator's own scope.

**Line to use:** *"Pre-AI we asked how systems talk to each other. We now also have to
ask how systems **explain themselves** — because one of the participants discovers the
integration at runtime instead of being wired to it at build time."*

---

## 4. The new substrate — MCP, A2A, Skills — mapped to what they replace

The interview trap is treating these as novel. The senior answer maps each one to its
pre-AI ancestor and then names precisely what is genuinely new.

| New | What it actually is | Pre-AI ancestor | Genuinely new part |
|---|---|---|---|
| **Tools / function calling** | A typed remote procedure the model may invoke | RPC endpoint, service operation | The *caller decides* whether and when to call |
| **MCP** (Model Context Protocol) | Standard client/server protocol for exposing tools, resources and prompts to a model | ODBC/JDBC — a driver standard that killed N×M bespoke adapters | Discovery + semantic description travel *with* the contract |
| **A2A** (Agent-to-Agent) | Inter-agent task delegation with capability discovery ("agent cards") | Service registry + async messaging / choreography | The delegate can renegotiate the task rather than just execute it |
| **Skills** | A packaged, portable capability — instructions, scripts, resources — loaded on demand | Library / plugin / runbook | Progressive disclosure: loaded only when relevant, so capability scales past context limits |
| **Context & memory** | Working state for a run; retained state across runs | Session state, cache, conversation store | It is *also* an attack surface and a correctness surface |

### The MCP = JDBC analogy — and where it breaks

Use it, then immediately show its limits. That's what makes it a senior answer.

> "MCP is the JDBC moment for tools — one protocol instead of N×M bespoke adapters.
> But the analogy breaks in two places, and both matter. JDBC exposed one resource type
> with one semantics; MCP exposes *arbitrary side effects*. And a JDBC caller was
> deterministic code that a reviewer had read. So MCP gives you standardisation without
> giving you safety — the adapter behind it still has to do authorisation, validation,
> and blast-radius control. The protocol is the easy half."

### What none of them give you

MCP, A2A and Skills solve **transport, discovery and packaging**. They do not solve:

- authorisation and delegated identity
- input validation and output verification
- idempotency and compensation
- rate limiting, budget caps and loop breaking
- audit, lineage and evidence
- routing, translation, dead-lettering — the actual Enterprise Integration Patterns

> "Which is exactly why the EIP catalogue is still on my desk. The new protocols moved
> the plumbing; they didn't move the plumbing's failure modes."

---

## 5. The deterministic / agentic boundary — the decision rule

This is her signature argument and it already runs through
[principles.md](../../architecture/principles.md) and the Ops Gate design. State it as a
**test**, not an opinion — a test is reusable, an opinion isn't.

### The test: use an LLM only if all three are true

1. **Unbounded input space** — the inputs cannot be enumerated or schematised in advance
   (an unfamiliar codebase, a free-text incident, a heterogeneous estate).
2. **Judgement, not lookup** — the mapping from input to output cannot be expressed as a
   rule, table, query or model you could have written.
3. **A wrong answer is survivable** — because it is checked downstream by something
   deterministic, or by a human, before it has effect.

Fail any one → it should be code. **Fail #3 in particular → it must never be the final
authority**, however good the model is.

### The reference flow — the shape of every agentic capability she has built

```
   observe            propose             validate            execute            record
 deterministic  →   agentic (LLM)   →  deterministic   →  deterministic   →  deterministic
  collect facts     interpret,          schema, policy,     idempotent,        immutable
  from telemetry,   diagnose,           safety, blast       reversible,        evidence,
  logs, catalog,    recommend,          radius, cost;       scoped creds       lineage,
  code, IaC         draft artefacts     human gate by                          signed bundle
                                        action class
```

Four of the five stages are deterministic. **The LLM occupies one stage — the one where
judgement is genuinely required — and it has no authority in any of the others.**

### The pushback to have ready

*"Isn't that under-using AI?"*

> "It's using AI where it wins and code where code wins. The model's advantage is
> interpretation of the unfamiliar; that's one step in five. In a regulated estate the
> other four have to be repeatable and auditable — and a rule is cheaper, faster and
> provable. This isn't scepticism about AI, it's a cost, latency and assurance
> conclusion. And it's *why* I can put an agent into a GxP-relevant path at all: the
> non-determinism is boxed in on both sides."

### The anti-pattern to name

**Agent-washing** — wrapping a deterministic workflow in an LLM so it can be called
agentic. Symptoms: the agent's every decision is a `case` statement in disguise; the
prompt enumerates the rules; removing the LLM changes nothing except cost, latency and
auditability.

> "If you can write the prompt as an exhaustive list of rules, you have just written the
> code. Ship the code."

---

## 6. Contracts — from syntactic to semantic

Contracts over coupling was already the principle. AI adds a layer rather than replacing
one.

| Layer | Contents | Consumed by |
|---|---|---|
| **Syntactic** | Schema, types, required fields, versioning, error codes | Code — unchanged |
| **Semantic** | Purpose, preconditions, postconditions, *when not to use*, worked examples, side-effect class, cost/latency envelope, data classification | **Agents**, and new engineers |
| **Operational** | SLO, rate limits, idempotency key, retry & compensation semantics, owner | Both, plus the control plane |

The critical addition is **side-effect class**, because it drives authorisation and human
gating automatically:

| Class | Meaning | Default gate |
|---|---|---|
| `read` | No state change | Auto |
| `write-reversible` | Changes state, cleanly undoable | Auto within budget; logged |
| `write-irreversible` | Cannot be undone (delete, send, pay, publish) | **Human approval, always** |
| `privileged` | Touches credentials, policy, IAM, production config | Human approval + second reviewer |

> "Once side-effect class is in the contract, the gate stops being a judgement call at
> runtime and becomes a lookup. That is what makes the control auditable — and reusable
> across every agent instead of re-argued per project."

This is a strong T2 artefact: it's a *reusable classification*, exactly what the grade bar
asks for.

---

## 7. Designing systems to be machine-legible

Her point about large codebases, upstream/downstream dependencies and auto-discovery.

**The reframe that lands:** making a system legible to an agent is the *same work* as
making it legible to the next engineer. AI didn't create a new requirement; it finally
put a price on hygiene that was always correct and always deferred.

| Legibility asset | Serves humans | Serves agents |
|---|---|---|
| Machine-readable contracts (OpenAPI, schemas, data contracts) | Onboarding | Discovery + safe invocation |
| Catalog, glossary, lineage | "Where does this number come from?" | Grounding; blast-radius calculation before a change |
| ADRs (decision records) | Why it is this way | Prevents the agent "fixing" a deliberate constraint |
| Small modules, explicit dependencies | Reviewability | Fits in context; bounded change surface |
| Executable golden path (templates, scaffolds, CI) | The easy path | The *discoverable* path |
| Repo-level agent instructions (`CLAUDE.md` / `AGENTS.md`) | Conventions in one place | Behavioural contract for code-writing agents |
| Tests as specification | Confidence | The safety net that makes autonomous change acceptable |

**The dependency point, stated crisply:**

> "An agent asked to change one service in a large estate has the same problem a new
> engineer has — it can't see the upstream and downstream consequence. The answer isn't
> a bigger context window, it's the same answer as for humans: lineage, contracts and
> a dependency graph as *queryable assets*, not as a diagram in a slide deck. If the
> blast radius of a change is computable, both the human and the agent get it right.
> If it isn't, neither will."

**Line to use:** *"I don't write documentation for agents. I make the system
self-describing — and both audiences are served by the same artefact. Documentation
drifts; a contract in CI cannot."*

---

## 8. The human plane — as an architectural component

Common failure: teams treat human-in-the-loop as UX, bolted on at the end. Treat it as a
participant with an interface, an SLA and a failure mode.

| Design question | Bad answer | Architectural answer |
|---|---|---|
| What does the human receive? | A chat message | A typed approval object: action, evidence, blast radius, reversibility, cost, expiry |
| How long do they have? | Unbounded | A timeout with a **defined default** — and the default is *deny*, never proceed |
| What if they're unavailable? | The run hangs | Escalation path, fallback approver, safe-park state |
| Can they be overwhelmed? | "They'll cope" | Rate-limit approvals; batch by class; if the queue is unmanageable the gate is theatre |
| What is recorded? | A log line | An immutable evidence bundle: who, what, on what basis, when |
| Where is accountability? | "The agent approved it" | A named person signs. Always. |

> "A human gate that arrives as 400 approval requests a day isn't a control — it's a
> rubber stamp with extra steps. Gate density is an architectural decision, and I size it
> by side-effect class so the human sees the twenty that matter, not the four hundred
> that don't."

And the accountability line, which is the T2 centrepiece:

> "The agent assembles the evidence. A named person signs. That boundary is not a
> limitation of today's models that we'll relax later — it's where accountability lives,
> and accountability doesn't move just because the model gets better."

---

## 9. Agentic design patterns — the catalogue, mapped

We had GoF (1994) for objects and EIP (2003) for integration. The agentic pattern
language is forming now. Show it as a *continuation of that lineage* — and be honest that
it is less mature.

### Composition patterns (how work is arranged)

| Pattern | What it does | Pre-AI ancestor |
|---|---|---|
| **Prompt chaining** | Decompose into fixed sequential steps | Pipes and filters |
| **Routing** | Classify, then dispatch to a specialist | Content-based router (EIP) |
| **Parallelisation** | Fan out independent subtasks; aggregate | Scatter-gather (EIP) |
| **Orchestrator–worker** | A planner decomposes and delegates dynamically | Process manager (EIP) / coordinator |
| **Evaluator–optimiser** | Generate → critique → revise until a bar is met | Feedback control loop |
| **Reflection** | The agent critiques its own output before returning | Code review / assertion |
| **Tool use** | Reach outside the model for facts and effects | Adapter, service invocation |
| **Multi-agent collaboration** | Specialist agents with distinct scopes | Microservices, separation of concerns |

### Containment patterns (how it is kept safe) — the ones that matter in a regulated estate

| Pattern | What it does | Pre-AI ancestor |
|---|---|---|
| **Validating adapter** | Every agent-initiated call passes through deterministic validation | Message translator + validating endpoint (EIP) |
| **Plan-then-confirm** | Agent emits a plan; the plan is approved; execution is mechanical | Change request / dry run / `terraform plan` |
| **Sandbox execution** | Prove the action in an isolated replica before touching production | Staging, canary, blue/green |
| **Bulkhead by scope** | One agent, one purpose, one credential, one budget | Bulkhead, least privilege |
| **Budget & loop breaker** | Hard caps on tokens, calls, depth, wall-clock | Circuit breaker, timeout, quota |
| **Compensating action / saga** | Every irreversible step has a defined undo, or it needs a human | Saga, compensating transaction |
| **Dead-letter for agent runs** | Failed or abandoned runs are parked, visible, triaged — never silently retried | Dead letter channel (EIP) |
| **Provenance & trust tiering** | Content is labelled by trust; untrusted content never reaches the control path | Taint tracking, input sanitisation |
| **Evidence by construction** | The audit artefact is emitted by the run, not reconstructed afterwards | Immutable audit log |

**Line to use:** *"Two thirds of that second table is EIP and resilience patterns with new
names. That's not a criticism — it's the reason I can put agents into a regulated estate
with a straight face. We are not starting from zero on safety; we're applying a mature
catalogue to a new kind of caller."*

---

## 10. Failure modes and blast radius — the T1 requirement

T1 names *"designing for resilience, failure modes"* explicitly. Have the new failure
modes enumerated with a control against each — that's the difference between a capability
map and a failure analysis.

| Failure mode | What it looks like | Control |
|---|---|---|
| **Hallucinated / malformed call** | Schema-valid nonsense, or a call to a tool that doesn't fit | Strict schema + semantic precondition checks in the adapter; reject, don't coerce |
| **Prompt injection via data** | Retrieved document instructs the agent | Trust tiers; data never enters the control path; capability scoping so a hijacked agent still can't do much |
| **Runaway loop / cost** | Agent re-plans indefinitely | Depth, call, token and wall-clock caps; budget as a first-class contract term |
| **Over-broad authority** | One agent holds credentials for everything | One agent, one scope, one credential; delegated on-behalf-of identity, short-lived |
| **Cascading multi-agent failure** | A wrong output is consumed as fact by three downstream agents | Provenance and confidence carried in the message; validation at each hop, not just the first |
| **Silent model drift** | Version bump changes behaviour, no code diff | Pin versions; eval suite as a CI gate; canary on model change like any dependency upgrade |
| **Non-reproducibility** | Cannot explain a decision made six months ago | Log inputs, model version, prompt version, tool calls, outputs — the *decision trace*, not just the result |
| **Context poisoning / staleness** | Agent reasons over stale or contaminated state | Bound context lifetime; re-ground from source of record on state-changing paths |
| **Human gate fatigue** | Approvers rubber-stamp | Gate by side-effect class; measure approval dwell time as a health metric |

**The blast radius formula, stated plainly:**

> "Blast radius = the union of what every tool in that agent's scope can reach. Not what
> it's *likely* to do — what it *can* do. So I scope tools, not intentions. Prompts and
> instructions are guidance; the credential and the tool list are the control."

That single sentence is the security-architect answer. It is also why "we told the agent
not to" is never an acceptable control.

---

## 11. The reference architecture — layered view

Draw this if asked "so what does it look like?" It is the North Star, with the agentic
layer made explicit.

```
┌──────────────────────────────────────────────────────────────────────────┐
│  EXPERIENCE           apps · copilots · chat · IDE · dashboards          │
├──────────────────────────────────────────────────────────────────────────┤
│  HUMAN PLANE          approvals · escalation · evidence review · sign-off│  ← typed, SLA'd
├──────────────────────────────────────────────────────────────────────────┤
│  AGENT CONTROL PLANE  registry · identity & delegation · policy & gates  │  ← DETERMINISTIC
│  (the reusable asset) budgets · eval harness · audit & evidence · trace  │     the reuse surface
├──────────────────────────────────────────────────────────────────────────┤
│  AGENT RUNTIME        planning · memory/context · reflection · skills    │  ← the only
│                       composite agents built from base agents            │     probabilistic layer
├──────────────────────────────────────────────────────────────────────────┤
│  INTEGRATION FABRIC   MCP servers · A2A · APIs · events · validating     │  ← EIP lives here
│                       adapters · translators · dead-letter · routers     │
├──────────────────────────────────────────────────────────────────────────┤
│  SYSTEMS & DATA       systems of record · lakehouse · catalog · lineage  │
│                       IaC · pipelines · observability                    │
└──────────────────────────────────────────────────────────────────────────┘
```

**Three things to say about this diagram:**

1. **The control plane is deterministic and sits *above* the runtime.** The agent doesn't
   police itself. Self-policing is not a control — it's the same actor.
2. **The control plane is the reusable asset.** It's identical for the Ops Gate agent, the
   Self-Healing agent and the Spark Modernization agent. That is the T3 pattern-reuse
   evidence: the second capability cost a fraction of the first because everything except
   the judgement step already existed.
3. **The integration fabric is where classical EIP does its work** — which is why the
   architecture is an extension of the pre-AI one rather than a replacement for it.

Maps directly onto the Horizontal Platform Services row of
[north-star-platform.md](../../architecture/north-star-platform.md).

---

## 12. Governance — how to say no to "make everything an agent"

Have a *procedure*, not an attitude. Senior engineers are recognised by having a
repeatable way to decide.

**The four-question intake, applied to any proposed agentic capability:**

1. **Can this be a rule?** If yes, it's a rule. (Kills most proposals — say so cheerfully.)
2. **What is the side-effect class of everything it can touch?** Determines the gate,
   before anyone writes a prompt.
3. **What is the blast radius if it is completely wrong, and how is that bounded?** If the
   answer needs the model to be right, the design is wrong.
4. **How is it evaluated, and what does regression look like?** No eval, no production.

> "Nine times out of ten this ends with a smaller agent and a bigger deterministic
> surface — and a capability that actually passes assurance. The tenth is genuinely
> valuable and gets built properly. I'd rather be the person who says 'that's a rule'
> early than the person explaining an incident later."

---

## 13. How this topic maps to the grade bar

| Theme | What in this topic carries it |
|---|---|
| **T1** Architecture & integration | The participant model and 3×3 grid as a reference design; the failure-mode table; the layered reference architecture; explicit resilience thinking |
| **T2** Assurance & regulatory | The deterministic/agentic test; side-effect classification driving gates; evidence by construction; human accountable at the boundary |
| **T3** Stakeholders & reuse | The control plane as the reused asset across four agents; contracts as the reuse mechanism; the four-question intake as an adoptable standard; the pattern catalogue as a teaching artefact |

**Evidence still to fill in** (open item 6 in [../../plan.md](../../plan.md)) — every answer
below is stronger with a real number:

- `[N]` engagements that adopted the participant model / integration standard
- `[X%]` reduction in integration rework or defects after the contract standard landed
- `[N]` agents built on the shared control plane, and the build-time delta between the
  first and the second
- `[N]` proposals converted from agentic to deterministic at intake, and what that saved
