# Layer 5 - Integration & Scale

**The question this layer answers:** Is AI embedded into how the business actually works, observable in production, cost-controlled, and hardened enough to survive real load and real failures?

**Why this layer matters:** This is where a pilot either becomes durable enterprise infrastructure or quietly dies the first time it fails under load, blows through budget, or turns out nobody is watching it. Every other layer can be strong and still fail here if production reality was never designed for.

**Primary stakeholders:** Enterprise Architecture, SRE / Platform Engineering, FinOps

---

## What Good Looks Like at Each Maturity Level

**Level 1 - Initial**
AI exists only in standalone tools disconnected from core business workflows. No observability exists beyond basic uptime - problems are discovered through user complaints. Cost is discovered only in the monthly bill. A single failure takes the AI system fully offline, with no failover or rollback strategy.

**Level 2 - Developing**
A handful of point integrations exist, built individually for each use case. Logs and dashboards exist for some systems but are not unified. Cost is monitored but not actively managed. Basic error handling exists but no failover or rollback strategy.

**Level 3 - Defined**
Shared integration patterns exist but embedding into workflows is still largely manual. Budgets exist per team or project, though optimisation is reactive. Some systems have retry logic but resilience is inconsistent across the estate. A basic pilot-to-production checklist exists but is applied inconsistently.

**Level 4 - Managed**
AI capabilities are embedded as native steps in core business workflow and process tools. Unified observability covers latency, cost, and quality signals across most systems. Proactive cost optimisation (caching, model routing, budget alerts) is applied broadly. Standardised resilience patterns are applied to most production AI systems.

**Level 5 - Optimising**
AI is embedded end-to-end across enterprise workflows with event-driven triggers and automated handoffs. Full-stack observability - traces, cost, quality, drift - is standard, with automated alerting and root-cause tooling. Automated cost governance dynamically optimises spend across models, caching, and routing. A mature, repeatable scaling framework moves pilots to enterprise production with predictable timelines and quality bars.

---

## Named Patterns

### Pattern 1 - Embedded Workflow Integration

**What it is:** AI capability delivered as a native step inside the business process and workflow tools people already use, rather than as a separate destination they have to remember to visit.

**When to use it:** Any AI capability intended for sustained operational use rather than occasional lookup. A standalone chat interface is fine for exploration; it is rarely how durable business value gets captured.

**How it works:**
- The AI capability is exposed as an API or event consumer that plugs into the existing workflow or BPM tool
- Triggers come from business events (a case is created, a document arrives) rather than a human remembering to open a separate tool
- Outputs write back into the system of record, with human review gates where the decision stakes warrant it

**Trade-offs:**
- Requires integration work with existing enterprise systems, which is slower than shipping a standalone tool
- Embedding AI into a critical workflow raises the stakes of a failure - which is exactly why Layer 5 hardening and observability matter

**Tools that implement this pattern:**

| Tool | Best For | Limitations |
|---|---|---|
| Kong / Apigee (API gateway) | Exposing AI capabilities as governed APIs into existing systems | Adds an infrastructure layer to operate |
| Temporal | Durable, embedded multi-step workflows with retries and state | Steeper learning curve, meaningful operational investment |
| Native workflow tool extensions (ServiceNow, Salesforce, etc.) | Embedding into a workflow tool the business already uses | Tied to that platform's extensibility model |
| Custom event consumer | Full control over trigger and handoff logic | Build and maintenance cost |

**CloudKraft recommendation:** Embed into the workflow tool the business already trusts and uses daily wherever possible - adoption is dramatically higher than any standalone AI destination, no matter how well built.

---

### Pattern 2 - Full-Stack AI Observability

**What it is:** Unified observability across the full AI request lifecycle - latency, cost, retrieval quality, and output quality - rather than infrastructure metrics alone.

**When to use it:** Every production AI system. Standard APM tools show you the system is "up" while giving no signal that answer quality has silently degraded.

**Trade-offs:**
- LLM-specific observability tools are a newer category than traditional APM - expect more integration work and faster tool evolution
- Capturing prompts and outputs for observability has data handling implications that must align with Layer 3 governance controls

---

### Pattern 3 - Progressive Production Hardening

**What it is:** A defined, staged path from pilot to full production - readiness gates, progressive rollout, and tested failure modes - rather than a single cutover from "works in the demo" to "live for everyone."

**When to use it:** Every AI system moving from pilot to production, proportional to the stakes of the use case.

