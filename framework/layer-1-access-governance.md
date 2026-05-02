# Layer 1 - Access & Governance

**The question this layer answers:** How does the organisation control who can use which AI systems, at what cost, under what policy, with what audit trail?

**Why this layer comes first:** Every other AI governance failure - ungoverned spend, shadow AI proliferation, compliance gaps, inconsistent quality - traces back to ungoverned access. You cannot measure what you cannot see. You cannot govern what you cannot control. Layer 1 is the prerequisite for every other layer in this framework.

**Primary stakeholders:** CISO, FinOps Lead, CTO, Enterprise Architecture

---

## What Good Looks Like at Each Maturity Level

**Level 1 - Initial**
Individual teams or developers hold their own LLM API keys. There is no central visibility into which models are being used, what they cost, or what data is being sent. Shadow AI is present but unmeasured. Compliance obligations are unaddressed.

**Level 2 - Developing**
Some awareness of the access problem. A central team may manage some API keys but teams routinely work around the process. A rough cost estimate exists but is not attributed to teams or use cases. An approved model list exists on paper but is not enforced.

**Level 3 - Defined**
A centralised gateway or proxy routes all sanctioned LLM traffic. Team-level authentication is in place. An approved model list is enforced at the access layer. Cost is attributed per team. An audit log exists and is structured for compliance review. Shadow AI containment policy is documented and communicated.

**Level 4 - Managed**
Real-time cost dashboards show spend per team, per model, and per use case. Rate limiting and budget alerts prevent cost overruns before they happen. Shadow AI detection runs continuously. Policy is enforced as code rather than process. Audit trails have been tested in at least one compliance review.

**Level 5 - Optimising**
Dynamic model governance with risk-tiered access. Automated compliance evidence collection. Shadow AI is detected, classified, and either sanctioned or blocked within hours. FinOps integration provides full AI cost visibility alongside other cloud spend. The AI access layer is treated as critical infrastructure with the same reliability and change management standards as core enterprise systems.

---

## Named Patterns

### Pattern 1 - Centralised AI Gateway

**What it is:** A single proxy service that sits between all enterprise applications and all LLM providers. Every LLM API call routes through the gateway. No application calls a provider directly.

**When to use it:** This is the default pattern for any enterprise with more than three teams using AI. It is the only pattern that provides complete visibility, consistent policy enforcement, and accurate cost attribution without relying on team discipline.

**How it works:**
- Applications send OpenAI-compatible requests to the gateway endpoint
- Gateway authenticates the caller using API keys or IAM identity
- Gateway applies policy (model allowlist, content filters, rate limits, budget checks)
- Gateway routes the request to the appropriate provider (OpenRouter, Azure OpenAI, Bedrock, Ollama)
- Gateway logs the request, response, token count, cost estimate, and metadata
- Gateway returns the response to the calling application

**Trade-offs:**
- Adds one network hop and 10-50ms latency on every request - acceptable for almost all enterprise use cases, not acceptable for real-time voice applications
- Creates a single point of failure that requires high availability design - mitigated by active-active deployment and circuit breakers
- Requires all teams to update their LLM endpoint configuration - a one-time migration cost, not an ongoing burden

**Tools that implement this pattern:**

| Tool | Best For | Limitations |
|---|---|---|
| Portkey | Multi-provider routing, observability, semantic caching | Requires hosted plan for full enterprise features |
| LiteLLM | Self-hosted, OpenAI-compatible, broad model support | Less mature enterprise governance features |
| AWS Bedrock Gateway | AWS-committed organisations | AWS-only, limited multi-cloud |
| Azure AI Foundry | Azure-committed organisations | Azure-only |
| Kong AI Gateway | Existing Kong API management customers | Complex configuration, higher operational overhead |
| Custom FastAPI proxy | Full control, specific compliance requirements | Build and maintenance cost |

**CloudKraft recommendation:** For most regulated Canadian enterprises, Portkey (cloud) or LiteLLM (self-hosted) are the fastest path to a production-grade gateway. Self-hosted LiteLLM is preferred for organisations with strict data residency requirements. Custom FastAPI is recommended only when compliance requirements exceed what packaged tools provide.

---

### Pattern 2 - Federated Access with Central Policy

**What it is:** Teams retain their own LLM integrations but policy is enforced through a central policy-as-code layer that all team implementations must satisfy.

**When to use it:** Organisations where team autonomy is a hard cultural constraint and centralised routing is politically infeasible. Also useful as a transitional pattern while migrating to a centralised gateway.

**Trade-offs:**
- Less complete than centralised gateway - relies on team compliance with policy
- Harder to enforce consistently across teams with different technology stacks
- Cost attribution requires teams to implement logging standards correctly
- Shadow AI is still possible if teams bypass the policy layer

**CloudKraft assessment:** This pattern is a compromise. It is better than no governance and worse than a centralised gateway. We recommend it as a stepping stone, not a destination.

---

### Pattern 3 - Shadow AI Containment

**What it is:** A systematic programme to discover, inventory, classify, and either sanction or block unsanctioned AI usage across the enterprise.

**When to use it:** Every enterprise. Shadow AI is present in every organisation that has not explicitly addressed it. The question is not whether it exists - it is how much it costs and what data it is handling.

**How it works:**
- Network egress analysis to identify traffic to known LLM provider endpoints
- SaaS spend analysis to identify AI tool subscriptions outside IT procurement
- Employee survey and self-declaration (supplemented by, not replaced by, technical detection)
- Classification of discovered AI usage by risk tier
- Formal sanctioning process for legitimate use cases
- Blocking or redirecting of high-risk unsanctioned usage

