# Agent Dreaming

**The Totem/Kick/Architect (TKA) pattern** — a three-layer secure design framework for agentic AI systems that hold execution authority over irreversible state.

---

## The problem

An agent is dreaming when it produces confident, coherent, wrong output it cannot detect from within its own context. Three independent architectural failures cause it:

1. **The Totem failure** — the validator shares context with the generator. A misreading at generation survives self-verification intact.
2. **The Kick failure** — there is no halting oracle inside the loop. The agent cannot recognize its own non-progress.
3. **The Architect failure** — the extraction step is unconstrained. A novel input is confidently mapped to an existing category and the deterministic core executes that misclassification faithfully.

These are orthogonal. Closing one leaves the other two open.

---

## The pattern

| Layer | Failure addressed | Invariant |
|---|---|---|
| **Totem** | Self-validated commit | No probabilistic component holds authority to commit irreversible state |
| **Kick** | Non-halting reasoning | The loop is terminated from infrastructure the model cannot reach |
| **Architect** | Unconstrained extraction | An external, pre-committed OOD gate governs input admissibility before the classifier runs |

Compliance is binary per layer. The system is TKA-compliant only if all three pass.

---

## Contents

- [`WHITEPAPER.md`](WHITEPAPER.md) — Full pattern specification with invariants, 19-item compliance checklist, reference architecture, anti-patterns, and audit methodology (v1.1)
- [`cosai/`](cosai/) — Proposed contribution to [CoSAI WS4](https://github.com/cosai-oasis/ws4-secure-design-agentic-systems): operationalizing MCP-T9 (Trust Boundary / Overreliance on LLM)

---

## Relationship to CoSAI MCP Security

The CoSAI MCP Security paper ([WS4, January 2026](https://github.com/cosai-oasis/ws4-secure-design-agentic-systems)) defines a rigorous twelve-category threat taxonomy for MCP deployments. MCP-T9 (Trust Boundary and Privilege Design Failures) states the control objective — "restricting LLM judgment for security-critical decisions" — but does not supply an architectural pattern or a compliance checklist for meeting it. TKA is proposed as that pattern, with a mapping across all twelve MCP threat categories.

See [`cosai/WS4_contribution.md`](cosai/WS4_contribution.md) for the full mapping and proposed integration.

---

## Status

- Pattern spec: **v1.1** (stable; adversarially reviewed)
- CoSAI contribution: **draft** (Claim 2 prior-art review pending)
- Audit tool: planned

---

## License

Apache 2.0
