# Template: Target Architecture

**Purpose:** Document the recommended technical architecture for an AI capability in enough detail to guide implementation, inform vendor and tool selection, and serve as the architectural baseline for ongoing governance.

**When to use:** During Tier 2 Architecture Blueprint engagements, after use cases are prioritised and investment is approved. One Target Architecture document per major capability or cluster of tightly related capabilities.

**Audience:** Platform engineers, data architects, enterprise architects, security, and technical programme leads. This is a technical document — write for practitioners, not executives.

---

## 1. Architecture Overview

### Capability Being Addressed

**Capability name:** [Short name matching the Use Case Intake]

**Business context:** [One paragraph. What does this system do, for whom, and why? Reference the approved business case.]

**Control Tower layers addressed:** [Which of the five layers does this architecture primarily address?]

**Architecture style:** [RAG / Agentic workflow / Classifier / Extraction pipeline / Orchestration bridge / Other]

### Current State (As-Is)

[Describe the current state in enough detail that a reader can understand what is changing. Include current tools, processes, and pain points. A one-paragraph description plus a simple diagram or component list is sufficient.]

**Current state components:**
- [Component 1: e.g., SharePoint knowledge base, manual search]
- [Component 2: e.g., Email-based escalation for complex queries]

**Current state limitations:**
- [Pain point 1]
- [Pain point 2]

### Target State (To-Be)

[High-level description of the target architecture. Followed by the detailed component-by-component description in subsequent sections.]

---

## 2. Architecture Diagram

*Insert a system context diagram here. At minimum, show:*
- *User-facing interfaces (who or what initiates requests)*
- *Core AI components (gateway, orchestration, retrieval, generation)*
- *Enterprise system integrations (what the AI system reads from and writes to)*
- *Data flows (arrows showing direction and nature of data movement)*
- *Trust boundaries (what is inside the enterprise network boundary, what is external)*

```
[Diagram placeholder — replace with architecture diagram]

Example component layout:

┌─────────────────────────────────────────────────────────────────┐
│  Enterprise Boundary                                            │
│                                                                 │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────────┐   │
│  │  User    │────►│  AI Gateway  │────►│  Orchestration   │   │
│  │  (Web UI)│     │  (LiteLLM)   │     │  (LangGraph)     │   │
│  └──────────┘     └──────────────┘     └────────┬─────────┘   │
│                         │                        │             │
│                   Audit Logs                ┌────▼────────┐    │
│                         │                  │  RAG Layer  │    │
│                   ┌─────▼──────┐           │  (Qdrant +  │    │
│                   │  Langfuse  │           │  BM25)      │    │
│                   │  (Observ.) │           └────────────-┘    │
│                   └────────────┘                │             │
│                                           ┌─────▼──────────┐  │
│                                           │  Knowledge Base │  │
│                                           │  (SharePoint,  │  │
│                                           │  Confluence)   │  │
│                                           └────────────────┘  │
└──────────────────────────────────┬──────────────────────────── ┘
                                   │ (External API calls only)
                               ┌───▼───────────────────┐
                               │  LLM Provider         │
                               │  (Azure OpenAI,       │
                               │  canadacentral)       │
                               └───────────────────────┘
```

---

## 3. Component Specifications

### 3.1 AI Gateway

| Attribute | Value |
|---|---|
| **Tool** | [e.g., LiteLLM self-hosted / Portkey hosted] |
| **Hosting** | [e.g., Azure AKS in canadacentral / Self-hosted VM] |
| **Authentication** | [e.g., Virtual API keys scoped to team+environment] |
| **Approved models** | [List the models in the allowlist for this use case] |
| **Rate limits** | [e.g., 100 RPM per team, 1M tokens/day per use case] |
| **Budget alerts** | [e.g., Alert at 70% and 90% of monthly budget] |
| **Audit logging** | [e.g., Structured JSON to Azure Blob Storage, 90-day retention] |

**Design decisions:**
[Note any significant decisions made in the gateway configuration and their rationale. Example: "LiteLLM self-hosted chosen over Portkey hosted due to OSFI requirement that all request logs remain within Canadian data centres."]

---

### 3.2 Data & Retrieval Layer

| Attribute | Value |
|---|---|
| **Retrieval pattern** | [Hybrid RAG / Vector-only / Keyword-only / Structured / Hybrid] |
| **Vector database** | [Tool, version, hosting] |
| **Keyword index** | [Tool, version, hosting — or "native hybrid" if vector DB supports it] |
| **Embedding model** | [Model name, version, hosting] |
| **Re-ranker** | [Model name, version, hosting — or "none" if not used] |
| **Chunk size** | [Tokens, with overlap] |
| **Top-K retrieval** | [e.g., Top 20 per retrieval path, top 5 after re-ranking] |

**Data sources:**

| Source | Type | Classification | Access Method | Ingestion Frequency | Owner |
|---|---|---|---|---|---|
| [e.g., SharePoint intranet] | Unstructured | Internal | [e.g., Graph API read] | Daily | [Team] |
| [e.g., Policy database] | Structured | Sensitive | [e.g., Read replica] | On-change | [Team] |

**Access control at the retrieval layer:**
[Describe how document-level access control is enforced. Example: "Each document chunk carries an `access_tier` metadata field (public / internal / sensitive / restricted). The retrieval pipeline filters by the requesting user's access tier before returning results. Sensitive and restricted documents require an additional `team_id` filter."]

