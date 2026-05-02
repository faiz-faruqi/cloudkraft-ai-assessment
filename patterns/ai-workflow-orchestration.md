# Pattern: AI Workflow Orchestration Bridge

**Layer:** 5 — Integration
**Problem:** AI systems are built in isolation and cannot interact with the enterprise systems — ticketing, CRM, ERP, ITSM, document management — where actual business work happens. AI produces outputs that users then manually copy into the systems of record.
**Solution:** A structured integration bridge that connects AI workflows bidirectionally to enterprise systems, using event-driven triggers, standardised API contracts, and idempotent writes to ensure that AI outputs land in systems of record reliably and auditably.

---

## Context

The most common failure mode in enterprise AI integration is not technical — it is architectural. Teams build impressive AI capabilities and then connect them to enterprise systems with fragile, point-to-point integrations: a Python script that calls the ServiceNow REST API, a cron job that polls a database, a webhook handler with no error handling and no retry logic.

These integrations work in demos. They fail in production because enterprise systems have rate limits, authentication expiry, schema changes, and maintenance windows. An AI system that loses its enterprise system connection silently fails in ways users cannot diagnose.

The Orchestration Bridge pattern treats AI-to-enterprise integration as a first-class architectural concern with the same reliability standards applied to any other enterprise integration.

---

## How It Works

```
Enterprise Event Source          AI Workflow            Enterprise System of Record
(ServiceNow, Jira, SharePoint,   (LangGraph,            (CRM, ERP, ITSM,
 email, S3, database trigger)     custom agent)          document management)
          │                            │                          │
          ▼                            ▼                          ▼
  Event Bus / Queue ────────► AI Orchestration ────────► Integration Adapter
  (SQS, Service Bus,          (trigger, execute,          (idempotent writes,
   EventBridge, Pub/Sub)       handle errors,              schema validation,
                               log execution)              retry with backoff,
                                                           dead-letter queue)
```

**Inbound (enterprise → AI):**
Enterprise events trigger AI workflows through an event bus rather than direct calls. This decouples the enterprise system from the AI execution layer, allowing the AI system to process at its own pace, buffer during outages, and retry without affecting the source system.

**Outbound (AI → enterprise):**
AI workflow outputs are written to enterprise systems through integration adapters that handle authentication, schema validation, idempotent writes, and retry logic. Every write is logged with the source workflow ID and timestamp, creating an end-to-end audit trail.

---

## Integration Scenarios

### Scenario 1 — Event-triggered AI processing

A document is uploaded to SharePoint. The upload triggers an event. The event is placed on a message queue. The AI workflow picks up the event, processes the document (classify, extract, summarise), and writes structured output back to the document library as metadata.

**Why use an event bus rather than a direct trigger:**
- The AI processing may take 30–120 seconds — longer than most synchronous webhook timeouts
- If the AI system is unavailable, events queue rather than being lost
- Processing rate can be controlled independently of upload rate
- Multiple AI workflows can subscribe to the same event without coupling them to each other

### Scenario 2 — AI-enriched ticket creation

A customer submits a support request. Before the ticket is created in ServiceNow, an AI workflow classifies the issue, identifies related known issues, estimates resolution time, and suggests the appropriate assignment group. The enriched ticket is written to ServiceNow with AI-generated fields clearly labelled as AI-assisted.

**Design considerations:**
- AI enrichment must not block ticket creation — run asynchronously and update the ticket within an SLA
- AI-generated fields must be distinguishable from human-entered fields in the ITSM schema
- The enrichment workflow must have a timeout and a fallback that creates the ticket with no AI enrichment if processing fails

### Scenario 3 — Bidirectional CRM integration

A sales AI assistant drafts meeting notes and action items after a call. The draft is reviewed by the account manager (human-in-the-loop), then written to Salesforce as activity records with the original AI draft preserved as an audit field. Follow-up tasks are created in Salesforce from the AI-extracted action items.

**Key design decisions:**
- SFDC has strict rate limits — buffer writes and respect the API limits
- The AI draft and the final approved version must both be stored — the delta shows where human judgment overrode AI output
- Opportunity and contact associations must be resolved before writing — the AI must identify which SFDC records to link, which requires a lookup step with error handling for ambiguous matches

---

## Integration Adapter Design

Every enterprise system integration should be implemented as a dedicated adapter module with a consistent interface:

```python
class EnterpriseAdapter:
    def write(self, payload: dict, idempotency_key: str) -> WriteResult:
        """Write output to the enterprise system with idempotent behaviour."""

    def read(self, query: dict) -> list[dict]:
        """Query the enterprise system — used for enrichment and context retrieval."""

    def health_check(self) -> bool:
        """Verify the adapter can reach the enterprise system."""
```

