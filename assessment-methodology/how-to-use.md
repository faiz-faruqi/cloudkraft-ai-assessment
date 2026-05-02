# How to Use the CloudKraft Assessment

This document explains how the CloudKraft AI Governance Maturity Assessment works, how to interpret your results, and how to use those results to drive a prioritised AI governance programme.

---

## Who This Assessment Is For

The assessment is designed for leaders and practitioners responsible for enterprise AI governance. It is most useful for:

- **CIOs, CTOs, and Heads of AI** evaluating the maturity of their AI programme and identifying where investment will have the most impact
- **CISOs and Compliance Officers** assessing AI-related risk exposure before a regulatory review
- **Enterprise Architects** evaluating the technical foundation their AI programme is built on
- **CloudKraft clients and prospects** who want a structured baseline before engaging advisory services

The assessment is not a vendor evaluation or a tool comparison. It measures the quality of your AI governance practices, not the tools you use to implement them.

---

## How the Assessment Works

### Format

The interactive online assessment at [assessment.cloudkraft.com](https://assessment.cloudkraft.com) asks approximately 25–30 questions across the five Control Tower layers. Each question describes an observable practice (not an intention or a plan) and asks you to indicate the degree to which your organisation has implemented it.

Completion time: **5–8 minutes** for a single respondent.

### Question Design

Each question is designed to expose the gap between intent and execution. The answer choices are calibrated so that:

- **Selecting a higher score requires evidence, not aspiration.** "We have a centralised gateway" earns more points than "We are planning to implement one."
- **Partial implementation is represented.** The choices include states like "implemented for some teams" or "in place but not consistently enforced" — these score between the full-implementation and not-started answers.
- **The question cannot be gamed by answering what sounds best.** The scoring weights are calibrated so that overrating your practices produces a misleadingly high score that does not survive a follow-up conversation with a CloudKraft assessor.

### Scoring

Each question contributes to one of the five layer scores. Layer scores are averaged from the question scores within that layer. The overall score is the average of the five layer scores.

The scoring uses a 1.0–5.0 scale with 0.5 increments. The resulting profile looks like:

```
Layer 1 — Access & Governance:   2.5 ████████░░░░░░░░░░░░
Layer 2 — Data & Retrieval:      3.0 ████████████░░░░░░░░
Layer 3 — Orchestration:         2.0 ████████░░░░░░░░░░░░
Layer 4 — Evaluation & Trust:    1.5 ██████░░░░░░░░░░░░░░
Layer 5 — Integration:           2.5 ████████░░░░░░░░░░░░
Overall:                         2.3 █████████░░░░░░░░░░░
```

---

## Interpreting Your Results

### Reading the Layer Profile

The layer profile is more diagnostic than the overall score. Look for:

**Asymmetries:** A layer significantly below the others is typically the binding constraint on the programme. A Layer 1 score of 1.5 in an organisation with a Layer 2 score of 3.5 means that sophisticated data infrastructure is operating without consistent governance — the quality of the retrieval layer is not producing its full value because the access and audit layer cannot enforce it.

**The floor:** The minimum score across all five layers is the practical ceiling for your AI governance posture. A weakness in a foundational layer limits the effectiveness of strengths in layers that depend on it.

**Regulatory adequacy:** Level 3 across all layers is the minimum threshold for a regulated enterprise. Any layer below 3 in a regulated context is a material gap.

### Common Profiles and What They Mean

**Profile: High Layer 2, Low Layers 1 and 4 ("sophisticated but ungoverned")**
Common in data-mature organisations (financial services, healthcare) that have invested in data infrastructure before addressing AI governance. You have good retrieval infrastructure but no audit trail for what it is retrieving on behalf of whom, and no systematic measurement of whether retrieval quality is being maintained. Priority: Layer 1 gateway first, then Layer 4 evaluation.

**Profile: Low Layer 1, mixed everything else ("grassroots AI")**
Common in technology companies where engineers have been building AI products independently for years. Every team has made their own AI decisions. There is useful capability scattered across the organisation but no governance, no consistent standards, and no centralised visibility. Priority: Layer 1 foundation before anything else — the investments in other layers will not hold without it.

**Profile: Even 2–2.5 across all layers ("early programme")**
Common in organisations that have started to think about AI governance but have not yet made the foundational investments. Nothing is terrible, nothing is good. The good news is that improvement in Layer 1 will lift all other layers. Priority: Layer 1 first, then the use case with the highest combined value and feasibility score.

**Profile: Layer 1 at 3–4, other layers at 1–2 ("governed but shallow")**
Common in organisations that prioritised access control and compliance over capability. You have good visibility and policy enforcement, but the systems being governed are not sophisticated — basic single-step LLM calls with no retrieval architecture and no evaluation. Priority: Layer 2 (retrieval) to build capability, Layer 4 (evaluation) to measure it.

---

## Using Results to Drive a Programme

### Step 1 — Baseline and Gap Analysis

The assessment output is a starting point, not a final answer. Use it to:

1. Identify the two or three layers with the lowest scores — these are the priority areas
2. Within each low-scoring layer, identify the specific practices that are below the threshold
3. Map each gap to the pattern or template that addresses it

A Layer 1 score of 1.5 maps directly to the [Centralised AI Gateway pattern](../patterns/centralised-ai-gateway.md) and the [Shadow AI Sprawl](../anti-patterns/shadow-ai-sprawl.md) and [Ungoverned Cost Accumulation](../anti-patterns/ungoverned-cost-accumulation.md) anti-patterns.

### Step 2 — Prioritise the Remediation Roadmap

Not all gaps are equal. Prioritise based on:

- **Risk exposure:** Gaps in Layer 1 (access and governance) carry direct compliance risk. Address these first.
- **Dependencies:** Layer 1 is a prerequisite for Layers 2–5. Addressing higher layers without Layer 1 in place produces diminishing returns.
- **Effort vs. impact:** Some gaps are quick to close (publishing an AI usage policy, configuring budget alerts). Others require significant build effort (deploying a vector database, building a golden evaluation dataset). Start with high-impact, low-effort improvements.

The [Roadmap template](../templates/roadmap.md) provides the structure to capture this prioritisation in a form that can be presented to executive sponsors and tracked over time.

### Step 3 — Use the Framework as a Delivery Guide

The five-layer framework is not just an assessment rubric — it is a delivery guide. For each layer you are addressing:

- **Framework documentation:** Read the layer document for the full context, patterns, anti-patterns, and tool recommendations
- **Patterns:** Use the named patterns as architectural blueprints, not starting-from-scratch designs
- **Templates:** Use the intake, business case, and architecture templates to structure the work
- **Key questions:** The "key questions for client engagement" at the end of each layer document are useful for internal discovery sessions — ask them of your own stakeholders

### Step 4 — Reassess to Track Progress

The assessment score is only meaningful as a trend. A single assessment tells you where you are. Reassessment at 6 months and 12 months tells you whether your investments are producing the expected maturity improvement.

If a layer score did not improve after a quarter of investment, the investment either did not address the right gaps or was not fully implemented. The layer document's "what good looks like at each maturity level" section is a useful diagnostic — check whether the specific practices required for the next level are genuinely in place.

---

## Using the Assessment in a CloudKraft Engagement

### Tier 1 — Diagnostic Assessment

The online assessment provides a self-reported baseline. The Tier 1 engagement builds on it with:

- **Discovery sessions** with 4–6 stakeholders per layer (not just the person who completed the online assessment)
- **Technical validation** of the self-reported scores — reviewing actual system configurations, gateway logs, evaluation reports, and policy documents
- **Calibrated scoring** — the CloudKraft assessor challenges scores that cannot be supported with evidence
- **Gap analysis and prioritised roadmap** — structured using the Prioritisation Matrix template and delivered as the Tier 1 output

The Tier 1 assessment typically reveals that self-reported scores are 0.5–1.0 points higher than the evidence-validated scores. This is not dishonesty — it is the standard gap between intended practice and implemented practice.

### Tier 2 — Architecture Blueprint

The Tier 2 engagement uses the validated gap analysis as input to produce:

- Target architecture for each priority use case (using the [Target Architecture template](../templates/target-architecture.md))
- Business cases for the highest-priority use cases (using the [Business Case template](../templates/business-case.md))
- A phased delivery roadmap (using the [Roadmap template](../templates/roadmap.md))

### Tier 3 — Implementation Advisory

Tier 3 engagements use monthly reassessments of the layer scores most relevant to in-progress work to track programme health. If a score does not move as expected, the advisory engagement pivots to understand why.

---

## Limitations of the Assessment

**Self-reported assessments have an optimism bias.** The online assessment is a fast, useful starting point. It is not a substitute for the validated scoring produced in a Tier 1 engagement, which involves reviewing actual evidence rather than relying on self-assessment.

**The score is not the goal.** Improving the score is not the objective — improving the governance practices that the score reflects is. An organisation that optimises for assessment score without implementing the underlying practices is producing a misleading picture without the actual risk reduction.

**The model is calibrated for regulated enterprises in Canada.** The maturity thresholds and the weighting of compliance-related practices reflect the requirements of regulated Canadian industries (financial services, healthcare, insurance, public sector). Organisations in less regulated contexts may find that Level 3 thresholds are higher than their actual requirements.

**The model changes as the AI landscape changes.** What constitutes Level 4 practice in 2026 will differ from what constituted Level 4 practice in 2024. CloudKraft reviews and updates the maturity model annually. Assessment scores from prior years should be interpreted in the context of the model version used.

---

## Take the Assessment

**[Start the AI Governance Maturity Assessment →](https://assessment.cloudkraft.com/assessment/)**

Complete in 5–8 minutes. Receive a scored maturity profile across all five layers with a prioritised gap analysis and recommended next steps.

**Questions?** Contact [faiz@cloudkraft.com](mailto:faiz@cloudkraft.com) or visit [cloudkraft.com](https://cloudkraft.com).

---

*Part of the [CloudKraft AI Control Tower framework](../framework/README.md).*
