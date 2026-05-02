# Template: Business Case

**Purpose:** Build the financial and strategic justification for an AI investment, structured for presentation to executive sponsors, finance, and the board.

**When to use:** After a use case or portfolio of use cases has been prioritised and before investment approval is sought. Also used to establish the benefit baseline that the programme will be measured against.

**Key principle:** A business case that cannot be measured is not a business case. Every benefit claim must have a named metric, a current baseline, a target, and a measurement method. If you cannot measure it, do not include it.

---

## Executive Summary

**Programme / Use Case Name:**
[Name]

**Investment Requested:**
$[X] [one-time / over [N] years]

**Expected Return:**
$[X] net benefit over [N] years

**Payback Period:**
[X] months

**Strategic Rationale (2–3 sentences):**
[Why this investment, why now, what happens if we do not make it]

**Recommendation:**
[Approve / Approve with conditions / Do not approve]

---

## 1. Strategic Context

### Alignment to Organisational Priorities

*Show explicitly how this investment connects to the organisation's stated strategic priorities. Do not assume the reader will make this connection themselves.*

| Organisational Priority | How This Investment Addresses It |
|---|---|
| [e.g., Reduce operational cost by 15% by 2026] | [e.g., Estimated $420K annual savings from analyst time reduction] |
| [e.g., Improve customer satisfaction to NPS 45+] | [e.g., Faster case resolution drives expected 3-point NPS improvement] |

### What Happens If We Do Not Invest

*The most underused section of most business cases. Name the specific risk or cost of inaction.*

[Example: "Competitors in our market segment have already deployed AI-assisted knowledge management. Without this investment, our analysts will remain 45 minutes slower per complex case than peers using AI tools — a structural productivity disadvantage that compounds over time. In addition, current manual search processes are the primary source of employee satisfaction complaints in annual surveys."]

---

## 2. Solution Description

### What Is Being Built

[Plain-language description of the AI system, suitable for a non-technical executive audience. Maximum one paragraph.]

### Scope

**In scope:**
- [Specific capability 1]
- [Specific capability 2]

**Out of scope (explicitly):**
- [What this investment will not address — important for managing expectations]

### Technology Approach

*One paragraph for an executive audience. Do not require the reader to understand LangGraph or RAGAS.*

[Example: "The solution uses a retrieval-augmented generation (RAG) architecture: the AI system searches the organisation's knowledge bases to find relevant documents, then uses a large language model to synthesise an answer with citations to the source documents. This approach grounds AI responses in company-specific knowledge rather than general internet training data, significantly reducing the risk of incorrect or invented answers."]

### Build vs. Buy Assessment

*Was a commercial alternative evaluated? Why was a custom build (or a specific commercial tool) chosen?*

[Document the evaluation, even if brief. Example: "Three commercial AI knowledge management tools were evaluated (Tool A, Tool B, Tool C). Tool A was rejected due to US-only data residency. Tool B was rejected due to inability to integrate with the legacy wiki system. Tool C was considered as a final candidate but was $180K/year more expensive than the custom build at expected scale and provided less flexibility for the access control requirements."]

---

## 3. Financial Analysis

### Investment Required

| Cost Component | Year 1 | Year 2 | Year 3 | Total |
|---|---|---|---|---|
| **Development (one-time)** | | | | |
| External development (CloudKraft or vendor) | $[X] | — | — | $[X] |
| Internal engineering time | $[X] | — | — | $[X] |
| **Infrastructure (ongoing)** | | | | |
| LLM API costs | $[X] | $[X] | $[X] | $[X] |
| Vector database hosting | $[X] | $[X] | $[X] | $[X] |
| Other cloud infrastructure | $[X] | $[X] | $[X] | $[X] |
| **Operations (ongoing)** | | | | |
| Maintenance and support | $[X] | $[X] | $[X] | $[X] |
| Evaluation and monitoring | $[X] | $[X] | $[X] | $[X] |
| **Total Investment** | **$[X]** | **$[X]** | **$[X]** | **$[X]** |

*LLM API cost assumptions: [Document the assumptions — model, expected query volume, average tokens per query, cost per token. These will be revisited at programme review.]*

### Expected Benefits

*State each benefit as a specific, measurable value. Note whether the benefit is quantified (hard) or estimated (soft).*

| Benefit | Year 1 | Year 2 | Year 3 | Total | Confidence |
|---|---|---|---|---|---|
| [e.g., Analyst time savings] | $[X] | $[X] | $[X] | $[X] | [High/Med/Low] |
| [e.g., Overtime reduction] | $[X] | $[X] | $[X] | $[X] | [High/Med/Low] |
| [e.g., Reduced case reopening cost] | $[X] | $[X] | $[X] | $[X] | [High/Med/Low] |
| [e.g., Avoided FTE headcount growth] | $[X] | $[X] | $[X] | $[X] | [High/Med/Low] |
| **Total Benefits** | **$[X]** | **$[X]** | **$[X]** | **$[X]** | |

