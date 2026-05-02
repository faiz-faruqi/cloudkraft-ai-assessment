# Pattern: Human-in-the-Loop

**Layer:** 3 — Orchestration
**Problem:** AI systems in high-stakes workflows make consequential errors that are not detected until after the damage is done — a regulatory filing submitted with incorrect figures, a customer commitment made outside policy, a contract clause approved with the wrong interpretation.
**Solution:** Explicit checkpoints in AI workflows where outputs are routed to a human reviewer before proceeding, with a structured review interface, defined SLAs, and complete audit logging of every decision.

---

## Context

Human-in-the-loop (HITL) is not a concession to AI limitations — it is a deliberate architectural choice for workflows where AI errors have material consequences. The goal is not to review every AI output (that eliminates the efficiency gain) but to identify the specific conditions under which AI errors are unacceptable and insert human review precisely there.

HITL is frequently a compliance requirement, not an optional design decision. In Canadian regulated industries:
- OSFI Guideline B-13 requires human oversight for AI systems used in risk-relevant decisions
- PIPEDA obligations around automated decision-making require meaningful human review for decisions affecting individuals
- CSA/IIROC guidelines for AI in client-facing financial services require escalation paths for low-confidence outputs

The pattern fails most often not because the AI is wrong, but because the review interface is inadequate — reviewers are asked to approve or reject outputs without the context needed to make an informed decision.

---

## When to Apply This Pattern

Apply human-in-the-loop whenever one or more of the following conditions are true:

**By consequence:**
- The AI output will be sent to a customer, regulator, or external party
- The AI output will trigger a financial transaction, regulatory filing, or binding commitment
- An incorrect AI output would require meaningful remediation effort (more than an hour of human work to fix)

**By confidence:**
- The AI system's confidence score falls below a defined threshold
- The query is outside the distribution the system was evaluated on
- Retrieved context quality is below a defined precision threshold

**By data sensitivity:**
- The workflow involves sensitive personal information, financial records, or regulated data
- The AI output contains medical, legal, or financial advice

**By policy:**
- The use case involves a credit decision, employment decision, or other automated decision subject to regulatory review requirements
- The document type or transaction type is on a mandatory review list

---

## How It Works

```
AI Workflow Step N
        ↓
  Escalation Evaluator
  (confidence check, output classifier, risk tier)
        ↓
  ┌─ Below threshold ─────────────────────────────┐
  │   Route to Review Queue                        │
  │   Notify reviewer with SLA timer               │
  │   Present: AI output + reasoning + sources     │
  │   Reviewer: Approve / Modify / Reject          │
  │   Log: reviewer ID, timestamp, decision, notes │
  │         ↓                                      │
  └─ Continue workflow with reviewed output ───────┘
  ┌─ Above threshold ──────────────────────────────┐
  │   Continue workflow automatically              │
  │   Log: auto-approved, confidence score         │
  └────────────────────────────────────────────────┘
```

---

## Design Principles

**Principle 1: Show the work, not just the answer**

A review interface that shows only the AI output is nearly useless. Reviewers cannot evaluate correctness without seeing the sources the AI used, the confidence indicators, and if applicable, the intermediate reasoning steps. The review interface must show:
- The AI-generated output (final answer or draft)
- The source documents or data used to generate it (with highlighted relevant passages)
- The confidence score or uncertainty indicators
- Any system-generated flags (e.g., "retrieved context was last updated 47 days ago")
- The original user request or triggering event

**Principle 2: SLAs are not optional**

A workflow that pauses indefinitely waiting for human review is not a production system. Every HITL queue must have a defined SLA (e.g., "high-priority reviews within 2 hours, standard within 24 hours"). The system must escalate automatically if a review is not completed within the SLA. SLA breaches must be visible in operational dashboards.

**Principle 3: Calibrate escalation thresholds, do not guess**

An escalation rate of 100% defeats the purpose. An escalation rate of 0% means the threshold is too permissive. Target escalation rates should be determined empirically: run the system in shadow mode (route all outputs through the HITL queue but record reviewer decisions without blocking the workflow) to calibrate the threshold before going live.

**Principle 4: Audit every decision**

