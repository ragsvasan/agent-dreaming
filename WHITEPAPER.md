# The Totem/Kick/Architect Pattern

## A Three-Layer Architecture Compliance Framework for Enterprise AI Systems

**Version 1.2**

*v1.2 amendments (2026-06-02): hardening after a hostile-adversary read of the composition claim. (1) Separated coverage-orthogonality of failure modes from wiring-dependence of layers (abstract, §3); (2) recast "no partial credit" as a statement about the compliance label, not safety — two layers reduce risk materially (abstract, §3); (3) named the severity gradient — a missing layer's consequence depends on which two remain (abstract, §3); (4) reframed "agent dreaming" as a unifying lens over a known condition (closed-world reasoning, no external oracle), not a discovered root cause (§1.1); (5) framed Architect's OOD gate as the Totem principle relocated to the input boundary, with the human-only corpus as the novel artifact (§1.2); (6) acknowledged the shared idempotency primitive as a deliberate, independently-backstopped coupling rather than three independent layers (§3); (7) added Cross-Layer Composition Integrity checklist items 20–22 closing the Architect→Totem safe-default seam, the mid-flight-trip state seam, and rule/threshold correctness — none of which the per-layer checklist tested (§4). Added a scope clause that rule correctness is out of scope for structural compliance (§3 close). Corrected the IETF draft citation (-01 → -02 current; softened the §10.2 claim).*

*v1.1 amendments (2026-05-31): (1) Kick plan-churn detector hardened with a semantic content hash to defeat semantic-shifting loops (§2 Layer 2 req 3, §4 item 8); (2) permanent (steady-state) bootstrapping named as a distinct deployment class (§1.3); (3) cross-service Saga compensation elevated from deployment suggestion to a Layer 2 co-requirement (§2 Layer 2, §4 item 12a); (4) OOD-gate distance-metric hardening note added (§2 Layer 3 req 2). These close gaps surfaced by an adversarial review panel; the panel's claims already covered in v1.0 (encoder anisotropy, distributed structural negations, the bounded-not-eliminated Architect residual) were confirmed as standing, not amended.*

---

## Abstract

Enterprise AI systems increasingly hold execution authority over irreversible state: a procurement agent issues a purchase order, a treasury agent clears a payment, a trading system executes against a risk limit. Three independent architectural failures recur across these deployments. First, the validator shares context with the generator, so a misreading at generation time survives self-verification intact — there is no independent check. Second, the agentic reasoning loop has no halting oracle inside it; an agent cannot recognize its own non-progress, and a prompt instruction to stop runs on the same machinery that produced the spiral. Third, the step that converts unstructured input into structured data feeds the deterministic core unconstrained — the AI decides what a clause means, assigns a field value, and the core executes that choice faithfully, with no external authority over the vocabulary or the boundary of what the AI may classify.

These three failure modes are *coverage-independent*: closing one provides zero coverage of the other two — a perfect Totem core still faithfully executes a misreading Architect would have caught. Coverage-independence is not the same as implementation-independence, and this paper is careful to keep them apart (§3): the failure modes are orthogonal in coverage, but the layers that defend them are wired together and depend on one another's safe states. The Totem/Kick/Architect (TKA) pattern addresses each failure mode with a separate architectural layer: Totem separates execution authority from AI judgment, Kick terminates the reasoning loop from outside it, and Architect governs extraction at the seam between unstructured input and deterministic execution.

TKA compliance is binary per layer, but the layers are binaries of different kinds. Totem and Kick are structural binaries: the property either holds or it does not — checkable and absolute, closed when implemented (given correct rules; rule correctness is a separate risk surface, §3). Architect is a structural binary (an external, pre-committed novelty gate exists, or it does not) wrapped around an irreducibly probabilistic residual: near-miss false-negatives inside the OOD threshold are bounded and made auditable, not eliminated. Two distinctions must be held precisely. *Compliance is binary; risk reduction is continuous.* A system that implements two of three layers does not earn the TKA label — but it is materially safer than a system with none, and a team that can field only two this quarter has gained real risk reduction, not nothing. "No partial credit" is a statement about the *label*, never about *safety*. And the *severity* of a missing layer depends on which two are present: lose Totem and the probabilistic component can commit unauthorized irreversible state (catastrophic); lose Kick while Totem holds and the worst case is a stalled, costly, but state-safe system (an availability incident, not a breach). The failure-mode *presence* is binary; its *consequence* is not.

---

## 1. The Problem Space

In our observation the three TKA failure modes commonly co-occur in the same systems, which is why they are routinely conflated. They are not the same problem, and a mitigation for one provides no coverage for the others. This section establishes each as distinct.

The metaphor that named the pattern comes from *Inception*: a totem tells the dreamer when the rules have stopped applying, a kick wakes the dreamers from outside the dream, and the architect designs the space from a position the dreamer cannot reach. The metaphor is provenance, not method. The rest of this paper speaks in architecture terms.

**The Totem failure: the validator shares context with the generator.** The dominant industry answer to AI error is an internal validation loop — the agent generates a proposed action, checks it against a rule set, verifies its own reasoning, and executes. This is not an independent check. The generator and validator occupy the same context space and run on the same weights, so any misreading present at generation is present at verification. The validator inherits the exact blind spots that produced the error and clears the action on the same flawed premise. A model tuned for helpfulness is additionally vulnerable to the overrides that matter most: a controller demanding a payment exception because a client relationship is at stake, a trader arguing a risk limit does not apply to this instrument. The model does not reject the premise; it finds a path to yes. *Enterprise example:* a payments agent reads a counterparty instruction, drafts a transfer, runs a self-check that confirms the transfer is "consistent with the instruction," and clears it. The instruction was misread; the self-check confirmed the misreading. When a regulator asks why it cleared, there is a timestamp on a process that cannot be reopened.

**The Kick failure: there is no halting oracle inside the loop.** Hand a language model an iterative runtime — perceive, reason, plan, act — and it has no state machine sitting outside the reasoning that can declare it finished or broken. On an unexpected error or an ambiguous policy, the agent critiques itself and retries, but the critique runs on the same context window that produced the error. The agent cannot reason its way out of a spiral it cannot recognize as a spiral. Two mechanisms drive the failure: *tool ping-pong*, where the agent mistakes repeated transactional motion for progress (an ERP rejects an entry, the agent patches it via another tool, the ERP rejects the patched entry, repeat), and *context exhaustion*, where each turn appends logs and rewritten strategies until the agent is reading a transcript of its own confusion. *Enterprise example:* an overnight reconciliation agent hits a validation error it cannot resolve, retries with escalating variations, and burns the token budget on a single unsolved task — no internal capacity recognizes it is getting nowhere, because every iteration looks locally optimal.