**Ingestion pipeline:**
[Describe the ingestion process: trigger (scheduled / event-driven), transformation steps (extract, clean, chunk, embed), and error handling for failed ingestion.]

---

### 3.3 Orchestration Layer

| Attribute | Value |
|---|---|
| **Framework** | [e.g., LangGraph 0.2.x] |
| **Workflow type** | [Sequential pipeline / Supervisor-worker / Event-triggered / Other] |
| **Maximum iterations** | [e.g., 5 — hard limit on any agent loop] |
| **Token budget** | [e.g., 8,000 tokens maximum context per query] |
| **Timeout** | [e.g., 30 seconds total workflow timeout] |
| **Fallback behaviour** | [e.g., Return a structured error with escalation URL] |
| **Human escalation trigger** | [e.g., Confidence < 0.7 or document classification = Restricted] |

**Workflow steps:**
1. [Step 1: e.g., Query classification — determine intent and retrieve relevant access tier]
2. [Step 2: e.g., Hybrid retrieval — dense + sparse search, RRF fusion, re-ranking]
3. [Step 3: e.g., Context assembly — format top-5 chunks with citations]
4. [Step 4: e.g., Response generation — LLM call with citation-enforcing prompt]
5. [Step 5: e.g., Confidence scoring — evaluate response against faithfulness threshold]
6. [Step 6: e.g., Route to review queue if confidence < threshold; else return to user]

---

### 3.4 Evaluation & Observability

| Attribute | Value |
|---|---|
| **Observability platform** | [e.g., Langfuse self-hosted] |
| **Evaluation framework** | [e.g., RAGAS] |
| **Pre-deployment gate** | [Yes/No — if yes, metrics and thresholds] |
| **Production evaluation schedule** | [e.g., Weekly on 10% sample] |
| **Alerting thresholds** | [e.g., Alert if faithfulness drops >5pp from 30-day baseline] |
| **Human feedback mechanism** | [e.g., Thumbs up/down in UI, captured to Langfuse] |

**Golden dataset:**
[Document the plan for building and maintaining the golden dataset. Number of examples, how they were labelled, who owns maintenance.]

---

### 3.5 Integration Layer

**Enterprise system integrations:**

| System | Direction | Method | Authentication | Data Classification | Notes |
|---|---|---|---|---|---|
| [e.g., ServiceNow] | Write (AI → system) | [e.g., REST API via adapter] | [e.g., Service account OAuth] | [Internal] | [e.g., AI-generated fields marked with ai_assisted=true] |
| [e.g., SharePoint] | Read (system → AI) | [e.g., Graph API] | [e.g., Managed identity] | [Internal/Sensitive] | [e.g., Daily delta sync via ingestion pipeline] |

**Event infrastructure:**
[Describe the event bus or message queue used for inbound triggers, if applicable. Tool, hosting, SLA.]

---

## 4. Non-Functional Requirements

### Data Residency and Privacy

| Requirement | How It Is Met |
|---|---|
| Canadian data residency | [e.g., Azure OpenAI canadacentral, Qdrant self-hosted in Azure canadacentral] |
| PII handling | [e.g., Presidio PII detection runs on all inputs; PII fields replaced before LLM call] |
| Data retention | [e.g., Audit logs retained 90 days; user query logs retained 30 days; no long-term storage of LLM responses] |
| Data classification enforcement | [e.g., Metadata filtering in retrieval prevents cross-tier data leakage] |

### Performance

| Requirement | Target | Measurement |
|---|---|---|
| p50 response latency | < [X] seconds | [e.g., Langfuse trace p50] |
| p95 response latency | < [X] seconds | [e.g., Langfuse trace p95] |
| Availability | [e.g., 99.5%] | [e.g., Azure Monitor uptime check] |
| Concurrent users | [e.g., 50 simultaneous] | [e.g., Load test prior to production launch] |

### Security

| Control | Implementation |
|---|---|
| Authentication | [e.g., Azure AD SSO for user-facing UI, team-scoped API keys for programmatic access] |
| Authorisation | [e.g., Role-based access, document-tier filtering in retrieval] |
| Secrets management | [e.g., Azure Key Vault for all API keys and connection strings] |
| Audit logging | [e.g., All LLM requests logged to immutable audit store] |
| Penetration testing | [e.g., Required before production launch] |

---

## 5. Architecture Decisions

*Document significant architectural choices as lightweight ADRs. Include enough context that a future architect can understand why the decision was made without relitigating it.*

### ADR-001: [Decision title]

**Status:** Accepted

**Context:** [What was the decision about? What options were considered?]

**Decision:** [What was decided?]

**Rationale:** [Why was this option chosen over the alternatives?]

**Consequences:** [What does this decision make easier? What does it make harder? What are the open questions?]

---

### ADR-002: [Decision title]

**Status:** Accepted

**Context:** [...]

**Decision:** [...]

**Rationale:** [...]

**Consequences:** [...]

---

## 6. Open Questions and Assumptions

| Item | Type | Owner | Status | Due Date |
|---|---|---|---|---|
| [e.g., Confirm data residency requirements with legal for customer data] | Assumption to validate | [Name] | Open | [Date] |
| [e.g., ServiceNow rate limits for AI write volume] | Open question | [Name] | Open | [Date] |

---

## 7. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | [Date] | [Name] | Initial draft |

---

*Template version 1.0. Part of the [CloudKraft AI Control Tower framework](../framework/README.md).*
