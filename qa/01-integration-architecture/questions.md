# Integration Architecture in the AI Era — interview Q&A

> Framework and full reasoning: [framework.md](framework.md).
> Each answer is written to be *spoken* in 60–90 seconds. Bracketed `[...]` are
> placeholders for real numbers — fill them from the evidence bank before the day.
>
> Default answer shape (from [../../evaluation-criteria.md](../../evaluation-criteria.md)):
> **situation → the design I owned → how it became reusable → who adopted it → the measured delta.**

---

## Q1. "How has AI changed how you approach solution architecture?"

**What they're testing:** whether you have judgement or just vocabulary. The wrong answer
is an enthusiastic list of agent frameworks.

**Answer**

> "Less than people expect, and the part that changed is very specific.
>
> Integration has always been the same question: how do independent systems interact
> safely across a boundary neither of them controls. That's unchanged. What changed is
> the cast. We used to have software systems integrating with software systems, with
> humans at the edges. Now there are three classes of participant — software systems,
> agentic systems and human systems — and every pair of them is an integration surface
> that needs a contract, a failure mode and an owner.
>
> So my architecture is the pre-AI architecture, extended. Same integration styles — a
> tool call is RPC, agent-to-agent delegation is messaging, there's no new fifth style.
> Same loose coupling, same contract-first discipline, same blast-radius thinking. Plus
> one new endpoint type that is non-deterministic, and therefore has to be *contained* by
> deterministic structure rather than trusted.
>
> If I had to name what genuinely broke, it's six assumptions: the caller is no longer
> deterministic; consumers are no longer known at design time; contracts can no longer be
> purely syntactic; data is no longer inert; behaviour now changes without a code diff;
> and a passing test no longer proves anything on its own. Everything I've designed in
> the last [N] months is a response to those six."

**Follow-up drills**

- *"Isn't that a bit conservative?"* → "It's what lets me put an agent into a GxP-relevant
  path at all. The conservative bit is the containment; the ambition is in what we're then
  allowed to automate."
- *"Give me one of the six that bit you."* → Have a real story ready — ideally the
  model-version-bump-with-no-code-diff one, because it's the one nobody anticipates.

---

## Q2. "What's your view on MCP, A2A and skills? Are they the future of integration?"

**What they're testing:** whether you can place new technology in a lineage rather than
being impressed by it.

**Answer**

> "They're important, and they're less novel than the noise suggests — which is exactly
> why I'm comfortable adopting them.
>
> MCP is the JDBC moment for tools. Before JDBC every application wrote a bespoke adapter
> per database; MCP does the same collapse for model-to-tool integration — one protocol
> instead of N×M adapters, with discovery and semantic description travelling with the
> contract. A2A is a service registry plus asynchronous messaging, applied to agents.
> Skills are a library or a runbook, packaged so it can be loaded on demand.
>
> But I'd stress where the JDBC analogy breaks, because it's the important part. JDBC
> exposed one resource type with one semantics, and its caller was deterministic code a
> reviewer had read. MCP exposes arbitrary side effects to a probabilistic caller. So it
> standardises transport, discovery and packaging — it does not give you authorisation,
> validation, idempotency, budget control, dead-lettering or audit. Those are still mine
> to design, and they're still the Enterprise Integration Patterns catalogue.
>
> So: yes, adopt them, because N×M bespoke adapters is a dead end and standardisation is
> genuinely valuable. But the protocol is the easy half. The validating adapter behind it
> is where the engineering is."

**Follow-up drills**

- *"Would you standardise on MCP across the estate?"* → Yes for tool exposure, with a
  house adapter template so every MCP server inherits authz, validation, budget and audit
  by construction rather than per team. That template is the reusable asset.
- *"What if the standard changes?"* → It's a transport binding behind our own capability
  contract. We'd re-bind, not re-architect. That's the point of the boundary.

---

## Q3. "Everyone's building agents. How do you decide what should be an agent and what shouldn't?"

**What they're testing:** the differentiating answer. This is the one to be crispest on.

**Answer**