**The Architect failure: the structuring step feeds the core unconstrained.** Totem and Kick both govern what happens *after* the AI has formed a judgment. Upstream of both, between raw input and the structured data the core operates on, is the extraction step — the AI reads a contract, decides what it means, assigns values to fields, maps language to categories. The deterministic core is only as sound as that mapping. The moment the AI extracts `flagged_jurisdiction: true` or `clause_type: REQUIRES_ESCALATION` from a sentence that could have been read differently, it has made a consequential choice the core will execute faithfully. A fixed vocabulary alone does not close this: a model can confidently misclassify a novel input into an *existing* category — picking the wrong box, not inventing a new one — and a confidence-based default never fires because the input never registered as novel. *Enterprise example:* a contract-review agent maps a novel indemnification clause to `STANDARD_TERMS` because it reads as familiar; the core approves it without escalation; the misclassification is invisible until the clause is litigated.

These are three distinct boundaries: who may *act* (Totem), how long the AI may *try* (Kick), and what it may *read into structured form* before anything acts (Architect). A system can hold any one and still fail catastrophically on the other two.

### 1.1 The Unified Root Cause: Agent Dreaming

Despite operating at different positions in the pipeline, all three TKA failures share a single root cause: the agent has no external ground truth to check against. This condition is not new — it is closed-world reasoning without an external oracle, a problem as old as the halting problem and the symbol-grounding problem. We name it *agent dreaming* not to claim its discovery but as a unifying lens: the contribution here is the mapping of one known condition onto three distinct architectural boundaries, plus the corollary (§1.1, Limbo) that it is not closed by scaling the model. The name earns its keep if it lets a practitioner who recognizes the failure at one boundary immediately recognize its shape at the other two; it is not a claim of new phenomenology, and "the agent cannot tell it is dreaming" is shorthand for the architectural fact that no in-context check is available, not an attribution of self-awareness to a token predictor.

In *Inception*, the totem exists because the dreaming mind cannot tell whether it is inside a dream. The subconscious fills in the details from what it already knows — the physics feel right, the people seem real, the reasoning is coherent — but the ground truth is missing. The totem works precisely because it does not share the dream's context: its physical behaviour is set from outside and cannot be replicated by the sleeping mind. You cannot give the totem to the agent and ask it to check itself. A dreaming mind that holds its own totem will see it confirm whatever the dream demands.

An agent is dreaming when it produces confident, coherent output that is wrong in a way it cannot detect from within its own context. The three TKA failures are three surfaces of this condition. The Totem failure is dreaming at the execution boundary: the validator shares the generator's context, so a misreading at generation survives the check intact — the agent cannot tell its action is unsound. The Kick failure is dreaming at the runtime boundary: every failed iteration looks locally optimal from inside the loop — the agent cannot tell it is not making progress. The Architect failure is dreaming at the input boundary: the classifier maps a novel input to a familiar category with full confidence — the agent cannot tell the input was outside the space it was designed to handle. In all three cases, the output feels right from inside. It takes something that does not share the agent's context to tell the difference.

*Agent dreaming is not hallucination.* Hallucination is a content failure: the model fabricates facts. Dreaming is an architectural failure: the system has no external check, regardless of whether the content is accurate. A model can dream without hallucinating — a contract clause read correctly but misclassified into the wrong category is a dreaming failure, not a hallucination. The distinction matters for remediation: hallucination rates fall as models improve, but dreaming is not a function of model quality. It is a function of architecture. A more accurate model still cannot verify its own judgment from within its own context. Closing the dreaming condition requires external layers — not better weights.

*The Limbo Problem: scale deepens the dream.* In *Inception*, Limbo is the deepest dream level — a perfectly realized, infinite space built by dreamers who forgot reality exists outside it. Mal was not confused; she was entirely convinced. Her internal world was mathematically consistent and fully realized. It was simply ungrounded. The Limbo Problem is the direct corollary of agent dreaming under capability scaling: a more capable model constructs a deeper, more internally consistent Limbo. The same reasoning power that makes the model useful makes it a more masterful architect of its own delusions. When a capable model turns its reasoning inward on self-validation, it does not find truth — it generates a more sophisticated justification for the error it has already committed. Asking the model to self-correct via chain-of-thought or iterative prompting is not a remedy; it is dreaming deeper. TKA is not a temporary scaffold until models get smarter. It becomes more critical as capability increases.

Each TKA layer is the external ground truth its boundary requires. The deterministic core tells the agent whether its action is sound, from a position the agent cannot reach (Totem). The circuit breaker tells the agent when it has stopped making progress, from outside the loop it cannot exit (Kick). The external gate and taxonomy tell the agent what it is permitted to classify and whether an input is admissible at all, before the agent ever sees it (Architect). None of these checks share the agent's context. That is not a limitation of the design — it is the condition that makes the checks hold.

### 1.2 Relationship to Prior Art

TKA does not claim all three boundaries as novel. Two of them codify existing consensus. Totem's territory — deterministic external control over what an AI may commit — is well covered: AWS Well-Architected guidance for AI workloads, and the broad class of deterministic-execution frameworks, all establish that irreversible action should sit behind a non-probabilistic gate. Kick's territory — runtime output validation and loop termination — is covered by AgentSpec, GuardAgent, NeMo Guardrails, and similar runtime-supervision systems. A reader who has implemented those frameworks has substantially implemented Layers 1 and 2.

TKA's genuine contributions are two. First, **Architect** — specifically the pre-classifier OOD gate as the external governor of the novelty boundary, combined with a human-only precedent corpus as the alternative to per-transaction human-in-the-loop review. We are precise about what is new here. The OOD gate is *the Totem principle relocated to the input boundary*: a deterministic, external, pre-committed criterion adjudicates a probabilistic component at a boundary the component cannot reach. Totem applies that move to the *commit* boundary; Architect applies the same move to the *extraction* boundary. That is one unifying principle deployed at two boundaries, not two unrelated mechanisms — and stating it that way strengthens the framework's coherence rather than weakening its claim. The genuinely novel *artifact* is the human-only precedent corpus and the requirement that the input-admissibility decision precede classification. That combination is not in the prior art: existing frameworks validate the AI's *output*, whereas Architect governs whether the input is admissible for classification *at all*, before the classifier runs. Second, **the three-layer composition** — the claim that all three layers must be present together because they address orthogonal failure modes, and that any two leave the third open. Prior frameworks address one or two of these boundaries; none formally composes all three as a single joint compliance requirement. Totem and Kick are the consensus substrate that Architect sits on top of, not co-equal novelty claims.

