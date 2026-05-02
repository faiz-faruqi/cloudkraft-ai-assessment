# AI Governance Maturity Model

The CloudKraft AI Governance Maturity Model provides a structured, layer-by-layer framework for assessing where an organisation stands in its AI governance programme and what it needs to do to improve.

The model is built on two foundational observations from practice:

1. **Organisations do not advance uniformly across layers.** A team can be at Level 4 in Orchestration (sophisticated multi-agent pipelines) and Level 1 in Access & Governance (no centralised gateway, no cost visibility). Assessing each layer independently surfaces these asymmetries, which are the actual source of most enterprise AI governance risk.

2. **Most organisations overestimate their own maturity.** When teams self-assess, they tend to score their intended practices rather than their actual practices. A gateway that exists but does not route all traffic is not a Level 3 gateway. An evaluation framework that was used once at launch but is not run regularly is not a Level 3 evaluation practice. The assessment questions are designed to expose the gap between intent and execution.

---

## The Five-Layer Model

Each of the five Control Tower layers is assessed independently on a 1–5 scale.

| Layer | Question It Answers | Primary Stakeholder |
|---|---|---|
| [1 — Access & Governance](../framework/layer-1-access-governance.md) | How does the organisation control who uses which AI systems, at what cost, under what policy? | CISO, FinOps, CTO |
| [2 — Data & Retrieval](../framework/layer-2-data-retrieval.md) | How does enterprise knowledge ground AI systems safely and accurately? | CDO, Data Architecture |
| [3 — Orchestration](../framework/layer-3-orchestration.md) | How are multi-step AI workflows designed, governed, and maintained? | VP Engineering, Platform |
| [4 — Evaluation & Trust](../framework/layer-4-evaluation-trust.md) | How is AI quality measured and maintained over time? | Head of AI, Risk |
| [5 — Integration](../framework/layer-5-integration.md) | How does AI connect to enterprise systems reliably and auditably? | Enterprise Architecture |

---

## The Five Maturity Levels

Each level is defined by observable, verifiable practices — not intentions or plans.

### Level 1 — Initial

**Characteristics:** Ad hoc and reactive. No formal approach to AI governance. Individual teams or individuals make their own AI decisions without enterprise oversight.

**Typical observable evidence:**
- Developers hold individual LLM API keys charged to personal or team budgets
- There is no approved model list or it exists only on paper
- AI systems go to production when developers feel they are ready — no formal quality gate
- No central visibility into what AI tools the organisation uses, what they cost, or what data they handle
- Compliance obligations relating to AI are unaddressed

**What Level 1 means in practice:** The organisation is using AI. Nobody knows the full scope of that usage, what it costs, what data it touches, or whether it is working. This is the starting state for most organisations in the first 12–18 months of AI adoption.

---

### Level 2 — Developing

**Characteristics:** Awareness of the problem exists. Partial controls are in place. Application is inconsistent across teams.

**Typical observable evidence:**
- A centralised team manages some API keys but teams routinely work around the process
- An approved model list exists but is not enforced at the technical level
- Some teams run informal tests before deployment; others do not
- A rough AI cost estimate exists but cannot be attributed below the organisation level
- Shadow AI is acknowledged but not systematically measured or contained
- Data residency requirements are discussed but not uniformly applied

**What Level 2 means in practice:** The organisation has recognised the governance problem and started to address it. The controls that exist are process-based rather than technical, which means they depend on team discipline to be effective. Level 2 governance fails under pressure.

---

### Level 3 — Defined

**Characteristics:** Documented approach, consistent application, technical enforcement for core controls. This is the minimum viable threshold for a regulated enterprise.

**Typical observable evidence:**
- A centralised AI gateway routes all sanctioned LLM traffic
- Team-level authentication and cost attribution are in place
- An approved model list is enforced at the gateway layer (not just on paper)
- A governance policy document exists, has been reviewed by legal and security, and has been communicated to affected teams
- At least one AI system has a formal evaluation baseline and a pre-deployment quality gate
- A test set with human-labelled examples exists for each production AI system
- Data classification at the retrieval layer is documented and enforced
- A shadow AI containment programme has been run and documented

**What Level 3 means in practice:** The organisation has crossed the threshold from "we know we should do this" to "we have done this consistently." Level 3 practices hold up under pressure because they are technical controls, not process controls. A regulator presented with a Level 3 organisation's AI governance evidence will find something to review — not a blank page.

