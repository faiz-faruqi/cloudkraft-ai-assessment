# Layer 3 - Orchestration

**The question this layer answers:** How does the organisation design, govern, and maintain multi-step AI workflows that are reliable, cost-controlled, and auditable?

**Why this layer matters:** Single-step LLM calls are demos. Multi-step workflows with conditional logic, tool use, human escalation, and error handling are production systems. The orchestration layer is where AI moves from a novelty to a business capability - and where most AI programmes accumulate their worst technical debt.

**Primary stakeholders:** VP Engineering, Platform Engineering, Head of AI, Enterprise Architecture

---

## What Good Looks Like at Each Maturity Level

**Level 1 - Initial**
AI is used in isolated, single-step interactions. Developers implement LLM calls directly in application code with no shared patterns. Each team reinvents the same retry logic, error handling, and prompt management. There are no multi-step workflows.

**Level 2 - Developing**
Basic chaining exists in individual applications. Some teams use LangChain or LlamaIndex for simple pipelines. Patterns are not shared across teams. Human oversight is informal - outputs are reviewed by whoever happens to be available.

**Level 3 - Defined**
Shared orchestration patterns are documented and reused across teams. A framework choice has been made (LangGraph, Semantic Kernel, or equivalent) and is consistently applied. Human-in-the-loop checkpoints are explicitly designed into workflows for high-stakes decisions. Architecture Decision Records document significant orchestration choices.

**Level 4 - Managed**
Workflow costs are tracked per orchestration step. Token budgets are enforced. Failure modes are explicitly handled with retry logic, fallbacks, and circuit breakers. Orchestration decisions are documented in ADRs with version history. Cross-team pattern reuse reduces duplication.

**Level 5 - Optimising**
Automated cost optimisation at the orchestration layer - semantic caching, model routing by cost tier, prompt compression. Continuous monitoring of workflow reliability and performance. Pattern library maintained as a shared engineering resource. New workflows are reviewed against the pattern library before implementation.

---

## Named Patterns

### Pattern 1 - Sequential Agent Pipeline

**What it is:** A linear chain of specialised agents or steps where each step receives the output of the previous step, processes it, and passes the result forward. No branching, no loops, no parallel execution.

**When to use it:** Well-defined, repeatable tasks with clear input-output contracts at each step. Document processing pipelines, structured data extraction, report generation. The majority of enterprise AI use cases that are actually in production today are sequential pipelines, not the autonomous agents that dominate conference talks.

**Example:** Regulatory document ingestion pipeline - extract text, classify document type, extract key entities, validate against schema, generate summary, route for human review.

**Trade-offs:**
- Simple to reason about, test, and debug
- Each step is independently testable
- Failures are easy to locate and retry from a known checkpoint
- Not suitable for tasks requiring dynamic branching or parallel work

---

### Pattern 2 - Supervisor-Worker Multi-Agent

**What it is:** A supervisor agent that breaks down a complex task, assigns subtasks to specialised worker agents, collects results, and synthesises a final output. Workers do not communicate with each other - all coordination goes through the supervisor.

**When to use it:** Complex tasks that naturally decompose into parallel or sequential subtasks handled by different specialisations. Research synthesis, competitive analysis, multi-document comparison. Use this pattern only when the task genuinely requires it - the complexity cost is real.

**Trade-offs:**
- Significantly higher cost than sequential pipelines - multiple LLM calls per task
- Harder to test - supervisor behaviour depends on how it interprets and assigns tasks
- Requires careful prompt engineering for the supervisor to decompose tasks consistently
- Failure modes are more complex - a worker failure may or may not be recoverable depending on the supervisor

**CloudKraft recommendation:** Most teams underestimate the cost and overestimate the benefit of multi-agent systems. Start with a sequential pipeline. Add a supervisor only when the sequential approach demonstrably fails due to task complexity, not in anticipation of future complexity.

---

### Pattern 3 - Human-in-the-Loop Escalation

**What it is:** Explicit checkpoints in an AI workflow where the system pauses and routes output to a human reviewer before proceeding. The human can approve, modify, or reject the AI output.

**When to use it:** Any workflow where AI errors have material consequences - regulatory filings, customer communications, financial calculations, legal document generation, medical information. In regulated industries, this pattern is frequently a compliance requirement, not an optional design choice.

**Design principles:**
- Define the escalation criteria explicitly - what confidence threshold, what output type, what risk tier triggers human review
- The human review interface must show the AI output, the reasoning, and the source documents - not just the output
- Escalation paths must have SLAs - a workflow that pauses indefinitely waiting for human review is not a production system
- Every human decision must be logged with the reviewer identity, timestamp, and decision rationale

**Trade-offs:**
- Increases end-to-end latency significantly for escalated cases
- Requires investment in reviewer tooling - the review interface is as important as the AI system
- Human bottlenecks can become system bottlenecks under load

---

### Pattern 4 - Event-Driven AI Trigger

**What it is:** An AI workflow triggered by an enterprise event (document uploaded, ticket created, email received, data threshold crossed) rather than by a direct user request.

**When to use it:** Operational automation use cases where AI processing should happen in the background without user initiation. Contract review on upload, anomaly analysis on threshold breach, ticket classification on creation.

