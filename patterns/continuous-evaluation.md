# Pattern: Continuous Evaluation

**Layer:** 4 — Evaluation & Trust
**Problem:** AI systems degrade silently. A RAG pipeline accurate at deployment may return stale, hallucinated, or low-quality answers six months later — and you will not know until a user complaint, a compliance incident, or a board-level embarrassment surfaces it.
**Solution:** An automated evaluation pipeline that runs against production traffic on a regular schedule, tracks quality metrics over time, and alerts on regressions before users are affected.

---

## Context

Evaluation-before-deploy is the minimum. Continuous evaluation in production is what separates mature AI programmes from those that are perpetually surprised by quality problems.

The failure mode is predictable: a system is evaluated during development, performs well on the test set, ships to production, and is never evaluated again. Three months later:
- The LLM provider quietly updated the model behind the same API endpoint
- Fifteen source documents in the knowledge base were superseded and the old versions were not removed
- Users have evolved their query patterns in ways that were not represented in the original test set
- A prompt that worked well at GPT-4-turbo performance characteristics behaves differently on the current model

Without continuous evaluation, none of these changes produce an alert. They produce a gradual quality decline that users eventually notice and stop trusting.

---

## How It Works

```
Production Traffic
        ↓
  Sampling Layer (5–20% of queries)
        ↓
  Evaluation Pipeline
  ├─ LLM-as-Judge scoring (faithfulness, relevance)
  ├─ RAGAS metrics (context precision, context recall)
  └─ User signal collection (thumbs, corrections, escalations)
        ↓
  Metrics Store (time-series)
        ↓
  Dashboard + Alerting
  (regression alert if metric drops > X% from 30-day baseline)
        ↓
  Evaluation Report (weekly)
```

---

## Three Evaluation Modes

**Mode 1 — Pre-deployment gate**
Runs against a static golden dataset before every deployment. Blocks deployment if metrics fall below defined thresholds. This is the minimum viable evaluation practice. It catches regressions caused by prompt changes, retrieval pipeline changes, or model updates.

**Mode 2 — Scheduled production evaluation**
Runs on a fixed schedule (daily or weekly) against a sample of real production queries. Uses LLM-as-judge to score quality on queries where you do not have human-labelled ground truth. Catches drift caused by changing user behaviour, knowledge base staleness, and silent model updates.

**Mode 3 — Real-time user signal monitoring**
Captures implicit signals (query reformulation immediately after a response suggests dissatisfaction, escalation rate, session abandonment) and explicit signals (thumbs up/down, corrections submitted). These signals are leading indicators — they deteriorate before technical metrics do.

In practice, all three modes are needed. The pre-deployment gate catches intentional changes. Scheduled evaluation catches environmental drift. User signals catch the failure modes that formal metrics miss.

---

## Core Metrics

### RAG-Specific Metrics (RAGAS framework)

| Metric | What It Measures | Target Threshold |
|---|---|---|
| **Answer Faithfulness** | What fraction of the answer is grounded in retrieved context? | > 0.85 |
| **Answer Relevance** | How relevant is the answer to the original question? | > 0.80 |
| **Context Precision** | What fraction of retrieved context was actually used? | > 0.70 |
| **Context Recall** | What fraction of needed information was in the retrieved context? | > 0.75 |

Thresholds are starting points. Calibrate to your specific use case: a legal document review system warrants higher faithfulness thresholds than an internal FAQ.

### Operational Metrics

| Metric | What It Measures | Alert Condition |
|---|---|---|
| **Escalation rate** | Fraction of queries routed to human review | Increase > 20% week-over-week |
| **Cost per query** | LLM token cost / number of queries | Increase > 30% month-over-month |
| **p95 latency** | 95th percentile end-to-end response time | Exceeds SLA threshold |
| **Error rate** | Fraction of queries returning errors | > 1% sustained |
| **User correction rate** | Fraction of outputs modified by human reviewers | Increase > 15% week-over-week |

---

## LLM-as-Judge Implementation

For production queries without human-labelled ground truth, use a separate LLM to evaluate response quality. This is not a substitute for human evaluation — it is a scalable proxy that catches obvious regressions and keeps you informed between human review cycles.

**Judge prompt structure (faithfulness example):**

```
You are evaluating whether an AI assistant's response is faithful to the provided source documents.

Source documents:
{retrieved_context}

User question:
{question}

AI response:
{response}

Score the faithfulness of the response on a scale of 0 to 1, where:
1.0 = every claim in the response is directly supported by the source documents
0.5 = most claims are supported but some are inferred or extrapolated
0.0 = the response contains claims not supported by the source documents

Return a JSON object: {"score": <float>, "reasoning": "<one sentence>"}
```