---

### Level 4 — Managed

**Characteristics:** Measured, monitored, and proactively optimised. Problems are detected before they become incidents.

**Typical observable evidence:**
- Real-time cost dashboards show spend per team, per model, and per use case
- Budget alerts fire before overruns; rate limiting prevents cost spikes
- Shadow AI detection runs continuously via network egress analysis
- Continuous production evaluation runs automatically; regression alerts are configured and tested
- RAGAS or equivalent metrics are tracked over time with trend visibility
- Human feedback is systematically collected and incorporated into the evaluation pipeline
- Orchestration workflows have explicit cost tracking, iteration limits, and circuit breakers
- Data freshness is monitored with automated staleness detection
- The AI governance programme has been tested against a real compliance or audit request

**What Level 4 means in practice:** The organisation is not just running good AI governance — it is measuring the quality of its governance practices and detecting gaps before they become problems. Level 4 organisations are early in the conversation when a new regulatory requirement emerges rather than scrambling to comply retroactively.

---

### Level 5 — Optimising

**Characteristics:** Continuous improvement, industry-leading practice, AI governance treated as a competitive capability.

**Typical observable evidence:**
- Dynamic model governance with risk-tiered access — different data classification tiers automatically route to different model allowlists
- Automated compliance evidence collection — the gateway generates regulatory audit reports programmatically
- Shadow AI detection operates within hours of a new tool appearing on the network
- FinOps integration provides full AI cost visibility alongside cloud compute, storage, and SaaS spend
- Real-time drift detection for production AI systems
- Business-outcome metrics (task completion rate, escalation rate, user satisfaction) tracked alongside technical metrics
- The AI evaluation framework is used to assess third-party AI risk, not just internal systems
- The organisation contributes to industry frameworks or standards for AI governance
- The AI platform is treated as critical infrastructure with the same SLOs and change management standards as core business systems

**What Level 5 means in practice:** AI governance is a differentiator, not just a compliance requirement. Level 5 is attainable but rare. In practice, achieving Level 4 consistently across all five layers is the realistic goal for most regulated enterprises within a 2–3 year programme.

---

## Score Aggregation

Each layer is scored independently. The overall programme maturity score is the unweighted average of the five layer scores, rounded to one decimal place.

**However, the overall average is less useful than the layer-by-layer profile.** An organisation scoring 4.0 overall with one layer at 1.5 has a critical vulnerability that the average obscures. Always present the layer-by-layer breakdown alongside the overall score.

**Minimum viable threshold:**
Level 3 (Defined) across all five layers is the minimum threshold for a regulated enterprise to be considered defensibly governed. A single layer below Level 3 in a regulated context represents a material governance gap.

**Typical first-assessment profiles:**
Most organisations engaging CloudKraft for the first time score between 1.5 and 2.8 overall. The most common profile is Layer 3 at a higher maturity than the other layers (because sophisticated data pipelines often predate AI governance) with Layers 1 and 4 significantly lower.

---

## Using Scores to Drive Prioritisation

Layer scores are not independent — foundational layers enable higher-layer maturity.

**Layer 1 is the prerequisite.** You cannot govern what you cannot see. Without a centralised gateway, cost attribution, and an audit trail, the investments in Layers 2–5 are less effective because they lack the visibility and enforcement infrastructure that Layer 1 provides.

**Layer 4 depends on Layers 1, 2, and 3.** Continuous evaluation requires audit log data from the gateway, retrieval metadata from the RAG pipeline, and execution data from the orchestration layer. A Layer 4 maturity target without Layer 1 in place will produce limited evaluation capability.

**The typical remediation sequence:** Layer 1 → Layer 2 → Layer 4 → Layers 3 and 5 in parallel.

---

## Reassessment Cadence

**First assessment:** Establishes the baseline. Every gap identified becomes a potential roadmap item.

**Reassessment at 6 months:** After foundational infrastructure is deployed, a reassessment validates that scores have moved as expected and identifies new gaps exposed by the first round of improvements.

**Annual reassessment:** Tracks long-term programme maturity. Also appropriate after a significant change: new regulatory requirements, a major AI investment, an acquisition, or an AI incident.

---

*Part of the [CloudKraft AI Control Tower framework](../framework/README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to receive a scored maturity profile for your organisation.*
