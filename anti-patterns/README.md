# Anti-Patterns

What enterprises consistently get wrong with AI governance, data infrastructure, orchestration, evaluation, and integration — documented from observed failures across real programmes.

Anti-patterns are not theoretical. Every entry in this directory has been observed in one or more enterprise AI engagements. Some appear in nearly every organisation at early maturity stages. Understanding them is one of the fastest ways to accelerate an AI governance programme — not by studying what good looks like, but by recognising the failure modes before they compound.

---

## Anti-Pattern Index

| Anti-Pattern | Layer | Risk Level | How Common |
|---|---|---|---|
| [Shadow AI Sprawl](./shadow-ai-sprawl.md) | 1 — Access & Governance | High | Universal |
| [Ungoverned Cost Accumulation](./ungoverned-cost-accumulation.md) | 1 — Access & Governance | High | Universal |
| [BYOLLM Chaos](./byollm-chaos.md) | 1 — Access & Governance | High | Very Common |
| [Evaluation After the Fact](./evaluation-after-the-fact.md) | 4 — Evaluation & Trust | High | Very Common |
| [Tight Coupling of AI Systems](./tight-coupling-ai-systems.md) | 5 — Integration | Medium | Common |

---

## How to Use This Directory

**If you are conducting a CloudKraft assessment:** These anti-patterns correspond directly to questions in the Tier 1 Diagnostic. When a client demonstrates one of these patterns during discovery, it is a reliable indicator of the maturity level and the priority order for remediation.

**If you are building an AI governance programme:** Read the anti-patterns for the layer you are addressing before reading the corresponding patterns. Understanding what fails and why is more durable than understanding what works in isolation.

**If you are reviewing an existing AI system:** Each anti-pattern document includes observable symptoms — signals you can look for in a real system rather than abstract failure descriptions.

---

## Risk Level Definitions

**High:** Directly exposes the organisation to financial, compliance, or security risk. Remediation should be treated as urgent.

**Medium:** Produces technical debt and operational fragility that will compound over time. Remediation should be planned within the current programme cycle.

**Low:** Suboptimal practice that reduces velocity and increases maintenance cost without immediate risk.

---

## Relationship to Patterns

Every anti-pattern in this directory has a corresponding pattern that addresses it:

| Anti-Pattern | Remediation Pattern |
|---|---|
| Shadow AI Sprawl | [Centralised AI Gateway](../patterns/centralised-ai-gateway.md) |
| Ungoverned Cost Accumulation | [Centralised AI Gateway](../patterns/centralised-ai-gateway.md) |
| BYOLLM Chaos | [Centralised AI Gateway](../patterns/centralised-ai-gateway.md) |
| Evaluation After the Fact | [Continuous Evaluation](../patterns/continuous-evaluation.md) |
| Tight Coupling of AI Systems | [AI Workflow Orchestration Bridge](../patterns/ai-workflow-orchestration.md) |

---

*Part of the [CloudKraft AI Control Tower framework](../framework/README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to identify which anti-patterns are present in your organisation.*