> "I use a three-part test, and all three have to be true before an LLM is allowed in.
>
> One — is the input space genuinely unbounded? If the inputs can be enumerated or
> schematised, it's a rule. Two — does it need judgement rather than lookup? If the
> mapping from input to output can be written as a rule, a query or a conventional model,
> write that. Three — is a wrong answer survivable, because something deterministic or a
> human checks it before it has effect?
>
> Fail any one and it's code. Fail the third specifically and the model must never be the
> final authority, no matter how good it gets.
>
> In practice this produces a consistent shape: observe, propose, validate, execute,
> record. Observe is deterministic — telemetry, logs, catalog, code, IaC. Propose is the
> LLM, interpreting the unfamiliar. Validate, execute and record are deterministic again —
> schema and policy checks, blast-radius and cost limits, human gate by action class,
> idempotent execution, immutable evidence. Four of five stages are deterministic. The
> model gets the one stage where judgement is real and has no authority in any other.
>
> The anti-pattern I actively push back on is agent-washing — wrapping a deterministic
> workflow in an LLM so it can be called agentic. The tell is that the prompt enumerates
> the rules. If you can write the prompt as an exhaustive list of rules, you've just
> written the code — ship the code."

**Follow-up drills**

- *"Sounds like you're under-using AI."* → "It's using AI where it wins and code where
  code wins. It's a cost, latency and assurance conclusion, not scepticism — and it's
  precisely why I can put an agent in a regulated path at all."
- *"Where did you apply the test and change your mind?"* → Have one example each way: one
  proposal you killed down to a rule, and one where the test said *yes* and you built it.
  The second matters as much as the first — it proves you're not just the person who says no.

---

## Q4. "Walk me through the riskiest edge in an agentic architecture."

**What they're testing:** whether you can prioritise risk rather than list it.

**Answer**

> "Agent to system of record. It's the only edge where a probabilistic caller touches
> something that matters, and it's where I spend the design budget.
>
> Three rules. First, the agent never gets raw access — no direct SQL, no admin API, no
> shell on the box. Shared-database integration was already an anti-pattern between two
> deterministic services; it doesn't become acceptable because the caller is now an LLM.
> It gets a capability instead: a narrow, purposeful, validated tool.
>
> Second, every tool has to be independently safe. Pre-AI I could reason that endpoint B
> is only ever called after endpoint A, because I'd read the calling code. An agent
> composes tools in orders nobody anticipated, so 'safe only when called after X' is not a
> property I'm allowed to rely on any more.
>
> Third, blast radius equals the union of what every tool in that agent's scope can
> reach — not what it's likely to do, what it *can* do. So I scope tools and credentials,
> not intentions. Instructions and prompts are guidance; the credential and the tool list
> are the control. 'We told the agent not to' has never been an acceptable answer to a
> security architect and it isn't one here."

**Follow-up drills**

- *"How do you handle an agent that needs broad access?"* → It doesn't get one agent with
  broad access; it gets several narrowly-scoped agents and a deterministic orchestrator
  holding the composition. Bulkhead by scope: one agent, one purpose, one credential, one
  budget.
- *"What about read-only agents?"* → Lower risk, not no risk — exfiltration and injection
  still apply. But read-only is where I start every capability, and a lot of them stay
  there and are still valuable.

---

## Q5. "How do you secure this? What's new in the threat model?"

**What they're testing:** the security-partner conversation — T3 names security explicitly.

**Answer**

> "Three things are genuinely new, and each maps to something we already know how to do.
>
> Prompt injection is the big one, and the honest framing is that it's this generation's
> SQL injection: untrusted data crossing into the control path. Same shape, same fix
> direction — separate control from data, treat retrieved content as tainted, and never
> let it reach the instruction path unmediated. But the difference matters: with SQLi we
> got parameterised queries, a complete structural fix. There is no equivalent complete
> fix here yet. So my primary defence isn't detection, it's capability scoping —
> assume the agent can be hijacked and make sure a hijacked agent still can't do much.
> That's a design stance, not a filter.
>
> Second, identity. The agent is a principal. It needs its own non-human identity,
> short-lived credentials, and delegated on-behalf-of authority that can never exceed the
> requesting human's own scope. Otherwise you've built a privilege-escalation service.
>
> Third, the audit question changes. Pre-AI I logged what happened. Now I have to log
> *why* — inputs, model version, prompt version, tool calls, outputs. The decision trace,
> not just the result. Without that you cannot explain a decision six months later, and in
> a regulated estate that's disqualifying."

**Follow-up drills**

- *"Can you stop prompt injection?"* → "No, and I'd be wary of anyone who says they can.
  I can bound the consequence. That's the design goal — containment, not perfect
  detection."
- *"Multi-agent makes this worse?"* → Yes. Provenance and confidence travel with the
  message, validation at each hop, delegation depth is capped, and delegated authority is
  strictly a subset of the delegator's.

---

## Q6. "How do agents work with a large legacy estate — huge codebases, unclear dependencies?"

**What they're testing:** her stated concern about discovery, and whether she has a real
answer rather than "bigger context window".

**Answer**