**Validation requirement:** Before using LLM-as-judge in production, validate that the judge model's scores correlate with human evaluator scores on a sample of 100+ queries. A judge model that disagrees with humans on quality is worse than no judge. Correlation coefficient of > 0.75 is a reasonable minimum bar.

---

## Tool Selection

### Evaluation Frameworks

| Tool | Type | Strengths | Weaknesses |
|---|---|---|---|
| RAGAS | Metric framework | RAG-specific, well-validated metrics, open source | Framework only — you provide the evaluation runner |
| DeepEval | Full evaluation framework | Many metrics, CI/CD integration, good documentation | Less RAG-specific than RAGAS |
| TruLens | Full evaluation framework | Snowflake integration, grounding metrics | Smaller community |
| PromptFoo | Prompt testing + red-teaming | Excellent for systematic prompt regression testing | Not optimised for production monitoring |

### Observability Platforms

| Platform | Best For | Hosting | Canadian Residency |
|---|---|---|---|
| Langfuse | Self-hosted requirements, LLM tracing, dataset management | Cloud or self-hosted | Yes (self-hosted) |
| LangSmith | LangChain/LangGraph ecosystems | Cloud | Limited |
| Arize AI | Teams with existing ML observability maturity | Cloud | Limited |
| Datadog LLM Observability | Teams already on Datadog | Cloud | Yes (AWS ca-central-1) |
| Custom time-series (Prometheus + Grafana) | Full control, infrastructure teams | Self-hosted | Yes |

**CloudKraft recommendation:** Langfuse self-hosted for regulated Canadian enterprises — it provides LLM tracing, evaluation dataset management, and user feedback collection in a single tool with full data residency. Pair with RAGAS as the metric computation framework. LangSmith for teams already deeply invested in the LangChain ecosystem who do not have strict data residency requirements.

---

## Golden Dataset Management

A golden dataset is a curated set of representative queries with human-labelled expected answers. It is the foundation of the pre-deployment gate and a key reference for evaluating drift.

**Building the initial dataset:**
- Minimum 50 examples; 200+ for production systems
- Representative of the full distribution of user queries — do not over-sample easy questions
- Include adversarial examples: questions the system should decline, questions at the edge of the knowledge domain, ambiguous questions with multiple valid answers
- Label expected answers with the source documents that support them — this enables context recall measurement

**Maintaining the dataset:**
- Review and update quarterly, or whenever the knowledge domain changes significantly
- Add examples from real production queries that exposed failures or near-misses
- Remove examples that are no longer representative of current user behaviour
- Treat the golden dataset as a version-controlled artifact — every change should be tracked and reviewed

---

## Drift Detection

Silent model updates by LLM providers are one of the most underappreciated risks in enterprise AI. When OpenAI, Anthropic, or AWS updates a model behind the same endpoint, your system's behaviour changes without any action on your part. Some updates improve quality. Some do not.

**Detection approach:**
- Maintain a 30-day rolling baseline for each core metric
- Alert when any metric deviates more than one standard deviation from the baseline
- Treat any confirmed model version change as a trigger for a full evaluation run against the golden dataset
- Monitor provider change logs and release notes — subscribe to status pages and developer changelogs

---

## Evaluation Report Template

Publish a weekly evaluation report to stakeholders. Keep it short:

```
Week of [date]

System: [name]
Queries evaluated: [N] (production sample)

Core metrics (vs. prior week):
  Answer Faithfulness:  0.87  (▲ +0.02)
  Answer Relevance:     0.83  (— 0.00)
  Context Precision:    0.74  (▼ -0.03)  ← watch
  Context Recall:       0.79  (▲ +0.01)

Escalation rate:  4.2%  (▲ +0.8pp)
Cost per query:   $0.018  (— flat)

Notable: Context Precision declined 3pp. Investigation in progress.
         Source: 12 documents in the Policy corpus were updated
         on [date] and not yet re-ingested.
```

---

## Relationship to Other Patterns

**Depends on:**
- [Centralised AI Gateway](./centralised-ai-gateway.md) — gateway audit logs are the primary data source for production evaluation
- [Hybrid RAG](./hybrid-rag.md) — context precision and context recall require retrieval metadata from the RAG pipeline
- [Human-in-the-Loop](./human-in-the-loop.md) — reviewer corrections are ground-truth labels for evaluation

**Feeds into:**
- Model selection decisions (if a model consistently underperforms on evaluation, update the gateway allowlist)
- Retrieval tuning (if context precision is low, the chunking strategy or re-ranker needs adjustment)
- Prompt engineering cycles (if faithfulness is declining, the generation prompt may need reinforcement)

---

*Part of the [CloudKraft AI Control Tower framework](../framework/README.md). See [Layer 4 — Evaluation & Trust](../framework/layer-4-evaluation-trust.md) for the full layer context.*