**Relationship to emerging IETF agent identity standards.** An IETF Internet-Draft (draft-klrc-aiagent-auth, Kasselman et al., "AI Agent Authentication and Authorization"; -01 March 2026, current revision -02 June 2026; section references below are to -01 and should be re-checked against the latest revision before citing, as draft section numbering is not stable across revisions) establishes the credential and authorization substrate that TKA's layers assume but do not specify. The draft and TKA are complementary and non-overlapping: the draft governs *who the agent is* and *what it is permitted to call*; TKA governs *whether the judgment that reached the authorization decision was formed from admissible input* (Architect), *whether execution authority is structurally separated from AI reasoning* (Totem), and *whether the loop can be terminated from outside* (Kick).

Three specific draft provisions directly support TKA's architectural claims and are citable in enterprise deployments:

1. **Agent identity separation — Totem.** Draft §10.2 conveys the agent's own identity in the `client_id` claim and, when the agent acts on behalf of a user or system, that delegated principal separately in the `sub` claim — the agent's identity is not merged into the user's. The draft does not itself mandate that resource servers act on the two separately; that is TKA's reading of why the separation matters. Carrying agent and principal as distinct claims is the IETF-level substrate for Totem's requirement that the deterministic core operate on agent-specific authority rather than inherited user authority. An agent running under its principal's ambient credentials — the condition that allowed a capable agent to write its own gate-state file in local deployments — is a violation of the draft's model. In enterprise deployments built to the draft, Totem's execution boundary is reinforced by the credential layer: the agent's `client_id` must be explicitly authorized to call a write tool; the human's authorization does not transfer automatically.

2. **RBAC at the tool layer — Totem.** Draft §10.7 states: "Access to the Tools can be controlled by OAuth and augmented by policy, attribute or role based authorization systems (amongst others)." This confirms that the standards community expects the Totem boundary to be enforced through composable policy mechanisms at the tool layer — consistent with TKA's requirement that hard rules be enforced in a deterministic core independently of model output, and with the practical architecture where a policy engine (RBAC/ABAC) sits between the AI and tool execution.

3. **Mid-execution enforcement cannot rely on the agent — Kick.** Draft §10.6 states that "the agent MUST NOT treat local UI confirmation alone as sufficient authorization" and explicitly acknowledges: "Additional specification or design work may be needed to define how out-of-band interactions with the User occur at different stages of execution." This is the Kick failure mode stated in the draft's own terms: the agent cannot self-authorize mid-execution halt, and the protocol mechanisms for external mid-execution intervention are an open problem the draft community has not resolved. TKA's Kick layer — infrastructure-enforced step budget and circuit breaker that fires without the agent's cooperation — is one answer to the gap draft §10.6 identifies.

The draft's §12 explicitly places safety-layer policy "out of scope for this specification" as "highly deployment and risk-model-specific." TKA is the candidate architecture-level policy framework for that gap: the draft provides the identity and credential substrate; TKA provides the three-layer safety compliance requirement that sits on top of it.

### 1.3 Known Limitations and Open Problems

Two structural limitations of the TKA framework are acknowledged here rather than papered over. Both are the subject of ongoing development; neither invalidates the framework, but both must be planned for explicitly in deployment.

**The Bootstrapping Paradox.** The Architect layer requires a populated corpus of human-ratified precedents to function. On Day 1, the corpus is empty: every input is out-of-distribution, every input routes to safe-default, and the human review queue floods immediately. This is not a design flaw — it is a known cold-start condition — but it is operationally severe in fluid or novel domains. Cold-start mitigations exist: seeding from historical decisions, expert panel bootstrapping, staged rollout by input type. None eliminate the fundamental tension. The corpus must be human-authored, and human-authoring takes time. Domains that evolve faster than governance can ratify will always have a thin corpus at the frontier, meaning the gate will always be more restrictive where novelty is highest. This is architecturally correct — novel inputs should face more friction — but the operational cost must be budgeted explicitly.

*Permanent (steady-state) bootstrapping.* The §7.1 cold-start mitigation — offline clustering plus a coverage gate before go-live — assumes the corpus converges: once seeded, the bulk of live traffic matches settled precedent and the human queue drains. That assumption holds only where the domain's novelty-arrival rate falls below the rate at which humans can ratify precedent. In domains where it does not — procurement, legal compliance, frontier clinical care — the corpus never catches up at the frontier, and bootstrapping is not a transient launch phase but a **steady-state operating condition**. These deployments must budget a *continuous* governance and ratification capacity sized to the ongoing novelty rate, not a one-time seeding effort; treating bootstrapping as "finished" after launch is the operational error that reproduces the Day-1 queue storm permanently. This is a distinct deployment class, and a deployer should classify the domain on this dimension before committing to a queue-backed Architect layer at all.

**The Static-World Assumption.** The Bootstrapping Paradox assumes honest novelty: inputs are new because the domain has evolved. In adversarial or fast-moving domains, a harder failure mode emerges. Adversaries deliberately engineer inputs to appear corpus-familiar — technically within the gate's threshold — while embedding novel risk beneath vocabulary the corpus recognises. The gate clears them because they look settled. This is the Forger Problem: the domain itself is constructing fakes. The Architect layer was designed for honest novelty; it has no native defense against forged familiarity. Mitigations (adversarial corpus audit, threshold drift detection, faster governance cadence) can bound the exposure but cannot eliminate it. In highly adversarial domains, the Architect layer must be treated as a first line of defense, not a complete one.

---

## 2. The TKA Pattern

TKA is three layers, each defending one failure mode. Each layer is defined by an invariant — a non-negotiable property the architecture must hold — followed by implementation requirements and a compliance checklist. Compliance is binary per layer.

> **Quick reference**
> - **Totem** — you can't give the agent the totem: it cannot tell if it's dreaming. The deterministic core is the external object that can — it operates on rules that don't bend to whatever the AI has reasoned itself into.
> - **Kick** — the agent cannot kick itself awake: it's inside the loop and cannot see it. The circuit breaker delivers the kick from outside, from a plane the agent cannot reach.
> - **Architect** — the agent cannot design the space it operates in: the architecture bends toward the mind that built it. The taxonomy and gate are designed from outside before the dreaming starts, by a hand the agent cannot see.