> "The agent has exactly the problem a new engineer has: it can't see upstream and
> downstream consequence. And the answer is the same answer, which I find reassuring
> rather than disappointing — lineage, contracts and a dependency graph as *queryable
> assets*, not as a diagram in a slide deck. If the blast radius of a change is
> computable, both the human and the agent get it right. If it isn't, neither will. That's
> not an AI problem, it's an estate-legibility problem that AI has finally put a price on.
>
> So the work is: machine-readable contracts, a catalog with lineage, decision records so
> the agent doesn't 'fix' a deliberate constraint, small modules with explicit
> dependencies, tests as the specification, and repo-level convention files that give a
> code-writing agent its behavioural contract.
>
> The reframe I'd offer is that none of that is agent-specific. Every one of those assets
> serves a human joining the team just as well. I don't write documentation for agents —
> documentation drifts and nobody trusts it. I make the system self-describing, in CI,
> where it can't drift. Both audiences get served by the same artefact, and the business
> case for hygiene that was always right but always deferred is suddenly easy to make.
>
> That's essentially what our reverse-engineering base agent does — recover an accurate
> model of a handed-over estate — and it's the first thing the Ops Gate runs."

**Follow-up drills**

- *"Reverse-engineering is exactly where the model will hallucinate."* → "Which is why its
  output is a *proposal* checked against ground truth — the deployed resources, the actual
  IaC, the observed lineage. It accelerates discovery; it doesn't get to assert facts."
- *"Won't the agent just make the codebase worse?"* → Tests as the safety net, sandbox
  validation before apply, and change size capped. Same controls we'd want on a
  contractor's first PR.

---

## Q7. "How do you test and assure something non-deterministic?"

**What they're testing:** T2. Also the most common place candidates fall apart.

**Answer**

> "By making sure the non-deterministic part is small and the boundary around it is
> deterministic — then testing each differently.
>
> The deterministic four-fifths gets conventional testing: unit, integration, contract,
> data tests. Unchanged, and it's most of the system by design.
>
> The model step gets *evaluation* rather than testing. A golden set of representative
> cases, scored against a threshold, run in CI as a gate — so behaviour regression blocks
> a release the same way a failing test does. The critical detail is that the model
> version and the prompt version are pinned and treated as versioned dependencies. A model
> upgrade is a dependency upgrade: it goes through the eval gate and it canaries. The
> failure mode nobody plans for is behaviour changing with a zero-line code diff, and
> pinning plus evals is the only thing that catches it.
>
> And then the structural argument, which is the one that actually satisfies assurance:
> the model's output is a proposal, and everything with an effect is validated
> deterministically afterwards. So the assurance question stops being 'is the model
> always right', which is unanswerable, and becomes 'is the validation complete' — which
> is a normal engineering question with a normal engineering answer.
>
> On our Ops Gate that's [N] checks, all deterministic, producing a signed evidence bundle
> by construction rather than assembled before an audit."

**Follow-up drills**

- *"What's your eval pass bar?"* → Set per capability by consequence; irreversible-effect
  paths carry a higher bar and a human gate regardless of score. Have your real number.
- *"Who owns the golden set?"* → The domain owner, not the engineer — otherwise it tests
  what we built rather than what's needed.

---

## Q8. "How do you handle human-in-the-loop without it becoming a bottleneck?"

**What they're testing:** whether HITL is architecture to you or UX.

**Answer**

> "By making the gate a lookup instead of a judgement call, so it can be sized.
>
> Every tool carries a side-effect class in its contract: read, write-reversible,
> write-irreversible, privileged. Read is automatic. Reversible writes are automatic
> within budget and logged. Irreversible writes — delete, send, pay, publish — always take
> a human. Privileged actions take a human plus a second reviewer. Once that's in the
> contract, the gate isn't argued per project; it's derived. That's what makes it both
> auditable and reusable.
>
> And gate density becomes an architectural decision rather than an accident. A human gate
> that produces four hundred approvals a day isn't a control, it's a rubber stamp with
> extra steps — so I size it deliberately, and I treat approval dwell time as a health
> metric. If approvals are being cleared in two seconds, the control has failed even
> though the audit trail looks perfect.
>
> The other half is treating the human as a typed participant. They receive a structured
> object — proposed action, evidence, blast radius, reversibility, cost, expiry — not a
> chat message. There's a timeout, and the default on timeout is deny, never proceed.
> There's an escalation path and a safe-park state, because 'the approver is on leave'
> is a failure mode you design for, not discover.
>
> Underneath all of it: the agent assembles the evidence, a named person signs. That
> boundary isn't a limitation of today's models that we relax later. It's where
> accountability lives, and accountability doesn't move because the model improves."

