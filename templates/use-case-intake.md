# Template: Use Case Intake

**Purpose:** Evaluate and document a proposed AI use case with enough detail to make a go/no-go decision, prioritise it relative to other use cases, and begin technical scoping.

**When to use:** Every AI use case that progresses beyond informal discussion. One intake document per use case.

**Completion time:** 60–90 minutes with the use case sponsor and at least one technical stakeholder.

---

## 1. Use Case Identity

| Field | Value |
|---|---|
| **Use Case Name** | [A short, descriptive name — not an acronym or project code] |
| **Sponsor** | [Name, title, business unit] |
| **Technical Owner** | [Name, title, team — the person responsible for build and operations] |
| **Date of Intake** | [YYYY-MM-DD] |
| **Status** | [Proposed / In Scoping / Approved / In Development / In Production / Paused / Cancelled] |
| **Control Tower Layer(s)** | [Which of the five layers does this use case primarily address? e.g., Layer 2 — Data & Retrieval] |

---

## 2. Problem Statement

*Describe the business problem this use case solves. Be specific about who is affected, how often, and what the current workaround is. Do not describe the solution here.*

**The problem:**
[Describe the problem in 2–4 sentences. Example: "Customer support analysts spend an average of 45 minutes per complex case searching across six internal knowledge bases to find relevant policy and product documentation. This search time delays case resolution, reduces analyst capacity, and degrades customer satisfaction scores."]

**Who is affected:**
[Name the specific role or team. Example: "35 customer support analysts across three service centres"]

**How often:**
[Frequency. Example: "Approximately 120 complex cases per day — roughly 30% of total case volume"]

**Current workaround:**
[What do people do today? Example: "Manual search across SharePoint, Confluence, and a legacy wiki. Senior analysts maintain personal bookmarks and share them informally."]

**Evidence of the problem:**
[Data, measurements, or stakeholder testimony. Example: "Average handle time for complex cases is 68 minutes vs. 22 minutes for standard cases. Analyst survey shows 78% cite knowledge search as their top time sink."]

---

## 3. Proposed Solution

**Solution description:**
[Describe the proposed AI solution in plain language. Example: "An AI-powered knowledge assistant that allows analysts to ask questions in natural language and receive answers drawn from all relevant internal knowledge bases, with citations to source documents."]

**AI capability required:**
[Select the primary capability: RAG / Classification / Extraction / Generation / Summarisation / Code generation / Image analysis / Other]

**Does a comparable solution exist commercially?**
[Yes/No. If yes, name the tool and explain why a custom build is preferred or why the commercial tool was evaluated and rejected.]

---

## 4. Business Value

### Expected Benefits

*For each benefit claimed, state a specific metric and a realistic target. Vague benefits ("improve efficiency", "enhance customer experience") cannot be measured or validated.*

| Benefit Category | Current State | Target State | Measurement Method |
|---|---|---|---|
| Time savings | [e.g., 45 min/case search time] | [e.g., 10 min/case] | [e.g., AHT tracking in CRM] |
| Volume / throughput | [e.g., 120 complex cases/day] | [e.g., 160 complex cases/day] | [e.g., case volume reports] |
| Quality / accuracy | [e.g., 12% case reopening rate] | [e.g., 7% case reopening rate] | [e.g., CRM case status] |
| Cost avoidance | [e.g., $X in annual analyst overtime] | [e.g., Eliminate overtime for knowledge search] | [e.g., Payroll data] |
| Revenue impact | [e.g., N/A or specific claim] | [e.g., $X annual] | [e.g., Sales pipeline data] |

### Value Realisation Timeline

*When will each benefit be measurable? Many AI use cases show limited benefit in the first 3 months and accelerate as adoption grows.*

| Milestone | Timeline | Expected State |
|---|---|---|
| Pilot deployment (limited users) | [e.g., Month 3] | [e.g., 10 analysts using the tool] |
| Full rollout | [e.g., Month 6] | [e.g., All 35 analysts on the tool] |
| Benefit measurement point | [e.g., Month 9] | [e.g., AHT data sufficient for comparison] |

---

## 5. Feasibility Assessment

### Data Readiness