**Trade-offs:**
- Requires reliable event infrastructure (message queue, event bus)
- Error handling is more complex - the triggering user may not be present when the workflow fails
- Cost control requires careful design - a high-volume event source can generate unexpected LLM spend

---

## Tool Selection Matrix - Orchestration Layer

| Tool | Best For | Strengths | Weaknesses |
|---|---|---|---|
| LangGraph | Complex stateful workflows, multi-agent | First-class state management, strong typing, production-ready | Steeper learning curve than LangChain basic |
| LangChain LCEL | Simple to medium chains | Broad ecosystem, many integrations | Abstraction complexity, frequent API changes |
| Semantic Kernel | Microsoft/Azure stack, enterprise .NET | Strong enterprise integration, Azure-native | Smaller community, slower ecosystem |
| CrewAI | Multi-agent with role-based design | Good developer experience, active community | Less mature for production, limited state management |
| AutoGen | Research-oriented multi-agent | Flexible, Microsoft-backed | Not production-ready for most enterprise use cases |
| n8n | Workflow integration, non-technical users | Visual design, broad integrations | Not designed for complex AI logic |
| Custom | Full control, specific requirements | No framework constraints | Build and maintenance cost |

**CloudKraft recommendation:** LangGraph for any stateful or multi-step workflow in production. It is the most mature framework for production AI orchestration as of 2026. LangChain LCEL for simple chains where state management is not required. Semantic Kernel for organisations deeply committed to Microsoft Azure with .NET teams. n8n for workflow integration use cases where the AI step is one component in a broader business process automation.

**What to avoid:** AutoGen in production environments - it is a research tool, not a production framework. CrewAI for high-stakes workflows until the framework matures. Any framework without active maintenance and a clear enterprise support path.

---

## Anti-Patterns

### Unbounded Agent Loops

**What happens:** An agent workflow has no maximum iteration limit. Under normal conditions it completes in 3-5 steps. Under edge conditions it loops 50+ times, consuming tokens and generating cost until an external timeout kills it.

**Why it is dangerous:** A single stuck agent can cost tens or hundreds of dollars. At scale, unbounded loops can generate thousands of dollars of unexpected spend before anyone notices. In addition to cost, unbounded loops produce unpredictable behaviour that is impossible to reason about in a compliance context.

**The fix:** Every agent loop has an explicit maximum iteration count. Every workflow has a maximum total token budget. Both limits cause a graceful failure with a structured error response, not an uncontrolled termination.

---

### Missing Fallback Logic

**What happens:** The orchestration layer calls an LLM provider. The provider returns a 429 (rate limit), 500 (server error), or 503 (service unavailable). The workflow crashes. The user sees an unhandled error. If the workflow is event-driven, the event is lost.

**Why it happens:** Fallback logic is considered a quality-of-life improvement, not a correctness requirement. It gets deferred to "after launch."

**The fix:** Every LLM call has retry logic with exponential backoff. Every workflow has a fallback path for unrecoverable provider failures - either a degraded response ("I cannot answer this right now") or a human escalation. Event-driven workflows use a dead-letter queue for failed events.

---

### Over-Engineering Single-Step Tasks into Multi-Agent Systems

**What happens:** A team reads about multi-agent systems at a conference. They build a five-agent pipeline to answer questions that could be answered with a single well-engineered prompt. The system is 10x more expensive to run, 5x harder to debug, and no more accurate than the single-prompt approach.

**Why it happens:** Multi-agent architectures are genuinely interesting. They are also genuinely unnecessary for most production enterprise AI use cases.

**The test:** Before adding an agent to a workflow, ask: "What does this agent do that a tool call or a better prompt cannot?" If the answer is unclear, the agent is probably not necessary.

---

## Key Questions for a Client Engagement

1. Show me your most complex AI workflow. Walk me through what happens when step 3 fails. What does the user see? What gets logged? What gets retried?

2. If your LLM provider had a 30-minute outage right now, which of your AI workflows would fail, which would degrade gracefully, and which would not be affected? How do you know?

3. Pick any AI workflow that a human should review before the output reaches an end user. Show me the review interface. How does the reviewer know what the AI was trying to do and why?

4. How much did your most expensive AI workflow cost to run last month, in LLM tokens? How does that compare to what you expected?

5. When a developer on your team wants to add a new AI workflow, what is the process? Is there an architectural review? Is there a pattern they should follow? Is there a cost estimate required?

---

## Relationship to Other Layers

**Layer 3 depends on Layer 1:** The gateway enforces the cost budgets and model routing rules that the orchestration layer relies on. Orchestration without a gateway means each workflow manages its own cost controls - an invitation to inconsistency.

**Layer 3 uses Layer 2:** Multi-step workflows frequently include retrieval steps. The quality of the retrieval layer directly affects orchestration reliability - a retrieval failure mid-workflow is a common source of cascading failures.

**Layer 3 feeds Layer 4:** Workflow execution data (step latencies, token counts, failure rates, human escalation rates) are the primary inputs to the Layer 4 evaluation and observability pipeline.

---

*Part of the [CloudKraft AI Control Tower framework](./README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this layer.*