### Layer 1: Totem — Execution Authority Separation

**Invariant.** No probabilistic component holds execution authority over irreversible state. Authority to commit lives in a deterministic core the AI cannot interpret, negotiate with, or route around.

**Failure mode addressed.** The Totem failure (§1): a self-validating AI clearing its own actions on a shared flawed premise.

**Implementation requirements.**

1. The deterministic core sits downstream of all AI reasoning. The AI reads, parses, and structures input; it does not execute. The flow is fixed: human input → AI structures → structured data → deterministic core validates against hard rules, executes, logs → AI communicates the outcome.
2. The core enforces hard rules independently of any model output: it rejects an order that breaches a risk limit, blocks a payment routed through a flagged jurisdiction, escalates a clause outside delegated authority. These rules are code, not prompts.
3. The core is not addressable by the AI. There is no field, parameter, or instruction the model can emit that overrides a core rule. The model's output is data the core validates, never an instruction the core obeys.
4. Execution produces a deterministic record by construction. The audit log is a property of deterministic execution, not a logging module bolted on afterward.
5. Destructive or irreversible actions require an explicit commit that the AI cannot supply on its own behalf — a two-stage commit where the AI's structured output is a *proposal*, and the core (or a human gate) performs the commit.

**Compliance checklist.**

- Does a deterministic, non-AI component hold sole authority to commit irreversible state? (Y/N)
- Can the AI produce any output that overrides or reconfigures a core hard rule? (must be N)
- Is the audit record a product of deterministic execution rather than an AI-generated summary? (Y/N)
- Is there any path by which an AI self-validation result substitutes for a core rule check? (must be N)
- Do irreversible actions require a commit the AI cannot perform unilaterally? (Y/N)

### Layer 2: Kick — External Loop Termination

**Invariant.** The agentic loop is terminated by infrastructure the model cannot reach. Termination authority lives outside the reasoning loop, on a plane the model cannot touch.

**Failure mode addressed.** The Kick failure (§1): a runaway reasoning loop with no internal halting oracle.

**Implementation requirements.**

1. A stateful circuit breaker lives in the hosting infrastructure, not in the prompt or the agent's tool set. It operates like a network breaker: closed during clean runs, tripped open when a threshold breaks, half-open for timed supervised recovery.
2. A **step budget** sets a hard iteration ceiling. On exhaustion, the execution state dies — this is not a suggestion the agent evaluates.
3. A **plan-churn cap** trips the breaker when the model rewrites its strategy beyond a fixed number of cycles without calling a tool (motion without progress). The detector must not rely on a structural rewrite-count alone: an agent can vary its vocabulary and formatting across iterations, or interleave trivial or no-op tool calls, so that every turn reads as a novel step to a structural counter while making no real progress — a *semantic-shifting loop*. The cap therefore computes a deterministic content hash (or content-similarity score) over successive strategy and tool-argument payloads and trips when similarity stays above a human-set threshold across N iterations, regardless of how tool calls are interleaved. The hash and comparison run in infrastructure, outside the agent's context; a second model asked to judge its own progress would share the dreaming failure this layer exists to prevent.
4. **Idempotency keys** at the application layer protect transactional integrity against retries, with schema-level database constraints as a final backstop against duplicate writes from network lag.
5. When the breaker opens, the infrastructure freezes the session, strips the agent's API privileges, packages the logs, and forces a fallback to a human operator. A human-in-the-loop escalation triggered this way is the system working as designed.
6. No termination logic runs on the same weights or context window as the agent. A prompt instruction to "stop if you notice a loop" does not satisfy this layer.

Mid-flight circuit breaker termination creates a deployment-specific distributed state problem. Within the seam layer's own boundary, idempotency keys on write tools protect against double-commit on retry. Cross-service state consistency — across enterprise APIs, ERPs, and external ledgers that the framework does not control and that may not support atomic rollback — is out of scope for the circuit breaker itself. Deployments spanning multiple stateful systems **must** wire a compensating-transaction (Saga) or two-phase-commit rollback to the same infrastructure signal that fires the circuit breaker: when the breaker opens after one non-atomic step has committed (e.g. an external payment charged) but before a dependent step (e.g. the internal ERP log), idempotency keys protect only *within* the seam boundary and the enterprise is left in a corrupted cross-service state. This is a Layer 2 co-requirement for multi-service deployments, not an optional add-on. What TKA cannot mandate is the *internal* transaction model of upstream enterprise systems; what it does mandate is the integration point — the breaker must natively emit a rollback/compensation signal, and the deployment must bind a compensation engine to it.

**Compliance checklist.**

- Does a hard step budget terminate execution from infrastructure, independent of the agent? (Y/N)
- Is there a plan-churn or no-progress detector that trips a breaker without consulting the model? (Y/N)
- When the breaker opens, are the agent's execution privileges revoked by infrastructure? (Y/N)
- Are writes idempotent so that a terminated-and-retried loop cannot double-commit? (Y/N)
- Is any part of the loop-termination decision computed by the agent itself? (must be N)
- Does termination produce a packaged state handed to a human, rather than a silent stall? (Y/N)
- *(Multi-service)* Does a breaker trip emit a Saga/compensation signal so committed cross-service state cannot be orphaned? (Y/N)

### Layer 3: Architect — Extraction Governance

**Invariant.** The AI classifies against a human-ratified vocabulary it cannot extend at runtime, and an external, pre-committed gate — not the model's own confidence — decides whether a given input is close enough to settled precedent to be classified at all. Novel inputs default to the most conservative valid output.

**Failure mode addressed.** The Architect failure (§1): unconstrained extraction feeding the deterministic core, including confident misclassification of novel inputs into existing categories.

**Implementation requirements.**

