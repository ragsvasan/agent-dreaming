# Operationalizing MCP-T9: a secure design pattern for restricting LLM judgment

**Proposed augmentation to the T9 mitigation guidance in a future revision of the MCP Security paper.**
Author: Rags Vasan · Status: draft for WS4 discussion · Companion spec: *The Totem/Kick/Architect Pattern* (v1.1)

> This is not a competing framework. It is proposed text for the gap the MCP paper already names
> but leaves open: T9 states that LLM judgment should be restricted for security-critical decisions,
> and stops there. This document supplies the pattern and a compliance checklist that make the
> control checkable. The companion spec carries the full invariants and reference implementation;
> this document carries the T1–T12 mapping and the proposed integration.

---

## 1. The gap in T9

MCP-T9 lists *overreliance on the LLM* and *overreliance on human-in-the-loop* as vulnerabilities.
Its control objective is **"restricting LLM judgment for security-critical decisions,"** mapped to
the *Secure Tool / UX Design* mitigation class (§3.2.8).

The objective is right. The paper does not say how to meet it. There is no architectural pattern
that enforces the restriction, and no way for a deployment to demonstrate it holds. A reviewer
auditing an agentic system against T9 today has a goal and no test.

The failure T9 points at is not a protocol flaw. It survives full T1–T12 hardening because it
lives in the model's own decision process: the agent forms a confident, internally-coherent, wrong
judgment that it cannot detect from inside its own context, then acts on it. This takes three
distinct forms, and they are orthogonal. Closing one leaves the other two open.

1. **Self-validated commit.** The validator shares context and weights with the generator. A
   misreading present at generation is present at verification, and the action commits on the same
   flawed premise.
2. **Non-halting reasoning.** No halting oracle sits inside the loop. The agent cannot recognize
   its own non-progress, and a prompt instruction to stop runs on the machinery that spiraled.
3. **Unconstrained extraction.** The step that turns unstructured input into structured data for
   the deterministic core is ungoverned. A novel input gets mapped into an existing category with
   full confidence, and the core executes that choice faithfully.

---

## 2. The pattern

TKA (Totem / Kick / Architect) is three layers, one per failure above. Compliance is binary per
layer. Full requirements and the 19-item checklist live in the companion spec. The essentials:

**Totem — execution-authority separation.** No probabilistic component holds authority to commit
irreversible state. Authority lives in a deterministic core the AI cannot address, override, or
route around. The AI's output is data the core validates, never an instruction the core obeys.
Irreversible actions use a two-stage commit: the AI proposes, the core or a human commits.

**Kick — external loop termination.** A stateful circuit breaker runs in the hosting
infrastructure, not in the prompt or the tool set. It enforces a hard step budget and a no-progress
detector keyed on a content hash of successive outputs, so an agent cannot dodge it by varying its
wording. On a trip it revokes the agent's privileges, packages state, and hands off to a human. No
termination logic runs on the agent's weights.

**Architect — extraction governance.** The AI classifies against a human-ratified vocabulary it
cannot extend at runtime. Before the classifier runs, an external pre-committed out-of-distribution
(OOD) gate decides whether the input is close enough to settled precedent to be classified at all.
Distant inputs route to a conservative safe-default and the classifier never runs. Precedents are
human-authored only. A contradiction gate rejects extractions that conflict with settled precedent.
Every extraction emits a self-contained checkpoint.

---

## 3. What is new, and what is not

This contribution does not claim all three layers as novel, and WS4 should hold it to that line.

Totem is existing consensus. AWS Well-Architected guidance for AI workloads and the broad class of
deterministic-execution frameworks already establish that irreversible action belongs behind a
non-probabilistic gate. Kick is covered by AgentSpec, GuardAgent, NeMo Guardrails, and similar
runtime-supervision systems. A team that has implemented those has substantially built Layers 1
and 2.

Three things are contributed, at different levels of novelty:

**1. Architect's pre-classifier OOD admission gate — a novel-combination claim.**

