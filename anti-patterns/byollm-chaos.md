# Anti-Pattern: BYOLLM Chaos

**Layer:** 1 — Access & Governance
**Risk Level:** High
**How Common:** Very Common — the natural outcome when teams choose their own AI stack without an enterprise platform

---

## What Happens

Different teams independently select different LLM providers, models, and integration approaches based on individual use case requirements, personal familiarity, or whoever gave the best demo. Over 12–18 months, the result is an ungovernable AI estate:

- The product team uses GPT-4o via direct OpenAI API with keys held by two developers
- The data engineering team uses Claude via AWS Bedrock with IAM role authentication
- The legal team is running a fine-tuned Llama 3 model on a GPU instance one engineer provisioned in their personal AWS account
- The customer support team bought a third-party AI chatbot platform that uses an undisclosed mix of models
- The security team built an internal tool using Azure OpenAI because "we're Azure-committed"
- The analytics team is experimenting with Mistral via an EU-based provider to test GDPR compliance
- Three separate RAG implementations exist, built on three different vector databases (Pinecone, Qdrant, and pgvector), with three different embedding models, using three different chunking strategies

Each of these choices may be individually defensible. In aggregate, they produce:

- **Zero consistent governance:** No common policy enforcement, no common audit logging, no common access controls
- **Fragmented cost visibility:** Spend across six providers and one SaaS platform cannot be aggregated without manual reconciliation
- **Inconsistent evaluation:** No common quality standards, no common metrics, no common definition of "good enough to ship"
- **Multiplied compliance exposure:** Six distinct vendor relationships, six distinct data processing agreements to negotiate, six distinct sets of data residency claims to verify
- **Unmaintainable fragmentation:** When a model is deprecated, when a provider changes pricing, or when a security vulnerability is discovered, the remediation effort is multiplied by the number of separate integrations

---

## Why It Happens

BYOLLM is a natural consequence of team autonomy combined with a fast-moving AI landscape and an absent enterprise platform. When:

- There is no approved model catalogue
- There is no centralised gateway
- There is no enterprise AI platform team
- Each team is responsible for its own AI outcomes

...each team makes the locally optimal decision. GPT-4o is better for some tasks. Claude is better for others. Bedrock makes sense if you are an AWS-committed shop. None of these decisions are wrong in isolation. The problem is the aggregate.

The pattern also frequently stems from a genuine gap: the enterprise platform team moves too slowly to serve the pace of AI adoption. Teams that cannot get access to approved tools within a reasonable timeframe will build their own path. BYOLLM is often a symptom of governance processes that cannot keep up with technology velocity.

---

## Why It Is Dangerous

**Security surface area multiplication:** Every provider integration is a potential data exfiltration path. Six providers mean six attack surfaces, six sets of credentials to manage, six incident response playbooks to write when something goes wrong.

**Compliance evidence fragmentation:** When a regulator asks "which AI systems did you use to process customer data, and what data did each one receive?", the answer requires coordinating across six separate systems with six separate logging formats. This is a material audit risk.

**Model governance failure:** When a model needs to be removed from use (deprecated, security advisory, regulatory prohibition), you need to know every place it is used. With BYOLLM, that inventory does not exist. You cannot govern what you cannot see.

**Team knowledge silos:** A team that built their own OpenAI integration cannot benefit from improvements made by the team that built their Bedrock integration. Platform improvements are not shared. Mistakes are repeated. Every team carries its own maintenance burden for what should be shared infrastructure.

**Vendor negotiation fragmentation:** LLM providers offer meaningful volume discounts for committed usage. When usage is split across six providers with six separate accounts, you negotiate from a position of low volume with every vendor. A consolidated spend picture could unlock 20–40% cost reduction through enterprise agreements.

---

## Observable Symptoms

- A senior engineer cannot enumerate all the AI providers and models the organisation uses
- AI vendor invoices come from more than two or three providers
- Different teams use the same model differently — different prompt structures, different chunking strategies, different evaluation approaches
- The same capability has been built independently by multiple teams
- There is no "approved model list" — or there is a list but it is advisory rather than enforced
- Adding a new LLM model to production requires no review or approval process

---

## The Fix

**The fix is a centralised AI gateway combined with an approved model catalogue.** This is not the same as restricting teams to a single model or eliminating team autonomy. Teams can still use the model best suited to their use case — but they access it through the enterprise gateway, and the model must be on the approved catalogue.

**Step 1 — Inventory the current state**
Document every LLM provider, model, and integration currently in use across the organisation. This is the discovery work. Include the team responsible, the use case, the data classification of inputs, and the approximate monthly spend. This inventory is uncomfortable to produce and essential to have.

**Step 2 — Define an approved model catalogue**
Based on the inventory, define which models and providers are approved for which data classification tiers. A reasonable starting catalogue for a regulated Canadian enterprise:

| Data Tier | Approved Models | Approved Providers |
|---|---|---|
| Public / internal non-sensitive | GPT-4o, Claude Sonnet, Gemini Pro | OpenAI, Anthropic, Google |
| Internal sensitive | Claude via Bedrock, GPT-4o via Azure OpenAI | AWS (ca-central-1), Azure (canadacentral) |
| Regulated / confidential | Self-hosted Llama 3 or Mistral, Azure OpenAI with no logging | Self-hosted or Azure with DPA in place |

**Step 3 — Deploy the centralised gateway**
Deploy LiteLLM self-hosted or Portkey to serve as the single access point for all models in the approved catalogue. Teams migrate to the gateway endpoint. Direct provider access requires a security exception.

**Step 4 — Sunset unapproved integrations**
Give teams a defined migration window (typically 60–90 days) to migrate to the gateway. After the window, direct provider credentials not managed through the gateway are identified as a policy violation.

**Step 5 — Establish a model addition process**
Define a lightweight process for adding new models to the catalogue: security review (data handling, residency, terms), architecture review (does this add meaningful capability not available in existing catalogue), FinOps review (cost impact). Target a 10-business-day turnaround for standard model additions.

---

## What Not to Do

**Do not attempt to standardise on a single model.** Different models genuinely perform differently on different tasks. The goal is centralised access and governance, not model monoculture. Claude and GPT-4o can both be in the approved catalogue — they just both route through the gateway.

**Do not try to migrate all integrations simultaneously.** Prioritise by risk: highest data classification first, highest spend second, highest-traffic third. A phased migration over 90 days is more realistic and less disruptive than a "big bang" migration.

---

## Related

- **Remediation pattern:** [Centralised AI Gateway](../patterns/centralised-ai-gateway.md)
- **Related anti-patterns:** [Shadow AI Sprawl](./shadow-ai-sprawl.md), [Ungoverned Cost Accumulation](./ungoverned-cost-accumulation.md)
- **Framework layer:** [Layer 1 — Access & Governance](../framework/layer-1-access-governance.md)

---

*Part of the [CloudKraft AI Control Tower framework](../framework/README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this and related governance gaps.*