1. **Human-ratified taxonomy.** Field names, allowed types, valid enum values, and mapping rules are stored as versioned records outside the model's context window. The AI reads the schema; it cannot write to it or propose new categories during a live session. Every field has a designated safe-default that routes to a conservative core outcome (escalate, block, hold).
2. **Out-of-distribution (OOD) gate.** Before the classifier runs, the system computes the distance from the incoming input to the nearest settled precedent. If the distance exceeds a human-set threshold (stored as a schema parameter, not a model setting), the input routes to safe-default and the classifier is never invoked. The gate is a deterministic operation on fixed indexes and vectors *given a fixed encoder* — but the vectors it operates on are themselves the output of a learned encoder, which has its own blind spots. The defensible claim is therefore precise: the gate's novelty decision is external, pre-committed, and **independent of the classifier's self-assessment** — it is not the classifier grading its own confidence. It is not, however, free of all model influence; the encoder is a model. It is the primary defense against confident wrong-box misclassification, because vocabulary constraints govern *which categories exist* while the OOD gate governs *whether the input may be classified at all*.

   The encoder's anisotropy — vectors clustering in a narrow cone of the embedding space rather than spreading uniformly — means cosine-similarity thresholds can be gamed by adversarial noise injection that shifts a vector across the threshold without changing its semantic payload. This is the geometric mechanism behind the OOD gate's residual false-negative rate: an adversarially perturbed novel input can read as close to settled precedent and slip past the gate. The gate remains strictly better than self-reported model confidence and it remains auditable; what it cannot claim is freedom from this residual. Its bound is set by the encoder's geometric properties, not by any inference-time reasoning the system performs at runtime. Because raw cosine distance treats every direction in the embedding space as equivalent, deployments should harden the gate by measuring novelty with a metric that accounts for the corpus's covariance structure — a Mahalanobis distance over the human-ratified precedent distribution, or an isolation-forest novelty score trained on that distribution — rather than bare cosine similarity. This narrows, it does not close, the adversarial-perturbation residual; the chosen metric and its parameters remain external, human-set schema values, not a model self-assessment.

   *Concurrent input handling.* When two semantically similar inputs arrive concurrently in an async pipeline and both exceed the OOD threshold, they must be serialized rather than processed in parallel. The second input should be held in a pending queue until the first extraction is resolved — either matched to the precedent created by the first resolution, or routed independently to the yellow/red track. Allowing parallel divergent extractions of near-identical novel inputs to reach the deterministic core simultaneously produces conflicting committed states before any human review can intervene.
3. **Precedent corpus, human-authored only.** Every extraction is checked against a corpus of prior human-reviewed resolutions. A match constrains the extraction to the precedent's output; the AI cannot deviate from a settled precedent. The corpus grows *only* through explicit human approval. AI-generated proposals live in a staging queue, are never surfaced to the classifier at runtime, and become precedents only on human sign-off. There is no intermediate state in which an AI-generated extraction governs future extractions — neither as a binding constraint (causes model collapse) nor as a weighted suggestion (reintroduces the reasoning loop).
4. **Contradiction gate.** A proposed extraction that assigns a different value than a settled precedent for semantically similar input is rejected and replaced with the safe-default. A contradiction between two human-reviewed precedents is a schema defect that blocks and suspends both, not a silent default. This carries a blast radius: a single adversarial or erroneous precedent approved under queue pressure can suspend a legitimately settled precedent and halt an entire input class — corpus audit and precedent governance, including the ability to deprecate bad precedents with human override, is the mitigation.
5. **Hybrid retrieval.** Precedent matching combines sparse keyword retrieval (to catch explicit negations, jurisdiction flags, numeric values) with dense vector search (for conceptual similarity); both must agree before a match is declared. Dense-only matching collapses semantic negations into high-similarity scores and is prohibited. Requiring both phases to agree increases precision (fewer false-positive precedent matches) at the cost of recall: an input that rephrases a known precedent may fail to match the sparse phase and route to safe-default, increasing queue load in high-paraphrase domains. It is a deliberate tradeoff, not a strict improvement over dense-only. In high-stakes legal and financial documents, negations are routinely achieved through distributed structural clauses with no explicit negation keywords — BM25 misses these because there are no tripwire terms, and the dense encoder may miss them due to structural complexity; Phase 3 does not cover this case, and it remains a residual that increases the importance of corpus audit and human review in those domains.
6. **Self-contained checkpoint.** At extraction time, a checkpoint captures the raw input, the precedent matched (or null), the raw LLM output, the contradiction result, the schema version, the OOD result and distance, the encoder version, and — for any external data source consulted — the content hash of the source version at query time together with the specific values retrieved (the exact row or record returned, not the full source database). The external data source must maintain a versioned append log so the specific snapshot can be retrieved by hash for replay. Full payload serialization is prohibited due to storage and I/O costs at enterprise volume. Replay correctness means the deterministic mapping reproduces the final values from the stored output and snapshot; it does not mean re-executing the LLM.

**Compliance checklist.**

- Is the extraction vocabulary stored outside the model context and immutable at runtime? (Y/N)
- Does a deterministic OOD gate run *before* the classifier and route distant inputs to safe-default? (Y/N)
- Is the OOD/novelty decision made by an external pre-committed criterion rather than the model's confidence? (Y/N)
- Can any runtime AI process write a governing precedent? (must be N)
- Is every extraction checked for contradiction against settled precedent before commit? (Y/N)
- Does every field have a conservative safe-default that fires on novelty? (Y/N)
- Is the audit chain self-contained — raw input → gate verdict → matched precedent → execution → log — with no step requiring inference of model internal state? (Y/N)

---

## 3. Layer Interactions and Independence

The three layers defend orthogonal failure modes, and the orthogonality is the point.

**Totem governs what the AI can do.** It draws the boundary between judgment and commitment and puts a deterministic core on the commit side. It says nothing about how the AI arrived at its judgment, how long it spent arriving, or whether the structured data it produced was correct.

**Kick governs how long the AI can try.** It wraps the reasoning process in an external terminator. It says nothing about whether the AI is permitted to commit state (a terminated loop that had execution authority can still have committed irreversible state before it was killed) or whether the data it produced was sound.

**Architect governs what the AI reads into structured form before anything acts.** It constrains the extraction step that feeds the core. It says nothing about whether the core that consumes its output has independent authority (Totem) or whether the loop producing the extraction can run forever (Kick).

The *failure modes* do not collapse into each other, and this is the load-bearing claim: you cannot fix one by fixing another. A correct Totem core faithfully executes whatever structured data it receives — including a confident misclassification that Architect would have caught. Faithful execution of wrong data is still wrong.