**Typical findings:** In CloudKraft assessments, shadow AI spend is consistently 2-4x higher than what appears in centralised cost reports. The most common sources are direct OpenAI API keys held by individual developers, AI-enabled SaaS tools not evaluated by security, and ChatGPT Plus or Claude.ai subscriptions expensed by employees.

---

## Tool Selection Matrix - Access & Governance Layer

| Capability | Portkey | LiteLLM | AWS Bedrock | Azure AI Foundry | Custom |
|---|---|---|---|---|---|
| Multi-provider routing | Excellent | Excellent | AWS only | Azure only | Full control |
| Self-hosted option | No | Yes | N/A | N/A | Yes |
| Data residency | Limited | Full | AWS regions | Azure regions | Full control |
| Cost attribution | Good | Good | AWS Cost Explorer | Azure Cost Mgmt | Custom |
| Policy enforcement | Good | Basic | Limited | Limited | Full control |
| Audit logging | Good | Basic | CloudTrail | Azure Monitor | Custom |
| Setup complexity | Low | Medium | Medium | Medium | High |
| Enterprise support | Yes | Community | AWS Support | Microsoft Support | Internal |
| Canadian data residency | Limited | Yes (self-hosted) | Yes (ca-central-1) | Yes (canadacentral) | Yes |

---

## Anti-Patterns

### The OpenAI Key Sprawl

**What happens:** Every developer who needs LLM access gets their own OpenAI API key. Keys are stored in .env files, shared in Slack, committed to git repositories, and never rotated. Finance sees a growing but opaque OpenAI bill. Security has no visibility into what data is being sent. When a developer leaves, their key is forgotten.

**Why it happens:** It is the path of least resistance. Getting your own API key takes five minutes. Getting a centralised gateway approved takes weeks.

**Why it is dangerous:** Beyond cost visibility, the security risk is material. A committed API key costs money every time it is used by anyone who finds it. Data sent to LLMs via unmanaged keys is outside your DLP and data classification controls. When a regulator asks what data your organisation sent to third-party AI systems, the answer is "we don't know."

**The fix:** Centralised gateway with team-based API key management. Existing direct keys are rotated and disabled on a defined date. New direct keys require a security exception.

---

### The Ungoverned Cost Accumulation

**What happens:** AI spend grows 20-30% month over month as new use cases are added. No one has visibility into which teams or use cases are driving the growth. The CFO asks for a breakdown. Nobody can provide one.

**Why it happens:** LLM costs are variable and proportional to usage, not fixed like traditional software licences. Finance and procurement processes are designed for fixed costs and are not equipped to track per-token spend across teams.

**Why it is dangerous:** Ungoverned cost accumulation creates two risks. The obvious one is financial - AI spend can scale faster than value. The less obvious one is strategic - without cost-per-use-case data, you cannot calculate AI ROI, which means you cannot justify continued AI investment to the board.

**The fix:** Per-team, per-use-case cost attribution through the gateway layer. Monthly cost reports with trend analysis. Budget alerts before overruns, not after.

---

### The BYOLLM Chaos

**What happens:** Different teams independently choose different LLM providers, models, and integration approaches. One team uses GPT-4o via direct API. Another uses Claude via Bedrock. A third uses a fine-tuned Llama model hosted on a GPU instance someone provisioned in a personal AWS account. Each has different authentication, different data handling, different costs, and different evaluation approaches.

**Why it happens:** Teams are optimising for their own use case rather than the enterprise platform. In isolation, each choice may be reasonable. In aggregate, the result is ungovernable.

**Why it is dangerous:** You cannot apply consistent governance, evaluation, or cost control across a fragmented AI estate. Each integration point is a potential data leak, a compliance gap, and a maintenance burden.

**The fix:** Centralised gateway with a defined model catalogue. Teams choose from the approved catalogue. Adding a new model goes through an architectural review, not a team decision.

---

## Key Questions for a Client Engagement

These are the questions CloudKraft asks in every Layer 1 assessment. They expose maturity gaps faster than any survey.

1. If I asked you right now how much your organisation spent on AI APIs last month, broken down by team and use case, how long would it take to get that answer?

2. Walk me through what happens when a developer joins your organisation and needs access to an LLM for the first time. Who approves it? How long does it take? What constraints are they given?

3. Can you show me your approved model list? Who approved it? When was it last reviewed?

4. How would you know if a team in your organisation started using an AI tool you had not sanctioned?

5. If a regulator asked you to produce an audit trail of every request your organisation sent to a third-party AI system in the last 90 days, what would you give them?

The quality of the answers to these five questions is a reliable predictor of the overall Layer 1 maturity score.

---

## Relationship to Other Layers

**Layer 1 enables Layer 3 (Orchestration):** The gateway is the enforcement point for orchestration policies - model routing rules, cost budgets, and rate limits that govern multi-step workflows.

**Layer 1 enables Layer 4 (Evaluation & Trust):** Audit logs from the gateway are the primary data source for evaluation and compliance reporting. Without Layer 1, Layer 4 has no data to work with.

**Layer 1 is informed by Layer 2 (Data & Retrieval):** Data classification at the retrieval layer determines which models are permitted to handle which data - a Layer 1 enforcement decision.

**Layer 1 is informed by Layer 5 (Integration):** IAM integration decisions (how the gateway authenticates callers) depend on the enterprise identity architecture documented in Layer 5.

---

*Part of the [CloudKraft AI Control Tower framework](./README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this layer.*