Every human decision in the HITL queue — approve, modify, reject — must be logged with reviewer identity, timestamp, the original AI output, the final output after review (if modified), and any reviewer notes. This log is compliance evidence. Treat it with the same retention and integrity standards as financial records.

**Principle 5: Close the feedback loop**

Reviewer corrections are the most valuable training signal in any enterprise AI programme. Build a pipeline that captures corrections, aggregates them into a fine-tuning or few-shot dataset, and periodically uses them to improve the model or prompt. A HITL system that does not improve the underlying AI is a permanent tax on human attention.

---

## Implementation Components

**Review queue infrastructure**
- Message queue (SQS, Azure Service Bus, or Pub/Sub) to hold pending reviews
- Review dashboard (custom web UI or integrated into existing tools like ServiceNow, Jira, or Slack with interactive message components)
- SLA timer and escalation logic
- Notification routing (email, Slack, Teams)

**Escalation evaluator**
The logic that decides whether a given AI output requires human review. This can be:
- A simple rule: if confidence score < threshold, escalate
- A classifier: a small model trained on historical outputs to predict review likelihood
- A structured policy: specific output types always escalate (e.g., all outputs in the "credit decision" category)

**Audit log store**
Structured storage for every review event with queryable fields. Minimum schema:

```
review_id          UUID
workflow_id        UUID (links to the triggering AI workflow)
timestamp_created  ISO8601
timestamp_resolved ISO8601
reviewer_id        string
ai_output          text (the original AI-generated output)
final_output       text (the output after human review, which may be identical)
decision           enum: approved | modified | rejected
confidence_score   float (the AI system's confidence at time of escalation)
escalation_reason  string
reviewer_notes     text
```

---

## Tool Integration

| Capability | Recommended Approach |
|---|---|
| Review queue | SQS + custom React dashboard; or ServiceNow workflow integration for enterprises already using it |
| Slack-based review | Slack Block Kit interactive messages with Approve/Modify/Reject buttons — good for internal ops teams, not recommended for compliance-critical workflows |
| LangGraph integration | Use `interrupt()` nodes in LangGraph to pause a graph and route to review — the cleanest orchestration-layer integration |
| Confidence scoring | Logprobs from the LLM (available from OpenAI and Anthropic APIs) as a proxy for output confidence |
| Audit logging | Write to the same structured log as the AI gateway — maintains a single audit trail |

---

## Trade-offs

**What you gain:**
- Material reduction in downstream errors for high-stakes outputs
- Audit trail that demonstrates human oversight — critical for regulatory and compliance evidence
- Feedback signal for continuous improvement
- Reduced liability exposure for AI-assisted decisions

**What it costs:**
- End-to-end latency increases for escalated cases — make this explicit in user expectations
- Human reviewer time — HITL is not free; the cost per review must be justified by the cost of the errors it prevents
- Review tooling must be built and maintained — the interface is as important as the AI system
- Human bottlenecks can become system bottlenecks under load — model escalation rate and reviewer capacity together

---

## Anti-Pattern: Checkbox HITL

The most common HITL failure is a review interface that shows the AI output with an Approve button and nothing else. Reviewers click Approve in under three seconds because they have no basis for any other decision. This produces an audit trail that claims human oversight occurred while providing none.

If the review interface does not show sources, confidence indicators, and context, it is not a HITL system — it is a liability shield with no actual protective value.

---

## Relationship to Other Patterns

**Depends on:**
- [Centralised AI Gateway](./centralised-ai-gateway.md) — gateway audit logs link the HITL review event back to the original LLM call
- [Hybrid RAG](./hybrid-rag.md) — retrieval confidence scores are a primary escalation trigger

**Feeds into:**
- [Continuous Evaluation](./continuous-evaluation.md) — reviewer corrections are ground-truth labels for evaluation
- [AI Workflow Orchestration Bridge](./ai-workflow-orchestration.md) — HITL queues integrate with enterprise workflow systems (ServiceNow, Jira) through the orchestration bridge

---

*Part of the [CloudKraft AI Control Tower framework](../framework/README.md). See [Layer 3 — Orchestration](../framework/layer-3-orchestration.md) for the full layer context.*