**Typical findings:** In CloudKraft assessments, the gap between pilot and production readiness is consistently underestimated - a system that works reliably for a five-person pilot group frequently reveals failure modes (rate limits, cost blowouts, degraded-mode gaps) within days of wider rollout.

---

## Tool Selection Matrix - Integration & Scale Layer

| Capability | LangSmith / Langfuse | Datadog / Grafana | OpenTelemetry | CloudZero / Vantage | Custom |
|---|---|---|---|---|---|
| LLM-specific tracing | Excellent | Limited | Good (with instrumentation) | No | Full control |
| Infrastructure observability | Limited | Excellent | Good | No | Full control |
| Cost attribution (FinOps) | Basic | Limited | No | Excellent | Custom |
| Alerting / incident response | Basic | Excellent | N/A (data layer only) | Limited | Custom |
| Vendor-neutral / self-hosted option | Langfuse (self-hosted) | Grafana (self-hosted) | Yes (open standard) | No | Yes |
| Setup complexity | Low-Medium | Medium | Medium-High | Low-Medium | High |

---

## Anti-Patterns

### The Pilot-to-Production Cliff

**What happens:** An AI system that worked well for a small pilot group is rolled out enterprise-wide with no changes to its architecture, cost controls, or resilience. It fails within days - rate limits are hit, costs spike, or a dependency that never mattered at pilot scale becomes a bottleneck.

**Why it happens:** Pilot success is treated as proof of production readiness, when it only proves the concept works, not that it works at scale.

**Why it is dangerous:** A visible production failure right after a high-profile rollout does more damage to organisational trust in AI than a pilot that was never scaled at all.

**The fix:** Progressive Production Hardening (Pattern 3) with explicit readiness gates between pilot and each stage of wider rollout.

---

### Cost Blindness at Scale

**What happens:** Per-request AI cost looked negligible during the pilot. At enterprise scale, the same per-request economics produce a monthly bill that blindsides finance, with no mechanism to have caught it earlier.

**Why it happens:** Variable, usage-based AI cost does not behave like a fixed software licence, and cost visibility that felt sufficient at low volume simply does not scale.

**Why it is dangerous:** Beyond the budget surprise, unmanaged cost growth is often the trigger that gets an otherwise valuable AI system shut down or scaled back under pressure.

**The fix:** Proactive cost optimisation and budget alerting (Pattern 2, plus the cost-attribution practices from Layer 1's gateway) built in before scale-up, not after the first alarming invoice.

---

### The Silent Failure

**What happens:** An AI system degrades in production - lower quality answers, a broken retrieval path, an upstream dependency quietly failing over to a fallback - and nothing alerts anyone. The degradation is discovered eventually through user complaints or a spot check, well after it began.

**Why it happens:** Traditional uptime monitoring shows the system as healthy because it is still responding; it just is not responding well.

**Why it is dangerous:** The longer a quality degradation goes undetected, the more decisions and outputs it has already contaminated, and the harder it is to determine the actual blast radius after the fact.

**The fix:** Full-Stack AI Observability (Pattern 2) that tracks quality and drift signals, not infrastructure health alone, with alerting tied to those signals.

---

## Key Questions for a Client Engagement

1. Walk me through how a user actually encounters your AI capability today - is it embedded in their normal workflow, or a separate destination?

2. If AI output quality silently degraded tomorrow, how would you find out, and how long would it take?

3. What happens to your AI cost if usage tripled overnight - do you know, and would anyone be alerted before the bill arrived?

4. What is your process for moving an AI system from pilot to full production, and what has to be true before that happens?

5. If your primary AI provider had an outage right now, what would your users experience?

The quality of the answers to these five questions is a reliable predictor of the overall Layer 5 maturity score.

---

## Relationship to Other Layers

**Layer 5 exposes weaknesses in every other layer:** Production scale is where gaps in data readiness, architecture patterns, governance, and operating model become visible and costly - it is the layer that tests whether the other four were actually built to last.

**Layer 5 depends on Layer 2 (Architecture Patterns):** The integration and orchestration patterns chosen determine how much re-architecture scaling requires.

**Layer 5 provides evidence for Layer 3 (Governance & Risk):** Production observability data is frequently the actual evidence used to demonstrate that governance controls are working, not just documented.

**Layer 5 requires Layer 4 (Operating Model):** Scaling an AI programme coherently across the enterprise requires the ownership and standards that Layer 4 provides - without it, scale-up happens unevenly, system by system.

---

*Part of the [CloudKraft AI Control Framework](./README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this layer.*