**Idempotency is not optional.** If an AI workflow writes a record to ServiceNow and then receives a timeout before the HTTP response arrives, it will retry. Without idempotent writes, you get duplicate tickets, duplicate records, and duplicate entries in the audit log. Every write operation must use an idempotency key derived from the workflow ID and step ID, and the enterprise system adapter must handle duplicate submissions gracefully.

---

## Tool Selection

### Event Bus / Message Queue

| Tool | Best For | Cloud | Notes |
|---|---|---|---|
| AWS SQS + EventBridge | AWS-committed organisations | AWS | Mature, reliable, good Lambda integration |
| Azure Service Bus | Azure-committed organisations | Azure | Strong enterprise features, good .NET SDK |
| Google Pub/Sub | GCP-committed organisations | GCP | Simpler API, good Python SDK |
| Kafka (Confluent) | High-throughput, cross-cloud | Self-hosted or cloud | Operational complexity not justified for most AI integration use cases |
| RabbitMQ | Self-hosted, simple use cases | Self-hosted | Good for teams wanting full control without Kafka complexity |

### AI Orchestration Frameworks

| Tool | Strengths | Best For |
|---|---|---|
| LangGraph | First-class state management, pause/resume, human-in-the-loop | Complex multi-step workflows with conditional logic |
| n8n | Visual design, 400+ native integrations, non-technical users | Integration workflows where the AI step is one component |
| Temporal | Durable execution, long-running workflows, built-in retry | Workflows that run for hours or days with human steps |
| Apache Airflow | Data engineering teams, batch workflows | AI workflows that fit naturally into data pipeline patterns |
| Custom FastAPI + Celery | Full control, specific requirements | Teams with strong Python backgrounds, complex requirements |

**CloudKraft recommendation:** LangGraph for AI-first workflows where the integration is a supporting concern. n8n for integration-first workflows where AI is one step in a broader business process. Temporal for workflows that run over long time periods (hours to days) and require durable execution guarantees — this is the correct choice for any AI workflow that involves human approval steps with multi-day SLAs.

### Enterprise Integration Platforms

| Platform | Notes |
|---|---|
| MuleSoft | Existing MuleSoft customers: use it as the integration layer, call AI APIs as a transformation step |
| Azure Integration Services | Azure-committed organisations, strong Logic Apps integration |
| Boomi | Mid-market organisations with existing Boomi investment |
| Custom adapters | For enterprise systems without good managed connectors |

---

## Governance and Audit

Every AI-to-enterprise write must generate an audit record linking:
- The source workflow execution ID
- The triggering event (document upload, ticket creation, API call)
- The AI system that generated the output
- The model and prompt version used
- The human reviewer who approved the output (if HITL was applied)
- The final payload written to the enterprise system
- The timestamp and the system user identity used for the write

This audit trail is the evidence that an AI system wrote this record, not a human — and it is critical for incident investigation, compliance review, and model debugging.

---

## Trade-offs

**What you gain:**
- AI outputs land reliably in systems of record rather than sitting in a separate AI tool
- Business workflows become AI-augmented without requiring users to change tools
- End-to-end audit trail from enterprise event to AI processing to enterprise output
- Decoupled architecture: enterprise systems and AI systems can be updated independently

**What it costs:**
- Significantly more complex than direct API calls — event bus, adapters, dead-letter queues
- Each enterprise system integration requires dedicated development and maintenance
- Schema changes in enterprise systems break adapters — requires versioning and change management
- Debugging distributed event-driven systems requires better tooling than debugging synchronous code

---

## Anti-Pattern: Synchronous Point-to-Point

The most common integration failure is building the AI-to-enterprise connection as a synchronous HTTP call in the AI workflow step. This creates tight coupling, is brittle to timeouts, has no retry mechanism, and produces no audit trail. It works in development and fails in production.

If you are calling a ServiceNow, Salesforce, or SAP API directly from an AI workflow step with no intermediate queue and no idempotency handling, you have a point-to-point integration, not an orchestration bridge. Fix it before it becomes a production incident.

---

## Relationship to Other Patterns

**Depends on:**
- [Centralised AI Gateway](./centralised-ai-gateway.md) — AI API calls made during orchestration route through the gateway for policy enforcement and cost attribution
- [Human-in-the-Loop](./human-in-the-loop.md) — the HITL review queue is itself an enterprise system integration concern

**Anti-patterns this pattern prevents:**
- [Tight Coupling of AI Systems](../anti-patterns/tight-coupling-ai-systems.md)

---

*Part of the [CloudKraft AI Control Tower framework](../framework/README.md). See [Layer 5 — Integration](../framework/layer-5-integration.md) for the full layer context.*
