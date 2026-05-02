# Template: Prioritisation Matrix

**Purpose:** Rank a portfolio of AI use cases to determine which to build first, using a consistent scoring methodology that balances business value against delivery risk.

**When to use:** After completing Use Case Intake documents for a set of candidate use cases. Typically produced as the primary output of a Tier 1 Diagnostic Assessment.

**Who should complete it:** The scoring should be completed with input from both business stakeholders (value scores) and technical stakeholders (feasibility and risk scores). Do not let either group score alone — business stakeholders underestimate technical risk, technical stakeholders underestimate business value.

---

## Scoring Methodology

Each use case is scored on two dimensions:

**Business Value (1–5):** The strategic and financial impact of the use case if successfully delivered.

**Delivery Feasibility (1–5):** The ease and speed with which the use case can be delivered given current organisational readiness.

The product of these two scores (1–25) gives the Priority Score. Use cases with high value AND high feasibility score highest. This deliberately avoids the common mistake of prioritising only by value — a high-value, low-feasibility use case that takes 18 months to deliver provides no value in the near term and consumes the capacity needed for faster wins.

---

## Business Value Scoring Criteria

| Score | Label | Criteria |
|---|---|---|
| 5 | Transformative | Measurable impact on revenue, cost, or risk at enterprise scale. >$1M annual benefit or equivalent risk reduction. Critical to strategic priorities. |
| 4 | High | Significant impact on a specific business unit. $250K–$1M annual benefit or equivalent. Directly addresses a top-3 business priority. |
| 3 | Moderate | Meaningful productivity improvement for a team or function. $50K–$250K annual benefit. Addresses a documented pain point. |
| 2 | Low | Nice-to-have improvement. <$50K annual benefit. Does not address a stated priority. |
| 1 | Negligible | Minimal discernible business impact. Primarily experimental or technology demonstration. |

**Value scoring factors (weight each 1–5 and average for the final Value score):**

| Factor | Weight | Score (1–5) | Weighted Score |
|---|---|---|---|
| Financial impact (cost reduction, revenue, avoidance) | 35% | | |
| Strategic alignment (connection to stated org priorities) | 25% | | |
| User reach (number of employees or customers affected) | 20% | | |
| Time-sensitivity (is the opportunity time-limited?) | 10% | | |
| Executive visibility and sponsorship strength | 10% | | |
| **Business Value Score** | | | **[Weighted average]** |

---

## Delivery Feasibility Scoring Criteria

| Score | Label | Criteria |
|---|---|---|
| 5 | Ready to Build | Data accessible and classified, technical team ready, stakeholders aligned, no known blockers. Could start in 2 weeks. |
| 4 | Nearly Ready | One or two minor blockers (data access confirmation pending, model selection in progress). Could start in 4–6 weeks. |
| 3 | Requires Preparation | Several prerequisites to complete before build can start (data ingestion pipeline, security review, team upskilling). Start in 2–3 months. |
| 2 | Significant Gaps | Material technical or organisational gaps. Requires foundational work (e.g., Layer 1 governance not in place) before this use case can proceed. |
| 1 | Not Feasible Now | Fundamental blockers — missing data, missing technical capability, unresolved compliance barriers, or lack of sponsor commitment. |

**Feasibility scoring factors:**

| Factor | Weight | Score (1–5) | Weighted Score |
|---|---|---|---|
| Data readiness (accessible, classified, quality) | 30% | | |
| Technical platform readiness (gateway, vector DB, orchestration) | 25% | | |
| Team capability (relevant AI/ML skills available) | 20% | | |
| Organisational readiness (stakeholder alignment, change management) | 15% | | |
| Regulatory clarity (no unresolved compliance blockers) | 10% | | |
| **Delivery Feasibility Score** | | | **[Weighted average]** |

---

## Priority Score Calculation

```
Priority Score = Business Value Score × Delivery Feasibility Score
Range: 1 (low value, low feasibility) to 25 (high value, high feasibility)
```

**Interpretation:**

| Priority Score | Interpretation | Recommended Action |
|---|---|---|
| 20–25 | **Tier 1 — Build First** | Start immediately. These are your quick wins and strategic movers. |
| 15–19 | **Tier 2 — Build Next** | Plan for the next delivery cycle. Address any feasibility gaps in parallel. |
| 10–14 | **Tier 3 — Build Later** | Wait until higher-priority use cases are in flight. Reassess in 3–6 months. |
| 5–9 | **Tier 4 — Needs Foundation** | These use cases are blocked by foundational gaps (typically Layer 1 governance). Address the foundation first. |
| 1–4 | **Tier 5 — Defer or Drop** | Low value and/or not feasible. Either drop from the roadmap or re-evaluate when conditions change. |

---

## Portfolio Scoring Summary

*Complete one row per use case. Sort by Priority Score descending.*

| Use Case | Sponsor | Value Score | Feasibility Score | Priority Score | Tier | Notes |
|---|---|---|---|---|---|---|
| [Use case name] | [Name] | [1–5] | [1–5] | [1–25] | [1–5] | [Key blocker or consideration] |
| | | | | | | |
| | | | | | | |
| | | | | | | |
| | | | | | | |

---

## Portfolio Analysis

### Distribution Summary

| Tier | Count | % of Portfolio |
|---|---|---|
| Tier 1 — Build First | | |
| Tier 2 — Build Next | | |
| Tier 3 — Build Later | | |
| Tier 4 — Needs Foundation | | |
| Tier 5 — Defer or Drop | | |
| **Total** | | 100% |

### Foundational Dependencies

*List any foundational capabilities (e.g., Layer 1 gateway deployment, RAG infrastructure, evaluation framework) that are blocking multiple use cases. These should be treated as programme dependencies rather than individual use case prerequisites.*

| Foundational Capability | Blocked Use Cases | Priority | Owner |
|---|---|---|---|
| [e.g., Centralised AI gateway] | [List use case names] | [H/M/L] | [Team/person] |
| [e.g., Vector database deployment] | [List use case names] | [H/M/L] | [Team/person] |

### Key Risks to the Portfolio

*What are the 2–3 risks that could affect multiple use cases or the overall programme priority?*

1. [Risk description — example: "If data residency requirements for customer data are interpreted strictly, three Tier 1 use cases may require reclassification to Tier 4 pending a compliant LLM hosting solution."]
2. [Risk description]
3. [Risk description]

---

## Recommended Roadmap Sequence

*Based on the Priority Scores and foundational dependencies, the recommended build sequence is:*

**Quarter 1:**
[List Tier 1 use cases, noting any that share infrastructure that should be built once]

**Quarter 2:**
[List Tier 2 use cases that will be unblocked after Q1 delivery and foundation work]

**Quarter 3–4:**
[List Tier 3 use cases and any Tier 4 use cases that will be unblocked by foundational work completed in Q1–Q2]

---

## Prioritisation Decision Log

*Record any significant decisions made during the prioritisation discussion that are not captured by the scoring alone.*

| Decision | Rationale | Date | Participants |
|---|---|---|---|
| [e.g., Scored Use Case X higher than its raw score suggests because of executive commitment] | [Rationale] | [Date] | [Names] |
| | | | |

---

*Template version 1.0. Part of the [CloudKraft AI Control Tower framework](../framework/README.md).*