**Follow-up drills**

- *"What if the business wants full automation?"* → Reversible actions can go fully
  automatic and often should. The line is reversibility and regulatory relevance, not
  ambition — and I'll move the line as evidence accumulates, per action class, with data.

---

## Q9. "Agent-to-agent — is that real, or is one agent enough?"

**What they're testing:** hype resistance, and whether you understand distributed systems.

**Answer**

> "It's real where it's a genuine separation of concerns, and theatre where it's one
> problem chopped up for a diagram.
>
> The justification for splitting agents is the same as for splitting services: different
> scope, different data sensitivity, different credentials, different owner, different
> release cadence. Our Ops Gate composes four base agents for exactly that reason — the
> sandbox creator, the test scaffolder, the data-expectation agent and the
> reverse-engineering agent all have different blast radii and different ownership. That
> split is bulkheading, and it's the reuse story too: the composite capabilities were
> assembled from those base agents rather than built from scratch.
>
> What I'd resist is multi-agent as a performance strategy — five agents chatting to
> reach an answer one could have reached. That's cost, latency and an unbounded failure
> surface bought for a diagram.
>
> And when I do split, I treat it as untrusted inter-service messaging, because that's
> what it is. Authenticate the caller, carry provenance and confidence in the message,
> validate at every hop rather than only the first, cap delegation depth, and make
> delegated authority strictly a subset of the delegator's. Otherwise a wrong output from
> agent one becomes an established fact for agents two, three and four — which is the
> cascading failure mode, and it's the same class of problem as trusting an internal
> service just because it's inside the perimeter. We learned that lesson once already."

---

## Q10. "What reusable patterns have you established?" *(the T3 question)*

**What they're testing:** the grade bar directly — reuse, adoption, authority.

**Answer** *(fill the brackets — this answer lives or dies on the numbers)*

> "Four, and the important thing about them is that they were extracted from delivery, not
> designed in the abstract.
>
> First, the participant model — the three classes and the nine integration edges, with a
> named contract and failure mode per edge. It started as how I explained one design and
> became the reference framing for [N] engagements.
>
> Second, the deterministic/agentic test — the three-part rule for whether something is
> allowed to be an LLM. That's now the intake question on new agentic proposals, and it's
> converted [N] of them into deterministic solutions that shipped faster and passed
> assurance.
>
> Third, side-effect classification driving human gating. That's the one security and
> compliance picked up, because it turns the approval decision into a lookup they can
> audit rather than a judgement they have to trust.
>
> Fourth, the agent control plane itself — registry, delegated identity, policy, budgets,
> eval harness, audit. It's identical across all four of our agents, which is the actual
> proof: the second capability took [X%] of the effort of the first, because everything
> except the judgement step already existed.
>
> And the Ops Gate is shipped as a skill specifically so another team can adopt it without
> me — [N] have. That's the test I hold myself to. A pattern that needs its author present
> isn't reusable, it's a consultancy engagement."

**Follow-up drills**

- *"How do you get people to adopt patterns?"* → Golden path, not documentation. Make the
  compliant path the easy path — a template that already carries authz, validation, budget
  and audit. Adoption follows convenience, never mandate.
- *"Who's disagreed with you on one of these?"* → Have a real one, and how it improved the
  pattern. Recognised authority means people push back and you incorporate it.

---

## Q11. "What are the failure modes you design for?" *(T1 names this explicitly)*

**Answer**

> "I'd separate the ones we already knew from the ones that are genuinely new.
>
> The classic ones don't go away — partial failure, timeout, retry storms, poison
> messages, cascading dependency failure. Bulkheads, circuit breakers, dead-letter
> channels, idempotency. Unchanged.
>
> New ones, with the control against each. Hallucinated or malformed calls — strict schema
> plus semantic precondition checks in the adapter, and reject rather than coerce, because
> coercing a wrong call into a valid one is how you get a confidently wrong action.
> Prompt injection — trust tiering and capability scoping, so a hijacked agent is
> contained. Runaway loops and cost — hard caps on depth, calls, tokens and wall-clock,
> with budget as a contract term rather than an afterthought. Over-broad authority — one
> agent, one scope, one credential. Silent model drift — pinned versions and an eval gate.
> Non-reproducibility — the full decision trace logged, not just the outcome. And human
> gate fatigue, which is a real failure mode people don't list: an overloaded approver is
> a failed control even when the audit trail looks immaculate.
>
> The one I'd single out is idempotency. It was good practice before; it's mandatory now,
> because agents retry and re-plan, and a non-idempotent tool turns a retry into a
> duplicate payment. Every irreversible action needs an idempotency key and a defined
> compensating action — or it needs a human, and that's a legitimate answer."