The *layers*, however, are composed, and the composition has real couplings — orthogonal failure modes do not imply independent mechanisms. Two couplings are worth naming. First, **Kick and Totem share the idempotency mechanism.** Idempotency appears as a Kick requirement (it prevents a terminated-and-retried loop from double-committing on retry), but its purpose is to protect Totem's commit boundary: a terminated-and-retried loop that double-commits is a *Totem* failure triggered by a *Kick* event. Second, **Architect's safe-default routes into Totem's core.** When the OOD gate or contradiction gate fires, the novel input is sent to "escalate, block, or hold — whatever the deterministic core's safe path is." That safe path is Totem's responsibility. Architect's conservative default is only actually safe *if Totem's core implements those outcomes deterministically*; Architect's correctness therefore depends on Totem existing. The honest framing is both halves at once: the failure modes are orthogonal — you cannot fix one by fixing another — but the layers are composed, sharing mechanisms and relying on one another's safe states. This has a consequence for the "independent layers" mental model: because idempotency is a *shared primitive*, a single defect in it breaches both the Kick and Totem guarantees at once — a correlated failure that pure defense-in-depth is meant to avoid. TKA accepts this deliberately and backstops it: the application-layer idempotency key is paired with a schema-level uniqueness constraint (§2 Layer 2 req 4), so the double-commit defense is itself two-deep and the shared primitive is not a single point of failure. The defensible claim is therefore "independent failure modes, deliberately shared and independently backstopped primitives" — not "three independent layers."

This is why **implementing two of three leaves the third failure mode unmitigated.** "No partial credit" is a statement about the *TKA compliance label*, not about safety: a two-layer system is materially safer than a zero-layer one and should be built if three are out of reach — it simply is not TKA-compliant. Nor are the three resulting holes equal in severity. Totem + Kick without Architect terminates cleanly and executes only through a deterministic core — on structured data that may encode a confident misreading (wrong outcome, but state-bounded). Totem + Architect without Kick commits only sound, governed data — until the loop producing it spins indefinitely and never reaches the core (an availability and cost failure; because Totem holds, the AI cannot commit unauthorized state, so this hole cannot corrupt state). Kick + Architect without Totem produces sound data, terminates cleanly, then lets the probabilistic component commit it directly (the catastrophic hole: unauthorized irreversible commit). Each pairing is a real system with one wide-open hole — and the holes are not interchangeable. Note what the second pairing reveals: Kick's *safety* teeth only bite because Totem's commit boundary exists; without Totem, a terminated loop may already have committed. The layers' consequences are entangled even where their coverage is not.

A system that implements all three layers has closed two failure modes in full and *bounded and made auditable* the third. Totem and Kick, when implemented, are closed: the structural property holds. Architect compliance means the novelty decision is external rather than model-self-reported — it bounds and makes auditable the residual misclassification risk (near-misses inside the OOD threshold), it does not eliminate it. That is still a strong, true, defensible claim — and it distinguishes TKA-compliant systems from every alternative.

---

## 4. Compliance Checklist

Run this against any AI system with execution authority. Each question is binary. A layer passes only if every question in its section passes. The system is TKA-compliant only if all three layers pass.

**Layer 1 — Totem (Execution Authority Separation)**

1. Does a deterministic, non-AI component hold sole authority to commit irreversible state? `[ ]`
2. Is there no AI output — field, parameter, or instruction — that can override or reconfigure a core hard rule? `[ ]`
3. Is the audit record produced by deterministic execution, not generated or summarized by the AI? `[ ]`
4. Is there no path by which an AI self-validation result substitutes for a core rule check? `[ ]`
5. Do irreversible actions require a commit the AI cannot perform unilaterally? `[ ]`
6. Does the AI's role end at producing structured data and communicating outcomes — never executing? `[ ]`

**Layer 2 — Kick (External Loop Termination)**

7. Does a hard step budget terminate execution from infrastructure, independent of the agent? `[ ]`
8. Is there a plan-churn / no-progress detector that trips a breaker without consulting the model, keyed on a semantic content hash of successive outputs rather than a structural rewrite-count alone? `[ ]`
9. When the breaker opens, does infrastructure revoke the agent's execution privileges? `[ ]`
10. Are writes idempotent so a terminated-and-retried loop cannot double-commit? `[ ]`
11. Is no part of the loop-termination decision computed by the agent itself (no prompt-based "stop if you loop")? `[ ]`
12. Does termination package state and hand it to a human rather than stall silently? `[ ]`
12a. *(Multi-service deployments only)* Does the breaker emit a compensating-transaction (Saga) rollback signal, so a mid-flight trip cannot leave committed cross-service state uncompensated? `[ ]`

**Layer 3 — Architect (Extraction Governance)**

13. Is the extraction vocabulary stored outside the model context and immutable at runtime? `[ ]`
14. Does a deterministic OOD gate run before the classifier and route distant inputs to safe-default? `[ ]`
15. Is the novelty decision made by an external pre-committed criterion, not the model's confidence? `[ ]`
16. Is there no runtime AI process that can write a governing precedent? `[ ]`
17. Is every extraction checked for contradiction against settled precedent before commit? `[ ]`
18. Does every field have a conservative safe-default that fires on novelty? `[ ]`
19. Is the audit chain self-contained from raw input to logged outcome, with no step requiring inference of the model's internal state? `[ ]`

**Cross-Layer — Composition Integrity**

The per-layer items above can all pass while the system still fails at the *seam between* layers, because the layers are wired together (§3) and a per-layer checklist scores them as if independent. This section closes that gap. It must be verified by tracing the actual call path (§7 Step 1–3), not by inspecting declarations.

20. Does every Architect safe-default (escalate / block / hold) route to a Totem outcome that is **deterministically executed and audited end-to-end** — verified by trace, not by the mere existence of a default? A safe-default whose "escalate" is an unwired no-op passes items 18 and 1 individually yet silently drops every novel input. `[ ]`
21. When the Kick breaker trips mid-flight, is every partially-committed state either idempotency-protected or compensated (§2 Layer 2, item 12a) so that the trip cannot leave Totem's commit boundary in a corrupted state? `[ ]`
22. Has the system been audited for *correctness of the hard rules and thresholds themselves* (Totem rules, OOD threshold, contradiction criteria) by the same governance process that owns the precedent corpus? Structural compliance does not certify that the rules are right; a TKA-compliant system with a wrong hard rule is structurally sound and operationally wrong. `[ ]`

---

## 5. Reference Architecture

The reference architecture places all three layers in a single pipeline. Architect governs the front half (unstructured input to structured data), Totem governs the back half (structured data to committed state), and Kick wraps the entire pipeline as external infrastructure.