The individual components are prior art. Mahalanobis distance OOD detection (Lee et al., 2018,
[arXiv:1807.03888](https://arxiv.org/abs/1807.03888)), semi-supervised novelty scoring (AdaDetect,
[arXiv:2208.06685](https://arxiv.org/abs/2208.06685)), and conformal prediction abstention are all
established methods. The reject-option literature (surveyed in Hendrickx et al.,
[arXiv:2107.11277](https://arxiv.org/abs/2107.11277)) names the architectural role: a "separated
rejector" — a component that decides admissibility before the classifier runs.

What is distinct is the placement and its governance property. Standard LLM guardrails (NeMo
Guardrails, Llama Guard, GuardAgent) validate model *output* — they run after the model. Reject-
option and selective-prediction classifiers fold rejection into the classifier itself via a reject
class or an internal confidence threshold. Conformal abstention is also internal. Architect's gate
is a standalone upstream component using an external, pre-committed criterion against a fixed
corpus, independent of the classifier's confidence at inference time. The admissibility decision
cannot be overridden by model reasoning. The contribution is the governance invariant and its
architectural placement, not a new algorithm.

**2. The strictly human-authored precedent corpus — a design principle, not yet a confirmed novel
claim.**

The principle: AI-generated extractions never become governing precedents, neither as hard
constraints nor weighted suggestions, because either path reintroduces the reasoning loop Architect
exists to prevent. This amortizes human judgment into a ratified corpus rather than spending it per
transaction — a direct answer to T9's *overreliance on human-in-the-loop* and to consent fatigue.

Model-collapse findings (Shumailov et al., Nature 2024) motivate the concern directionally but are
not a direct citation: they address iterative self-training on generative models, not a governing
precedent corpus. To our knowledge this specific governance invariant has not been named as a
pattern in the case-based reasoning or HITL-active-learning literature. We flag it as a candidate
for further prior-art review before this contribution is finalized.

**3. The joint-composition requirement — a synthesis claim.**

All three layers must be present together because the failure modes are orthogonal and any two
leave the third in full force. No existing agentic security framework (MAESTRO, OWASP Agentic,
NIST AI RMF, MITRE ATLAS) composes exactly these three boundaries under a "no partial credit"
invariant. The composition is the contribution, not any individual layer.

Totem and Kick are the cited substrate Architect sits on. They are not co-equal novelty claims.

---

## 4. Mapping to MCP-T1–T12

For each threat: the paper's control objective, what TKA adds at the reasoning layer, and the
residual. The *Not addressed* rows bound the claim and matter as much as the rest.

| MCP threat | Paper's control objective | TKA contribution | Fit | Residual / limit |
|---|---|---|---|---|
| **T1** Improper Auth / Identity | Agent identity, secure delegation | — | Not addressed | TKA presumes authenticated identity, solved upstream. |
| **T2** Missing / Improper Access Control | Secure delegation, RBAC | Totem: even with over-broad grants, irreversible commit requires the deterministic core, and no AI output overrides a core rule. A backstop behind RBAC. | Backstop | Reversible actions within granted scope still execute. TKA does not fix the grant. |
| **T3** Input Validation / Sanitization | Data sanitization, guardrails | — | Not addressed | Injection sanitization is a protocol-layer concern, upstream of extraction. |
| **T4** Input/Instruction Boundary (prompt injection, tool poisoning) | Input sanitization, context isolation | Architect routes novel or anomalous extractions to safe-default. Totem ensures a successful injection cannot emit a core-overriding instruction. Bounds blast radius at the extraction-to-core seam. | Bounded | Does not prevent injection. Inputs engineered to read as corpus-familiar can pass the OOD gate. Corpus audit and threshold-drift detection bound this; they do not close it. |
| **T5** Inadequate Data Protection | Encryption, secrets mgmt | — | Not addressed | Confidentiality is orthogonal to reasoning governance. |
| **T6** Missing Integrity / Verification | Code signing, tamper-evident logs | Architect's checkpoint binds raw input, external-source content hashes, and schema/encoder version into replayable extraction provenance. | Secondary | Complements, does not replace, message- and tool-definition integrity. |
| **T7** Session / Transport Security | TLS, session lifecycle | — | Not addressed | Transport layer. |
| **T8** Network Binding / Isolation | Segmentation, binding | — | Not addressed | Network layer. |
| **T9** Trust Boundary / Overreliance on LLM | **Restrict LLM judgment for security-critical decisions** | Primary. Totem operationalizes "no probabilistic component holds commit authority." Architect's human-only corpus replaces unsustainable per-transaction review. This is the pattern the control objective currently lacks. | **Primary** | Architect's residual (§5) applies. Totem and Kick are structurally complete when implemented. |
| **T10** Resource Mgmt / Rate-Limit Absence (Denial of Wallet, recursive task exhaustion) | Rate limiting, quotas | Primary. Kick's external step budget, content-hash plan-churn breaker, and privilege revocation on trip answer recursive task exhaustion and Denial of Wallet directly. | **Primary** | Cross-service committed state on a mid-flight trip needs a bound Saga/compensation signal, a Layer-2 co-requirement. |
| **T11** Supply Chain / Lifecycle | Server provenance, signed tools | Defense-in-depth only: a poisoned or shadow tool still cannot commit through the deterministic core. | Not claimed | Provenance and verification are the real control. TKA is a downstream backstop. |
| **T12** Insufficient Logging / Observability | Audit logging, traceability | Strong. Totem's audit record is a product of deterministic execution, not an AI summary. Architect's checkpoint gives an end-to-end replayable trace from raw input through gate verdict and matched precedent to execution. Kick packages state on every trip, defeating stateless-loop blindness. | **Strong** | Protocol-layer observability (tool invocations, auth decisions) stays the host's responsibility. |

TKA materially addresses T9 (primary, it supplies the missing pattern), T10 (primary), and T12
(strong), with a backstop on T2, a bounded contribution on T4, and a secondary one on T6. It does
not address T1, T3, T5, T7, T8, or T11. Those are protocol, transport, and supply-chain threats
outside the reasoning residual.

---

## 5. Limitations

Carried from the spec, not softened.

**Architect bounds misclassification risk; it does not eliminate it.** The OOD gate's novelty
decision is external and pre-committed, which is auditable and better than model self-confidence.
But the vectors it scores come from a learned encoder with its own blind spots. Encoder anisotropy
lets an adversarially perturbed input read as close to settled precedent and slip the gate. The
honest framing is auditable governance of the extraction boundary with a bounded residual, not a
security guarantee. Deployments should score novelty with a covariance-aware metric over the
precedent distribution rather than bare cosine similarity. That narrows the residual. It does not
close it.

**Cold-start is structural.** Architect needs a populated human-ratified corpus. On Day 1
everything is OOD and the review queue floods. In fast-moving domains such as procurement, legal
compliance, and frontier clinical care, bootstrapping is a steady-state condition that needs
continuous ratification capacity, not a one-time seeding effort.

**Forged familiarity is unsolved.** Architect was designed for honest novelty. An input engineered
to look corpus-familiar while carrying novel risk is a standing residual. In highly adversarial
domains Architect is a first line of defense, not a complete one.

---

## 6. Reference artifact

The companion spec includes a binary 19-item compliance checklist (six Totem, seven Kick including
a multi-service Saga co-requirement, seven Architect) and a worked gap-finding format. A finding
names the failing layer, the checklist item, the file and line where the boundary lives, the line
where an unsafe value reaches the core, and the structural fix. A working audit tool emits these
findings against a codebase, and a reference pipeline implements all three layers.

All live deployment thresholds (OOD distances, step and write budgets, breaker caps, similarity
cut-offs) are deployment-set schema values. They are not included in any published artifact.
Examples in the spec are illustrative.

---

## 7. Proposed integration

We propose adding this pattern to the T9 mitigation guidance in a future revision, in two places.

**In the T9 row of the threat table (§3, Control and Mitigation column),** alongside *Secure tool
design / UX design*, add a reference to a checkable execution-authority pattern: probabilistic
components may form judgments but may not hold commit authority over irreversible state; that
authority sits in a deterministic core the model cannot address; human judgment is amortized into a
ratified precedent corpus rather than spent per transaction.

**In §3.2.8 (Secure Tool and UX Design),** add the three-layer pattern and its binary compliance
checklist as a normative reference for meeting the T9 objective, with the limitations in §5 stated
in line so the residual is on the record.

The checklist and audit tool are offered as the reference-implementation deliverable the paper
anticipates for follow-on work.

Open items for the workstream: whether the checklist belongs inline or as a linked annex, and
whether Architect's residual is better placed in the T9 mitigation text or in a limitations note
attached to it.
