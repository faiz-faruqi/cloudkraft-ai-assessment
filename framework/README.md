# CloudKraft AI Control Tower - Framework

The CloudKraft AI Control Tower framework is a five-layer methodology for designing, governing, and operationalising enterprise GenAI platforms. It is opinionated, tool-specific, and built for practitioners who need to make real architectural decisions - not consultants who need to avoid them.

Each layer addresses a distinct architectural concern. Each has named patterns, real tool comparisons, anti-patterns drawn from observed failures, and key questions that surface in every serious enterprise AI engagement.

## The Five Layers

| Layer | Concern | Primary Stakeholder |
|---|---|---|
| [1 - Access & Governance](./layer-1-access-governance.md) | How AI access, policy, and cost are controlled | CISO, FinOps, CTO |
| [2 - Data & Retrieval](./layer-2-data-retrieval.md) | How enterprise knowledge grounds AI systems | CDO, Data Architecture |
| [3 - Orchestration](./layer-3-orchestration.md) | How AI workflows are designed and maintained | VP Engineering, Platform |
| [4 - Evaluation & Trust](./layer-4-evaluation-trust.md) | How AI quality is measured over time | Head of AI, Risk |
| [5 - Integration](./layer-5-integration.md) | How AI connects to enterprise systems | Enterprise Architecture |

## How to Use This Framework

**If you are evaluating CloudKraft services:** Read the layer most relevant to your current pain. The tool selection matrices show you what an opinionated senior architect recommends and why - without a vendor agenda.

**If you are inside a CloudKraft engagement:** These documents are the methodology backbone. Each Tier 1 Assessment scores your organisation against these five layers. Each Tier 2 Blueprint produces the target architecture for the layers with the lowest scores.

**If you are an architect building your own AI governance programme:** Start with Layer 1 (Access & Governance) regardless of where you think your biggest gap is. In practice, ungoverned access is the root cause of most other AI governance failures.

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

**Tool-specific over abstract.** Naming real tools (LangSmith, Portkey, Presidio, LangGraph) is more useful than generic categories. Tool recommendations are vendor-neutral in the sense that we hold no reseller relationships, not in the sense that we refuse to name a winner.

**Layered over monolithic.** Each layer can be addressed independently. A mature Layer 2 does not require a mature Layer 1 - though the combination is dramatically more powerful than either alone.

**Evidence-based over theoretical.** Every anti-pattern in this framework has been observed in real enterprise AI programmes. Every pattern recommendation has been implemented and validated.

## Related Resources

- [Assessment Tool](https://assessment.cloudkraft.com) - Score your organisation across all five layers
- [Patterns](../patterns/) - Named, reusable architectural patterns
- [Anti-patterns](../anti-patterns/) - What enterprises consistently get wrong
- [Templates](../templates/) - Engagement deliverable templates
- [CloudKraft Services](https://cloudkraft.com) - Tier 1, 2, and 3 advisory offerings