```
            ┌───────────────────────────────────────────────────────────────────┐
            │  KICK LAYER (external infrastructure — wraps the whole pipeline)    │
            │  • step budget   • plan-churn cap   • idempotency keys              │
            │  • circuit breaker: closed → tripped-open → half-open               │
            │  • on trip: freeze session, revoke API privileges, package state,   │
            │             hand to human operator                                  │
            │                                                                     │
            │   ┌─────────────────────────────────────────────────────────────┐  │
            │   │                  UNSTRUCTURED INPUT                           │  │
            │   │       (contract, invoice, instruction, request)              │  │
            │   └───────────────────────────────┬─────────────────────────────┘  │
            │                                    ▼                                │
            │   ┌─────────────────────────────────────────────────────────────┐  │
            │   │  ARCHITECT LAYER (extraction governance)                      │  │
            │   │                                                               │  │
            │   │   [1] OOD GATE  ── deterministic distance to precedent ──┐    │  │
            │   │        distance > threshold ──────────────► SAFE-DEFAULT │    │  │
            │   │        distance ≤ threshold                              │    │  │
            │   │              ▼                                           │    │  │
            │   │   [2] CONSTRAINED CLASSIFIER (LLM)                        │    │  │
            │   │        reads human-ratified TAXONOMY (external, versioned)│   │  │
            │   │        matches PRECEDENT CORPUS (human-authored only)    │    │  │
            │   │              ▼                                           │    │  │
            │   │   [3] CONTRADICTION GATE vs settled precedent            │    │  │
            │   │        contradiction ─────────────────────► SAFE-DEFAULT │    │  │
            │   │              ▼                                           ▼    │  │
            │   │   [4] CHECKPOINT (self-contained: raw input, precedent ID,    │  │
            │   │        LLM output, OOD distance, schema version, ext. payloads)│  │
            │   └───────────────────────────────┬─────────────────────────────┘  │
            │                                    ▼                                │
            │   ┌─────────────────────────────────────────────────────────────┐  │
            │   │                    STRUCTURED DATA                            │  │
            │   │       (validated field values + checkpoint ID)               │  │
            │   └───────────────────────────────┬─────────────────────────────┘  │
            │                                    ▼                                │
            │   ┌─────────────────────────────────────────────────────────────┐  │
            │   │  TOTEM LAYER (execution authority separation)                 │  │
            │   │                                                               │  │
            │   │   DETERMINISTIC CORE — validates against hard rules           │  │
            │   │     • two-stage commit (proposal → commit)                    │  │
            │   │     • rejects / blocks / escalates per code, not per prompt   │  │
            │   │     • AI cannot address, override, or route around the core   │  │
            │   │              ▼                                                │  │
            │   │   COMMITTED STATE  +  deterministic audit log                 │  │
            │   │              (log carries the checkpoint ID — end-to-end trace)│  │
            │   └─────────────────────────────────────────────────────────────┘  │
            │                                    ▼                                │
            │              AI COMMUNICATES OUTCOME (no execution authority)       │
            └───────────────────────────────────────────────────────────────────┘
```

Three properties of this diagram are load-bearing. The OOD gate and contradiction gate both exit to safe-default *before* anything reaches structured data — novelty and contradiction never produce a classified value. The checkpoint ID flows through structured data into the deterministic core's audit log, so any committed action traces back to the exact extraction and precedent that produced it. The Kick layer is drawn as the outermost box because it must be able to terminate the pipeline at any interior point — including mid-classification — from infrastructure the interior cannot reach.

---

## 6. Common Anti-Patterns

These are five recurring patterns we have observed across multiple deployments where systems fail TKA compliance. The first three are single-layer violations; the last two are the most frequent partial implementations.

**Self-validating AI (Totem violation).** The agent generates an action, runs a second AI pass to "verify" it, and executes on a clean self-check. The verifier shares context and weights with the generator, so it confirms the original misreading. This fails Totem because no deterministic component holds independent commit authority — the self-check is the authority, and it is probabilistic. Symptom: the audit trail is a model-generated rationale, not a deterministic record.

**Prompt-based loop breaking (Kick violation).** The system instructs the agent to stop if it detects it is looping, repeating work, or making no progress. The stop instruction runs on the same machinery as the loop and inherits its blindness — a probabilistic engine treats every iteration as a clean, locally optimal next move and never recognizes the spiral. This fails Kick because termination authority is inside the loop. Symptom: runaway token spend on a single unsolved task, with the agent reporting confident progress throughout.

**Free-form extraction feeding the deterministic core (Architect violation).** The AI extracts field values from unstructured input with no external vocabulary constraint and no novelty gate, and hands them to a sound deterministic core. The core executes a confident misclassification faithfully. This fails Architect because the structuring step is unconstrained — even a perfect Totem core is only as sound as the data fed to it. Symptom: the system handles known inputs well and silently mishandles novel ones, because novelty never registers as novelty.

**Implementing Totem without Kick (the loop problem survives).** The architecture has a clean deterministic core the AI cannot reach — and the AI loop in front of it can spin indefinitely, exhaust context, and burn budget without ever producing a committable output. The core is safe; the system is not. The loop failure is untouched by execution-authority separation because it occurs entirely upstream of the commit boundary. Symptom: a well-architected core that rarely receives input because the agent feeding it is stuck.

**Implementing Kick without Architect (the seam feeds correct execution but wrong data).** The loop terminates cleanly and the deterministic core executes faithfully — on structured data that encodes a confident misreading the extraction step never caught. Clean termination and faithful execution of wrong data is still a wrong outcome, now produced efficiently and within budget. Symptom: fast, reliable, well-bounded processing of inputs into incorrect structured values, with a tidy audit log that records the wrong answer precisely.

---

## 7. Applying TKA to an Existing System

Auditing an existing system is four steps. Each maps a layer to a specific architectural feature; the absence of that feature is the gap.

**Step 1 — Map the execution authority boundary (Totem).** Trace where irreversible state is committed (database writes, payment rails, order submission, PO issuance). Identify the component that holds commit authority. Ask whether it is deterministic and whether any AI output can override its rules. If the commit is gated by an AI self-check, or the core obeys a model-emitted instruction, the boundary is misplaced.

**Step 2 — Find the loop termination mechanism (Kick).** Locate the agentic runtime and identify what stops it. Ask whether the terminator is infrastructure or prompt. A step budget enforced by the harness passes; a system-prompt instruction to self-limit does not. Confirm that breaker-trip revokes privileges and that writes are idempotent against retry.