| Question | Answer | Risk Level |
|---|---|---|
| What data sources will ground the AI system? | [List specific systems: SharePoint, Confluence, database name, etc.] | [H/M/L] |
| Are these sources accessible via API or export? | [Yes/No/Partial — detail which are not accessible] | [H/M/L] |
| Are the sources current and maintained? | [Yes/No — note any known staleness issues] | [H/M/L] |
| What is the data classification of the input data? | [Public / Internal / Sensitive / Restricted] | [H/M/L] |
| Are there access control requirements on the data? | [Yes/No — if yes, describe: role-based, team-based, etc.] | [H/M/L] |
| How frequently is the data updated? | [Daily/Weekly/Monthly/Ad hoc] | [H/M/L] |

### Technical Readiness

| Question | Answer | Risk Level |
|---|---|---|
| Does the organisation have a centralised AI gateway? | [Yes/No/In progress] | [H/M/L] |
| Does the organisation have a vector database available? | [Yes/No — if yes, which one?] | [H/M/L] |
| What orchestration framework is in use? | [LangGraph/LangChain/Custom/None] | [H/M/L] |
| Is there an evaluation framework in place? | [Yes/No — if yes, which metrics?] | [H/M/L] |
| Does the team have AI/ML development capability? | [Yes/No — note headcount and experience level] | [H/M/L] |

### Organisational Readiness

| Question | Answer | Risk Level |
|---|---|---|
| Does the sponsor have budget authority? | [Yes/No — if no, who does and are they aligned?] | [H/M/L] |
| Is there stakeholder resistance to AI in this domain? | [Yes/No — if yes, describe the concern] | [H/M/L] |
| Are affected employees aware of and supportive of this use case? | [Yes/No/Unknown] | [H/M/L] |
| Are there change management requirements? | [Training, role definition changes, process documentation needed?] | [H/M/L] |

---

## 6. Risk Assessment

### Key Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| [e.g., AI produces incorrect policy answers that analysts act on] | [H/M/L] | [H/M/L] | [e.g., Human-in-the-loop review for high-stakes responses, citation-required output format] |
| [e.g., Data freshness: outdated policies returned as current] | [H/M/L] | [H/M/L] | [e.g., Daily ingestion pipeline, staleness warning in response] |
| [e.g., Analyst over-reliance on AI without independent judgment] | [H/M/L] | [H/M/L] | [e.g., Training programme, confidence indicators in UI] |
| [e.g., Data residency: customer data sent to US-based LLM] | [H/M/L] | [H/M/L] | [e.g., Azure OpenAI in canadacentral region] |

### Regulatory and Compliance Considerations

[Note any relevant regulatory obligations: PIPEDA for personal information, OSFI B-13 for financial services AI, sector-specific data localisation requirements, model explainability requirements for regulated decisions.]

---

## 7. Initial Cost Estimate

*This is a rough order-of-magnitude estimate only. A detailed estimate requires technical scoping.*

| Cost Component | Monthly Estimate | Basis |
|---|---|---|
| LLM API costs | $[X] | [e.g., 120 queries/day × $0.02/query × 30 days] |
| Embedding / vector DB | $[X] | [e.g., Qdrant hosted or self-hosted estimate] |
| Development (one-time) | $[X] | [e.g., Estimated person-months × blended rate] |
| Ongoing maintenance | $[X/month] | [e.g., 0.25 FTE estimated] |
| **Total first-year cost** | $[X] | |

---

## 8. Recommendation

**Go / No-Go / Needs More Information**

*Circle one and provide a brief rationale.*

**Rationale:**
[2–4 sentences explaining the recommendation. Reference the most important factors from the feasibility assessment and risk assessment.]

**Conditions (if any):**
[If Go with conditions, list the conditions. Example: "Proceed to scoping on condition that data access to the ServiceNow knowledge base is confirmed within two weeks."]

**Priority relative to other use cases:**
[High / Medium / Low — for use in the Prioritisation Matrix]

---

## 9. Next Steps

| Action | Owner | Due Date |
|---|---|---|
| [e.g., Confirm data access to SharePoint and Confluence] | [Name] | [Date] |
| [e.g., Technical scoping session with platform team] | [Name] | [Date] |
| [e.g., Security review initiation for LLM provider selection] | [Name] | [Date] |

---

*Template version 1.0. Part of the [CloudKraft AI Control Tower framework](../framework/README.md).*