*Benefit calculation methodology: [Document how each figure was calculated. For time savings: [current time per task] × [expected reduction %] × [number of users] × [fully-loaded FTE cost]. Show your work.]*

### Net Benefit and ROI

| Metric | Value |
|---|---|
| Total 3-year investment | $[X] |
| Total 3-year benefit | $[X] |
| Net 3-year benefit | $[X] |
| ROI | [X]% |
| Payback period | [X] months |
| Net Present Value (at [X]% discount rate) | $[X] |

### Sensitivity Analysis

*What happens to the business case if key assumptions prove wrong?*

| Scenario | Investment | Benefit | NPV | Payback |
|---|---|---|---|---|
| Base case | $[X] | $[X] | $[X] | [X] months |
| Downside: 30% lower adoption | $[X] | $[X] | $[X] | [X] months |
| Upside: 20% higher query volume | $[X] | $[X] | $[X] | [X] months |
| Worst case: Build cost 50% over | $[X] | $[X] | $[X] | [X] months |

*The business case is viable in all scenarios above if [state the condition — e.g., "NPV remains positive even in the worst-case scenario"].* 

---

## 4. Risk Assessment

| Risk | Likelihood | Impact | Mitigation | Residual Risk |
|---|---|---|---|---|
| AI quality insufficient for production use | [H/M/L] | [H/M/L] | [e.g., Formal evaluation gate before launch, human-in-the-loop for low-confidence responses] | [H/M/L] |
| Data access delays extend timeline | [H/M/L] | [H/M/L] | [e.g., Data access confirmed as a programme prerequisite] | [H/M/L] |
| User adoption lower than expected | [H/M/L] | [H/M/L] | [e.g., Change management programme, adoption target in KPIs] | [H/M/L] |
| LLM provider pricing increase | [H/M/L] | [H/M/L] | [e.g., Multi-provider gateway enables switching; 50% pricing increase modelled in sensitivity] | [H/M/L] |
| Regulatory change affects permissible AI use | [H/M/L] | [H/M/L] | [e.g., OSFI/regulatory monitoring, modular architecture] | [H/M/L] |

---

## 5. Delivery Plan

### Phases

| Phase | Description | Duration | Cost | Key Milestone |
|---|---|---|---|---|
| Phase 1 — Foundation | [e.g., Data pipeline, gateway deployment] | [X weeks] | $[X] | [e.g., Data accessible in vector DB] |
| Phase 2 — Build | [e.g., RAG system, review UI] | [X weeks] | $[X] | [e.g., Internal pilot with 5 users] |
| Phase 3 — Pilot | [e.g., Limited rollout, evaluation, tuning] | [X weeks] | $[X] | [e.g., Quality gate passed, NPS measured] |
| Phase 4 — Full Deployment | [e.g., All users, monitoring live] | [X weeks] | $[X] | [e.g., 100% adoption, benefits tracking active] |

### Go / No-Go Gates

*Define the measurable conditions that must be met before each phase proceeds.*

| Gate | Condition | Measurement |
|---|---|---|
| Phase 1 → Phase 2 | [e.g., All data sources accessible with correct access controls] | [e.g., Security sign-off document] |
| Phase 2 → Phase 3 | [e.g., Answer faithfulness > 0.85 on evaluation golden dataset] | [e.g., RAGAS evaluation report] |
| Phase 3 → Phase 4 | [e.g., Pilot NPS > 30 and escalation rate < 10%] | [e.g., User survey, system metrics] |

---

## 6. Success Metrics and Measurement Plan

*These are the metrics that will be reported to the programme sponsor throughout delivery and used to validate business case assumptions at the post-implementation review.*

| Metric | Baseline | Target | Measurement Method | Review Frequency |
|---|---|---|---|---|
| [e.g., Average case handle time for complex cases] | [e.g., 68 minutes] | [e.g., 30 minutes] | [e.g., CRM AHT report] | Monthly |
| [e.g., Case reopening rate] | [e.g., 12%] | [e.g., 7%] | [e.g., CRM case status] | Monthly |
| [e.g., AI answer faithfulness] | [e.g., Establish at launch] | [e.g., > 0.85] | [e.g., RAGAS evaluation] | Weekly |
| [e.g., User adoption rate] | [e.g., 0%] | [e.g., 85% of target users active weekly] | [e.g., System usage logs] | Monthly |

**Post-Implementation Review Date:** [Date — typically 6 months after full deployment]

---

## 7. Approvals

| Role | Name | Decision | Date |
|---|---|---|---|
| Executive Sponsor | | Approve / Reject | |
| Finance | | Approve / Reject | |
| CISO / Security | | Approve / Reject | |
| Architecture Review | | Approve / Reject | |

---

*Template version 1.0. Part of the [CloudKraft AI Control Tower framework](../framework/README.md).*
