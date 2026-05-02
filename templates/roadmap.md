# Template: AI Programme Roadmap

**Purpose:** Structure the phased delivery plan for an enterprise AI programme, with clear milestones, dependencies, resource requirements, and success criteria at each phase.

**When to use:** As the primary delivery planning output of a Tier 2 Architecture Blueprint engagement. Updated quarterly throughout Tier 3 Implementation Advisory.

**Key principle:** A roadmap is a series of commitments, not a list of aspirations. Every item on the roadmap must have an owner, a target date, and a measurable definition of done. Items without these are not roadmap items — they are a backlog.

---

## Programme Overview

| Field | Value |
|---|---|
| **Programme Name** | [e.g., AI Control Tower Implementation] |
| **Programme Sponsor** | [Name, title] |
| **Programme Lead (Technical)** | [Name, title] |
| **Roadmap Version** | [e.g., v1.2] |
| **Roadmap Period** | [e.g., Q2 2026 – Q1 2027] |
| **Last Updated** | [YYYY-MM-DD] |

### Programme Objective

[One paragraph. What does this programme aim to achieve by the end of the roadmap period? Be specific about the business outcomes, not just the technical deliverables. Example: "By Q1 2027, the organisation will have a governed AI platform operating at a minimum maturity level of 3 across all five Control Tower layers, with three use cases in production delivering measurable business value, and a compliant audit trail for all AI activity suitable for regulatory review."]

### Starting Point

**CloudKraft Assessment Score (baseline):**

| Layer | Score | Key Gaps |
|---|---|---|
| Layer 1 — Access & Governance | [1.0–5.0] | [Top 1–2 gaps] |
| Layer 2 — Data & Retrieval | [1.0–5.0] | [Top 1–2 gaps] |
| Layer 3 — Orchestration | [1.0–5.0] | [Top 1–2 gaps] |
| Layer 4 — Evaluation & Trust | [1.0–5.0] | [Top 1–2 gaps] |
| Layer 5 — Integration | [1.0–5.0] | [Top 1–2 gaps] |
| **Overall** | **[1.0–5.0]** | |

**Target Assessment Score (end of roadmap):**

| Layer | Current | Target |
|---|---|---|
| Layer 1 | [X.X] | [X.X] |
| Layer 2 | [X.X] | [X.X] |
| Layer 3 | [X.X] | [X.X] |
| Layer 4 | [X.X] | [X.X] |
| Layer 5 | [X.X] | [X.X] |

---

## Roadmap Summary

*One-page visual summary. Replace with an actual Gantt chart or visual roadmap in the delivered document.*

```
                Q2 2026          Q3 2026          Q4 2026          Q1 2027
                Apr  May  Jun    Jul  Aug  Sep    Oct  Nov  Dec    Jan  Feb  Mar

FOUNDATION
AI Gateway      ████ ████
Vector DB             ████ ████
Eval Framework              ████

USE CASE 1
Build                        ████ ████
Pilot                                  ████ ████
Production                                       ████ ████ ████

USE CASE 2
Build                             ████ ████ ████
Pilot                                            ████ ████
Production                                                    ████ ████

INTEGRATION
ServiceNow                                  ████ ████ ████
SharePoint sync       ████ ████

GOVERNANCE
Policy docs     ████
Model catalogue      ████
Monthly reviews                ████ ████ ████ ████ ████ ████ ████ ████ ████
```

---

## Phase 1 — Foundation

**Target completion:** [Date]
**Budget:** $[X]
**Owner:** [Name]

### Objective

Establish the foundational AI infrastructure that all use cases will run on. A foundation phase is necessary when the organisation does not yet have a centralised gateway, a vector database, or an evaluation framework. Without this infrastructure, each use case would build its own — the BYOLLM pattern.

### Deliverables

| # | Deliverable | Owner | Due Date | Definition of Done |
|---|---|---|---|---|
| 1.1 | AI gateway deployed (LiteLLM or Portkey) | [Name] | [Date] | All developer LLM access routes through the gateway; existing direct keys rotated |
| 1.2 | Team-based API key structure in place | [Name] | [Date] | Each team has a gateway API key; cost dashboard shows per-team attribution |
| 1.3 | Approved model catalogue published | [Name] | [Date] | Catalogue reviewed by security; enforced at the gateway layer |
| 1.4 | Vector database deployed | [Name] | [Date] | Qdrant (or equivalent) running in production environment; ingestion pipeline template available |
| 1.5 | Evaluation framework configured | [Name] | [Date] | Langfuse (or equivalent) deployed; RAGAS integrated; test suite runs against example data |
| 1.6 | AI usage policy published | [Name] | [Date] | Policy reviewed by legal and CISO; communicated to all affected teams |

### Dependencies and Risks

| Item | Type | Mitigation |
|---|---|---|
| [e.g., Security review for gateway tool selection] | Dependency | [e.g., Security review initiated week 1; target 2-week turnaround] |
| [e.g., Data centre region selection for residency] | Decision | [e.g., Legal + CISO to confirm by end of week 2] |

### Phase 1 Success Criteria

- [ ] 100% of developer LLM API traffic routes through the centralised gateway
- [ ] Cost attribution report produced for the first complete month
- [ ] Vector database accessible from the development environment
- [ ] Evaluation framework runs a complete RAGAS evaluation without errors

---

## Phase 2 — Use Case [Name]: Build and Pilot

**Target completion:** [Date]
**Budget:** $[X]
**Owner:** [Name]

### Objective

Build and pilot the highest-priority use case identified in the Prioritisation Matrix. The pilot phase should involve a defined subset of users in a real production environment — not a demo environment — so that quality issues, edge cases, and integration problems are discovered before full rollout.

### Deliverables

