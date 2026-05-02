# Pattern: Centralised AI Gateway

**Layer:** 1 — Access & Governance
**Problem:** Multiple teams are calling LLM providers directly, producing ungoverned cost, inconsistent policy enforcement, and zero audit trail.
**Solution:** A single proxy service that every LLM call in the enterprise routes through, with authentication, policy enforcement, cost attribution, and logging at the gateway layer.

---

## Context

This pattern applies to any enterprise with more than one team using AI. It becomes urgent when any of the following are true:

- Finance cannot answer "how much did we spend on AI last month, by team?"
- Security cannot answer "what data did we send to external AI providers this quarter?"
- A compliance audit asks for an AI request audit trail and you cannot produce one
- Teams are holding their own API keys with no central visibility

It is the prerequisite for every other governance pattern in this framework. You cannot enforce a model allowlist without a gateway. You cannot attribute cost without a gateway. You cannot block data exfiltration without a gateway.

---

## How It Works

```
Application → Gateway → LLM Provider
                ↓
         Policy Engine
         Auth (API Key / IAM)
         Model Allowlist
         Content Filter
         Rate Limiter
         Budget Check
                ↓
         Audit Log (request, response, tokens, cost, caller identity, timestamp)
```

1. All applications send OpenAI-compatible requests to the gateway endpoint — not directly to any provider
2. The gateway authenticates the caller using a team-scoped API key or workload IAM identity
3. Policy rules are evaluated: is the model on the allowlist? Is the team within its budget? Does the request contain classified data patterns?
4. If all checks pass, the request is forwarded to the appropriate provider (OpenAI, Azure OpenAI, Bedrock, Anthropic, or Ollama for self-hosted)
5. The response is returned to the caller and every request-response pair is written to a structured audit log with token count, estimated cost, and metadata

---

## Implementation Guidance

**Phase 1 — Install and configure the gateway (week 1–2)**

Stand up LiteLLM (self-hosted) or Portkey (hosted). Configure it as a pass-through proxy for the primary model. Issue team-scoped virtual API keys. Do not enforce policy yet — the goal is visibility first.

**Phase 2 — Route all traffic (week 3–4)**

Migrate all teams to the gateway endpoint. Rotate and disable existing direct API keys on a fixed date. Treat any team still calling providers directly as a policy exception requiring sign-off.

**Phase 3 — Enforce policy (week 5–8)**

Enable the model allowlist, rate limits, and budget alerts. Configure PII detection if required by your data classification policy. Deliver the first monthly cost-attribution report.

**Phase 4 — Optimise (ongoing)**

Add semantic caching to reduce repeat-query cost (Portkey semantic cache or GPTCache). Configure model fallback routing (GPT-4o primary → Claude Sonnet fallback on provider outage). Integrate audit logs with SIEM.

---

## Tool Selection

| Tool | Hosting | Best For | Limitations |
|---|---|---|---|
| LiteLLM | Self-hosted | Data residency requirements, full control, OpenAI-compatible | Requires infrastructure, less mature UI |
| Portkey | Hosted | Fast setup, semantic caching, strong observability | Less control over data handling on cloud plan |
| Kong AI Gateway | Self-hosted | Organisations already running Kong for API management | Higher operational complexity, slower iteration |
| AWS Bedrock Gateway | AWS-native | AWS-committed organisations, Bedrock models only | AWS-only, no multi-cloud routing |
| Azure AI Foundry | Azure-native | Azure-committed organisations | Azure-only |
| Custom FastAPI proxy | Self-built | Compliance requirements that packaged tools cannot meet | Full build and maintenance burden |

**CloudKraft recommendation:** LiteLLM self-hosted is the fastest path to a production-grade, data-resident gateway for regulated Canadian enterprises. It supports every major provider, is OpenAI-compatible, and can be stood up in a day. Portkey hosted is acceptable for organisations without strict data residency requirements who want a managed service. Custom FastAPI is warranted only when audit evidence requirements go beyond what packaged tools produce (e.g., specific fields required by a regulator).

---

## Trade-offs

**What you gain:**
- Complete visibility into AI spend, by team, by model, by use case
- Audit trail for every LLM request — queryable, structured, retention-controlled
- Model governance enforced at the network layer, not by team discipline
- Shadow AI detection: requests that do not route through the gateway are anomalous by definition
- Multi-provider fallback: if one provider has an outage, the gateway can route to a secondary

**What it costs:**
- One additional network hop: 10–50ms added latency. Acceptable for all enterprise AI use cases except real-time voice and sub-100ms interactive applications
- A new infrastructure component to operate, monitor, and maintain
- One-time migration cost: teams must update their endpoint configuration. In practice, this is a one-line environment variable change
- Creates a single point of failure — mitigate with active-active deployment and circuit breakers from day one

---

## Configuration Example (LiteLLM)

```yaml
model_list:
  - model_name: gpt-4o
    litellm_params:
      model: openai/gpt-4o
      api_key: os.environ/OPENAI_API_KEY
  - model_name: claude-sonnet
    litellm_params:
      model: anthropic/claude-sonnet-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

router_settings:
  routing_strategy: latency-based-routing
  fallbacks:
    - { "gpt-4o": ["claude-sonnet"] }
  allowed_fails: 3

litellm_settings:
  success_callback: ["langfuse"]
  failure_callback: ["langfuse"]

general_settings:
  master_key: os.environ/GATEWAY_MASTER_KEY
  database_url: os.environ/DATABASE_URL
```

---

## Relationship to Other Patterns

**This pattern enables:**
- [Hybrid RAG](./hybrid-rag.md) — the gateway enforces which models the retrieval pipeline is permitted to call
- [Continuous Evaluation](./continuous-evaluation.md) — gateway audit logs are the primary data source for production evaluation
- [Human-in-the-Loop](./human-in-the-loop.md) — the gateway can enforce that high-risk request types require a human review queue rather than direct model access

**Anti-patterns this pattern prevents:**
- [Shadow AI Sprawl](../anti-patterns/shadow-ai-sprawl.md)
- [Ungoverned Cost Accumulation](../anti-patterns/ungoverned-cost-accumulation.md)
- [BYOLLM Chaos](../anti-patterns/byollm-chaos.md)

---

## Maturity Indicators

| Maturity Level | What It Looks Like |
|---|---|
| 1 — Not started | Teams hold individual API keys, no central routing |
| 2 — Partial | Gateway exists but some teams still call providers directly |
| 3 — Defined | All sanctioned LLM traffic routes through the gateway |
| 4 — Managed | Real-time dashboards, budget alerts, policy-as-code enforced |
| 5 — Optimising | Semantic caching, dynamic model routing, automated compliance evidence generation |

---

*Part of the [CloudKraft AI Control Tower framework](../framework/README.md). See [Layer 1 — Access & Governance](../framework/layer-1-access-governance.md) for the full layer context.*