---

## Q12. "How do you avoid lock-in to a model vendor?"

**Answer**

> "By treating the model as what it is — a versioned third-party dependency with a vendor,
> a price, a drift profile and an exit cost. Same as a database.
>
> Practically: the model sits behind our own capability contract, so the application
> depends on 'summarise this incident with this schema', not on a specific provider's SDK.
> Prompts and evals are versioned assets in the repo, not embedded in application code.
> The eval suite is the portability test — it's how I know in an afternoon whether an
> alternative model clears the bar, rather than it being a research project.
>
> I'd also be straight about the trade-off, because pretending it's free isn't credible.
> Full abstraction costs you provider-specific capability, and some of that capability is
> genuinely valuable. So I abstract at the capability boundary, not at every feature — and
> where I do use something provider-specific I make it a deliberate, recorded decision with
> a known replacement cost, rather than something that leaked in.
>
> The deeper point is that most of my system doesn't call a model at all. Four of five
> stages are deterministic. Lock-in exposure is proportional to how much of the
> architecture you handed to the model — which is one more reason the deterministic-first
> position is an engineering position rather than a philosophical one."

---

## Q13. "A senior stakeholder wants everything agentic. How do you handle that?"

**What they're testing:** T3 — stakeholder engagement, and whether you can disagree well.

**Answer**

> "I don't argue the philosophy, I run an intake — because a procedure is reusable and an
> opinion isn't, and people accept a process they can see applied consistently to everyone.
>
> Four questions. Can this be a rule? What's the side-effect class of everything it can
> touch? What's the blast radius if it's completely wrong, and how is that bounded? How is
> it evaluated, and what does regression look like?
>
> That conversation usually ends somewhere better than either starting position: a smaller
> agent, a bigger deterministic surface, and a capability that actually passes assurance
> and ships. It also reframes the disagreement — I'm not the person blocking AI, I'm the
> person getting it into production, and the constraint is what makes that possible.
>
> The one thing I'd never do is win the argument quietly by building it my way. If a
> stakeholder wants the agentic version and accepts the assurance position, that's a
> legitimate decision and it's theirs to make. My job is to make sure the trade-off is
> explicit and recorded, and to design the containment properly either way. Where I hold
> firm is a very narrow line: a model doesn't give the final approval on regulated change.
> That isn't a preference, and it isn't mine to trade."

---

## Q14. "If you had to give one piece of architectural advice to a team starting on agents?"

**Answer**

> "Design the boundary before you design the agent.
>
> Teams start with the prompt and the framework, and the first serious question — what can
> this thing actually reach, and what happens when it's wrong — arrives during a security
> review, when it's expensive. If you start from the other end, from side-effect class and
> blast radius and what the human signs, the agent almost designs itself and it's a lot
> smaller than you expected.
>
> The second thing, which I'd say to anyone I'm mentoring: the interesting engineering in
> an agentic system is almost never in the model. It's in the contracts, the validation,
> the evidence and the failure modes. That's not a consolation prize — it's the same craft
> that made integration work in the first place, which is why the pre-AI catalogue is
> still the most valuable thing on my desk."

---

## Quick-reference — one-liners to have ready

| Prompt | Line |
|---|---|
| Framing | "Same integration questions, one new participant — and that participant is non-deterministic." |
| On MCP | "The JDBC moment for tools. Standardisation, not safety. The protocol is the easy half." |
| On new styles | "A tool call is RPC. A2A is messaging. There's no fifth integration style." |
| On raw access | "Shared-database integration was already an anti-pattern. A probabilistic caller doesn't fix it." |
| On determinism | "If the answer is knowable, it should be a rule." |
| On agent-washing | "If the prompt enumerates the rules, you've written the code. Ship the code." |
| On blast radius | "Blast radius is what the agent *can* reach, not what it's likely to do. I scope tools, not intentions." |
| On prompt controls | "Prompts are guidance. Credentials and tool scope are the control." |
| On injection | "It's this generation's SQL injection — but there's no parameterised query yet. So I bound the consequence, not the input." |
| On accountability | "The agent assembles the evidence. A named person signs." |
| On legibility | "I don't write documentation for agents. I make the system self-describing — in CI, where it can't drift." |
| On testing | "'Is the model always right' is unanswerable. 'Is the validation complete' is a normal engineering question." |
| On HITL | "Four hundred approvals a day isn't a control, it's a rubber stamp with extra steps." |
| On reuse | "A pattern that needs its author present isn't reusable — it's a consultancy engagement." |
