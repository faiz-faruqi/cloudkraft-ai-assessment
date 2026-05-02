# Anti-Pattern: Ungoverned Cost Accumulation

**Layer:** 1 — Access & Governance
**Risk Level:** High
**How Common:** Universal — present in every organisation without per-team cost attribution

---

## What Happens

AI spend grows month over month as new use cases are added, existing use cases scale, and token consumption increases with model capability upgrades. No one has visibility into which teams or use cases are driving the growth. When the CFO asks for a cost breakdown, the answer is a single line item from the LLM provider's invoice with no attribution below the organisation level.

The pattern has a predictable progression:

**Month 1–3:** AI spend is small and treated as an experiment budget. No one asks for attribution. Finance sees a modest new line item.

**Month 4–6:** A few use cases go to production. Spend grows 40–60% month over month. Finance notices the growth rate and asks for a forecast. The answer is "we're not sure — it depends on usage."

**Month 7–12:** Multiple teams have production workloads. Monthly spend is in the tens or hundreds of thousands. Finance is now asking for ROI data. Engineering cannot provide it because they cannot attribute cost to outcomes. The CFO considers cutting AI investment. The AI team cannot defend its budget because the numbers do not exist.

In one CloudKraft engagement with a mid-sized financial institution, the team discovered $340,000 in annual AI spend that had never been attributed to any business use case or team. No one could determine which spend was generating value and which was orphaned from cancelled projects.

---

## Why It Happens

LLM cost is fundamentally different from traditional software cost. Software licences are fixed, predictable, and easy to allocate. LLM API spend is variable, proportional to usage, and billed at a granularity (per token) that does not map naturally to existing cost allocation processes.

The compounding factors:

- **Shared API keys** make team-level attribution impossible without gateway-layer tagging
- **FinOps processes** are designed for compute, storage, and SaaS licences — not per-token variable API spend
- **Development and experimentation** usage is commingled with production usage in cost reports
- **Model upgrades** silently increase cost per query when teams switch to more capable (more expensive) models without cost impact analysis
- **Prompt engineering changes** that add context or few-shot examples can increase token count significantly without anyone noticing

---

## Why It Is Dangerous

**Direct financial risk:** AI spend at scale is material. A production workflow running GPT-4o at 5,000 queries per day at $0.05 average cost per query is $75,000 per month. This is not unusual for enterprise AI at moderate scale. Without attribution, this spend cannot be controlled.

**Strategic risk:** Without cost-per-use-case data, you cannot calculate ROI. Without ROI data, you cannot justify continued AI investment to the board. Ungoverned cost accumulation does not just create a financial problem — it creates a credibility problem for the entire AI programme.

**Budget overrun risk:** LLM spend can scale faster than anticipated. An event-triggered workflow that processes a high-volume event source (email inbound, document uploads, database changes) can generate unexpected costs within hours of a volume spike. Without budget alerts, the first signal is the monthly invoice.

**Vendor negotiation weakness:** LLM providers offer significant volume discounts and enterprise agreements. Organisations without clean usage data cannot negotiate effectively because they cannot demonstrate predictable consumption volumes.

---

## Observable Symptoms

- The monthly OpenAI / Anthropic / AWS Bedrock bill is a single number
- No one can answer "what did it cost to run use case X last month?"
- Development, staging, and production LLM usage are billed to the same account with no separation
- AI costs are in the "miscellaneous cloud spend" budget category
- Multiple teams are surprised when informed of the total monthly AI spend
- No budget alerts exist for AI spend — the first signal of an overrun is the invoice
- ROI discussions about AI use cases stall because cost data does not exist

---

## The Fix

**The root cause is almost always a missing gateway.** Every fix described here requires per-request metadata that only a gateway layer can provide systematically.

**Step 1 — Deploy cost attribution at the gateway layer**
Configure the gateway (LiteLLM, Portkey, or equivalent) to tag every request with:
- `team_id` — the team making the request
- `use_case_id` — the specific workflow or application
- `environment` — production / staging / development

Use virtual API keys scoped to team + environment to enforce these tags at authentication time rather than relying on applications to self-report.

**Step 2 — Build a cost dashboard**
Connect gateway audit logs to a BI tool (Looker, Power BI, Tableau, Metabase). Publish a weekly cost report to team leads showing:
- Spend by team this month vs. last month
- Spend by model
- Top 5 use cases by token consumption
- Month-over-month growth rate

**Step 3 — Set budget alerts**
Configure per-team and per-use-case budget thresholds with automated alerts at 70%, 90%, and 100% of monthly budget. Alert the team lead and their manager. At 100%, throttle the team's requests rather than allowing unlimited overrun — this changes the incentive structure.

**Step 4 — Separate development from production**
Development and experimentation should run against a separate budget with a hard monthly cap (e.g., $500/team/month). This prevents runaway experimentation from inflating production cost reports and gives developers a clear budget to work within.

**Step 5 — Add cost to the deployment checklist**
Every new AI use case going to production must include a cost estimate: expected queries per day × average token count × model cost per token. This estimate becomes the baseline for the initial budget alert threshold.

---

## Cost Optimisation Once Attribution Exists

Attribution is the prerequisite for optimisation. Once you know where cost is coming from, common levers include:

- **Model right-sizing:** Use GPT-4o-mini, Haiku, or Llama for tasks that do not require frontier-model capability. Cost differences are 10–50x between model tiers.
- **Semantic caching:** Cache responses to near-duplicate queries (Portkey semantic cache, GPTCache). Effective for FAQ-style use cases with query pattern repetition.
- **Prompt compression:** Remove unnecessary context from prompts. A 20% reduction in average prompt length is a 20% reduction in input token cost.
- **Batch processing:** Use the OpenAI Batch API or Anthropic Message Batches for non-real-time workflows. Cost is 50% of synchronous rates.
- **Retrieval trimming:** Reducing context passed to the LLM from 10 chunks to 5 chunks reduces token cost without necessarily reducing quality — validate with evaluation.

---

## Related

- **Remediation pattern:** [Centralised AI Gateway](../patterns/centralised-ai-gateway.md)
- **Related anti-patterns:** [Shadow AI Sprawl](./shadow-ai-sprawl.md), [BYOLLM Chaos](./byollm-chaos.md)
- **Framework layer:** [Layer 1 — Access & Governance](../framework/layer-1-access-governance.md)

---

*Part of the [CloudKraft AI Control Tower framework](../framework/README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this and related governance gaps.*
