# CloudKraft AI Control Framework - Framework

The CloudKraft AI Control Framework is a five-layer methodology for designing, governing, and operationalising enterprise GenAI platforms. It is opinionated, tool-specific, and built for practitioners who need to make real architectural decisions - not consultants who need to avoid them.

Each layer addresses a distinct architectural concern. Each has named patterns, real tool comparisons, anti-patterns drawn from observed failures, and key questions that surface in every serious enterprise AI engagement.

## The Five Layers

| Layer | Concern | Primary Stakeholder |
|---|---|---|
| [1 - Data Readiness](./layer-1-data-readiness.md) | How data quality, lineage, and vector stores prepare information for AI consumption | CDO, Data Architecture |
| [2 - Architecture Patterns](./layer-2-architecture-patterns.md) | How RAG design, agent orchestration, and system topology structure AI integration | VP Engineering, Platform |
| [3 - Governance & Risk](./layer-3-governance-risk.md) | How RBAC, audit logging, and responsible AI controls manage regulatory compliance | CISO, Risk, Legal |
| [4 - Operating Model](./layer-4-operating-model.md) | How team structure, ownership, and vendor management govern the AI programme | Head of AI, CTO |
| [5 - Integration & Scale](./layer-5-integration-scale.md) | How workflow embedding, observability, and cost control harden AI in production | Enterprise Architecture, FinOps |

## How to Use This Framework

**If you are evaluating CloudKraft services:** Read the layer most relevant to your current pain. The tool selection matrices show you what an opinionated senior architect recommends and why - without a vendor agenda.

**If you are inside a CloudKraft engagement:** These documents are the methodology backbone. Each Tier 1 Assessment scores your organisation against these five layers. Each Tier 2 Blueprint produces the target architecture for the layers with the lowest scores.

**If you are an architect building your own AI governance programme:** Start with Layer 1 (Data Readiness) regardless of where you think your biggest gap is. In practice, ungoverned data is the root cause of most downstream AI quality and trust failures.

## The Maturity Scale

Each layer is scored on a 1-5 scale in the CloudKraft Assessment:

| Score | Label | What It Means |
|---|---|---|
| 1 | Initial | Ad hoc, reactive, no formal approach |
| 2 | Developing | Awareness exists, partial controls, inconsistent application |
| 3 | Defined | Documented approach, consistent application, some automation |
| 4 | Managed | Measured, monitored, proactively optimised |
| 5 | Optimising | Continuous improvement, industry-leading practice |

A score of 3 (Defined) is the minimum viable threshold for a regulated enterprise. Most organisations engaging CloudKraft score between 1.5 and 2.8 on first assessment.

## Design Principles

**Opinionated over neutral.** This framework makes explicit recommendations. When there are two reasonable options, we tell you which one we would choose and why. You may disagree - the ADR format is designed exactly for that.

**Tool-specific over abstract.** Naming real tools is more useful than generic categories. Tool recommendations are vendor-neutral in the sense that we hold no reseller relationships, not in the sense that we refuse to name a winner.

**Layered over monolithic.** Each layer can be addressed independently. A mature Layer 2 does not require a mature Layer 1 - though the combination is dramatically more powerful than either alone.

**Evidence-based over theoretical.** Every anti-pattern in this framework has been observed in real enterprise AI programmes. Every pattern recommendation has been implemented and validated.

## Related Resources

- [Assessment Tool](https://assessment.cloudkraft.com) - Score your organisation across all five layers
- [Patterns](../patterns/) - Named, reusable architectural patterns
- [Anti-patterns](../anti-patterns/) - What enterprises consistently get wrong
- [Templates](../templates/) - Engagement deliverable templates
- [CloudKraft Services](https://cloudkraft.com) - Tier 1, 2, and 3 advisory offerings