| # | Deliverable | Owner | Due Date | Definition of Done |
|---|---|---|---|---|
| 2.1 | Golden dataset built (100+ examples) | [Name] | [Date] | Dataset reviewed by business stakeholders; includes adversarial examples |
| 2.2 | RAG pipeline developed and tested | [Name] | [Date] | RAGAS faithfulness > 0.85, relevance > 0.80 on golden dataset |
| 2.3 | User interface built | [Name] | [Date] | Pilot users can access the system; UI shows citations and confidence indicators |
| 2.4 | Integration with [enterprise system] | [Name] | [Date] | AI outputs written to [system] with idempotency and audit logging |
| 2.5 | Human-in-the-loop review queue | [Name] | [Date] | Low-confidence outputs route to review queue with defined SLA |
| 2.6 | Pilot launched with [N] users | [Name] | [Date] | Pilot users onboarded; feedback mechanism live; metrics baseline established |
| 2.7 | Pilot evaluation report | [Name] | [Date] | 4-week pilot data reviewed; go/no-go decision for full rollout |

### Phase 2 Go / No-Go Criteria (pilot → full rollout)

| Criterion | Target | Actual | Pass / Fail |
|---|---|---|---|
| Answer faithfulness | > 0.85 | [TBD] | |
| User satisfaction (pilot survey) | > 3.5 / 5.0 | [TBD] | |
| Escalation rate | < 15% | [TBD] | |
| System availability | > 99% over pilot period | [TBD] | |
| No P1 security findings unresolved | 0 open P1s | [TBD] | |

---

## Phase 3 — Use Case [Name]: Full Deployment

**Target completion:** [Date]
**Budget:** $[X]
**Owner:** [Name]

### Deliverables

| # | Deliverable | Owner | Due Date | Definition of Done |
|---|---|---|---|---|
| 3.1 | Full user rollout complete | [Name] | [Date] | All [N] target users onboarded; training delivered |
| 3.2 | Production monitoring live | [Name] | [Date] | Weekly evaluation reports automated; alerting configured |
| 3.3 | Benefits measurement baseline | [Name] | [Date] | Pre-intervention metrics captured for comparison at 6-month review |
| 3.4 | Operational runbook published | [Name] | [Date] | On-call team can diagnose and respond to system incidents |

---

## Phase 4 — Use Case [Name 2] (if applicable)

*Repeat the phase structure for each additional use case. Use case 2 should begin build during the pilot phase of use case 1 to maintain programme momentum.*

---

## Programme Governance

### Decision Log

| # | Decision | Made By | Date | Rationale |
|---|---|---|---|---|
| [1] | [e.g., LiteLLM selected as gateway over Portkey] | [Name] | [Date] | [e.g., Data residency requirement; Portkey hosted plan does not guarantee Canadian data residency] |
| [2] | | | | |

### Monthly Review Cadence

| Review | Frequency | Participants | Agenda |
|---|---|---|---|
| Programme status | Monthly | Sponsor, Programme Lead, Tech Lead | Milestone status, risks, decisions required |
| Technical review | Bi-weekly | Engineering team + architects | Build progress, blockers, architecture decisions |
| Steering committee | Quarterly | Sponsor, CFO rep, CISO, Head of AI | Financial tracking, risk review, roadmap adjustments |

### Change Control

*Changes to the roadmap that affect scope, timeline, or budget by more than [10%] require written approval from [the Programme Sponsor]. Changes within these thresholds can be approved by the Programme Lead.*

---

## Risk Register

| # | Risk | Likelihood | Impact | Owner | Mitigation | Status |
|---|---|---|---|---|---|---|
| R1 | [e.g., Key engineer leaves during build phase] | M | H | [Name] | [e.g., Knowledge documentation required; bus factor >1 for all critical components] | Open |
| R2 | [e.g., LLM provider outage during pilot] | L | H | [Name] | [e.g., Multi-provider gateway with automatic failover] | Mitigated |
| R3 | [e.g., Data access to legacy system blocked] | M | M | [Name] | [e.g., Confirmed access as programme dependency; escalation path to CIO] | Open |
| R4 | [e.g., Regulatory guidance on AI in [sector] changes] | L | H | [Name] | [e.g., Legal monitoring; modular architecture for quick compliance adjustments] | Monitoring |

---

## Resource Plan

| Role | Allocation | Phase 1 | Phase 2 | Phase 3 | Phase 4 | Notes |
|---|---|---|---|---|---|---|
| AI/ML Engineer | [FTE] | 0.5 | 1.0 | 0.5 | 1.0 | [Internal / CloudKraft / Contract] |
| Data Engineer | [FTE] | 0.5 | 0.5 | 0.25 | 0.25 | |
| Frontend Engineer | [FTE] | 0 | 0.5 | 0.25 | 0.25 | |
| Security Architect | [FTE] | 0.25 | 0.25 | 0.1 | 0.1 | |
| Programme Manager | [FTE] | 0.25 | 0.5 | 0.5 | 0.5 | |
| Business Analyst | [FTE] | 0.25 | 0.5 | 0.25 | 0.25 | |

---

## Budget Tracker

| Phase | Approved Budget | Actual / Forecast | Variance | Notes |
|---|---|---|---|---|
| Phase 1 — Foundation | $[X] | $[X] | $[X] | |
| Phase 2 — Use Case 1 Build | $[X] | $[X] | $[X] | |
| Phase 3 — Use Case 1 Full Deployment | $[X] | $[X] | $[X] | |
| Phase 4 — Use Case 2 | $[X] | $[X] | $[X] | |
| **Total** | **$[X]** | **$[X]** | **$[X]** | |

*Infrastructure run costs (LLM API, hosting) are tracked separately in the FinOps dashboard.*

---

*Template version 1.0. Part of the [CloudKraft AI Control Tower framework](../framework/README.md).*