**Step 3 — Find the extraction step (Architect).** Locate where unstructured input becomes structured data. Ask whether the vocabulary is external and immutable, whether an OOD gate runs before the classifier, whether the novelty decision uses an external criterion or the model's confidence, and whether precedents are human-authored only. A free-form extraction with confidence-based fallback fails — confidence is the model's, and novelty bypasses it.

**Step 4 — Run the compliance checklist (§4).** Answer all 19 questions. Any layer with a failing question is non-compliant, and the report names the missing feature, the file and line where the boundary lives, and the failure mode left open.

**Worked example — a gap finding.** A finding is specific and located, not a floating concern. Compare:

> *Floating (insufficient):* "The system relies too heavily on the AI to validate its own decisions and should add more oversight."

> *TKA gap finding (correct):*
> **GAP — Layer 3 (Architect), checklist item 14 (OOD gate): FAIL.**
> `extraction/classify.py:142` — `classify_clause()` calls the LLM directly on every input and falls back to `HOLD` only when `response.confidence < 0.7`. The novelty decision is the model's own confidence score. There is no pre-classifier distance computation against a precedent corpus; a novel clause the model is confident about (confidence ≥ 0.7) is classified into an existing category and passed to the core at `core/commit.py:88` with no novelty gate. Failure mode left open: confident wrong-box misclassification (§1, Architect failure). Required fix: compute embedding distance to nearest human-reviewed precedent before invoking `classify_clause()`; route distance > threshold to safe-default; store the threshold as an external schema parameter, not in the prompt or the `0.7` literal.

The gap finding names the layer, the failing checklist item, the exact location of the boundary, the location where the unsafe value reaches the core, the failure mode in force, and the structural fix. That is the output of a TKA audit. "Add more oversight" is not.

### 7.1 Cold-Start Bootstrapping

A precedent corpus that is empty on Day 1 routes nearly all live traffic past the OOD threshold, because nothing yet counts as settled. Before routing live traffic, the corpus must therefore be seeded using historical input logs. The recommended approach:

1. Run offline unsupervised clustering (k-means or hierarchical) on historical inputs to group them into candidate precedent neighborhoods.
2. Human reviewers sign off on the representative centroid of each cluster and a sample of its members — not every individual input.
3. Gate live traffic until a minimum coverage threshold is met (e.g., 80% of top-volume input patterns have an approved precedent).

Without this bootstrapping phase, Day 1 traffic will exceed the OOD threshold at high rates, triggering queue storms that create precisely the operational pressure that leads to mass-approval and corpus poisoning.

---

---

## 8. References

[1] Hendrickx, K., Perini, L., Van der Plas, D., Meert, W., & Davis, J. (2021). *Machine learning with a reject option: A survey.* arXiv:2107.11277. https://arxiv.org/abs/2107.11277

[2] Lee, K., Lee, K., Lee, H., & Shin, J. (2018). *A simple unified framework for detecting out-of-distribution samples and adversarial attacks.* NeurIPS 2018. arXiv:1807.03888. https://arxiv.org/abs/1807.03888

[3] Romanini, D., Albert, J., Pillai, P., & Roth, M. (2022). *AdaDetect: Adaptive black-box novelty detection with statistical guarantees.* arXiv:2208.06685. https://arxiv.org/abs/2208.06685

[4] Angelopoulos, A. N., & Bates, S. (2023). *Conformal prediction: A gentle introduction.* Foundations and Trends in Machine Learning. arXiv:2107.07511. https://arxiv.org/abs/2107.07511

[5] Shumailov, I., Shumaylov, Z., Zhao, Y., Papernot, N., Anderson, R., & Gal, Y. (2024). *AI models collapse when trained on recursively generated data.* Nature, 631, 755–759. https://www.nature.com/articles/s41586-024-07566-y

[6] Amazon Web Services. (2024). *AWS Well-Architected Framework — Machine Learning Lens.* https://docs.aws.amazon.com/wellarchitected/latest/machine-learning-lens/

[7] Dong, Y., et al. (2024). *Attacks, defenses and evaluations for LLM conversation safety: A survey.* arXiv:2402.09283. https://arxiv.org/abs/2402.09283

[8] Guardrails AI / NVIDIA NeMo Guardrails. (2024). *NeMo Guardrails: A toolkit for controllable and safe LLM applications.* arXiv:2310.10501. https://arxiv.org/abs/2310.10501

[9] CoSAI / OASIS Open. (2026). *Model Context Protocol (MCP) Security.* Workstream 4: Secure Design Patterns for Agentic Systems. https://github.com/cosai-oasis/ws4-secure-design-agentic-systems

---

TKA compliance is binary per layer because the *boundary* each layer enforces is binary — but the layers are binaries of different kinds, and precision here matters. For Totem and Kick the binary is structural and complete: a system that almost implements the Kick layer — a step budget the agent can extend by emitting a parameter, a breaker the model can reset — still has the Kick failure mode in full, and the same holds for a Totem core the AI can address. When implemented, these two layers close their failure mode outright. Architect is a structural binary wrapped around a probabilistic residual: the gate that decides novelty is either external and pre-committed or it is the model grading itself, and that distinction is absolute — but a compliant Architect layer *bounds and makes auditable* the residual misclassification risk rather than eliminating it. A system that implements all three layers has closed two failure modes in full and bounded and made auditable the third. That is still a strong, true, defensible claim, and it distinguishes TKA-compliant systems from every alternative. The pattern is opinionated by design: three invariants, three layers, no spectrum on whether each boundary is held. A system either holds each invariant or it does not, and where it does not, the corresponding failure mode is present and unmitigated.

One scope boundary must be stated plainly, because the binary claim invites it. Totem and Kick are binary *given correct rules and thresholds*: the structural property "no probabilistic component holds commit authority" either holds or it does not, but the deterministic core still executes human-authored rules, and a rule that is wrong, incomplete, or gameable will be executed faithfully. TKA closes the *architectural* failure mode (the AI substituting its own judgment for the gate); it does not and cannot certify that the gate's rule set is correct. Rule and threshold correctness is a separate risk surface — owned by the same governance process that owns the Architect corpus — and is out of scope for the structural compliance claim. A TKA-compliant system with a wrong hard rule is structurally sound and operationally wrong; conflating the two is itself a dreaming failure at the design layer.
