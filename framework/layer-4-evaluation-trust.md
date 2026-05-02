# Layer 4 - Evaluation & Trust

**The question this layer answers:** How does the organisation know its AI systems are working correctly today, and how will it know when they stop working?

**Why this layer matters:** AI systems degrade silently. A RAG pipeline that was accurate in January may be returning stale, incorrect, or hallucinated answers in June because the knowledge base was not updated, the embedding model changed, or the document corpus drifted. Without systematic evaluation, the first signal of quality degradation is a user complaint, a compliance incident, or a board-level embarrassment.

**Primary stakeholders:** Head of AI, Risk and Compliance, Data Science, Product

---

## What Good Looks Like at Each Maturity Level

**Level 1 - Initial**
No evaluation framework. Quality is assumed based on qualitative impressions during development. Systems go live when the development team feels they are ready. Quality degradation is discovered through user complaints.

**Level 2 - Developing**
Informal testing before release. Some teams maintain a small set of example queries they check manually before deploying changes. No production monitoring. No baseline metrics.

**Level 3 - Defined**
A structured evaluation test set exists. Automated evaluation runs before deployment. At least one quality metric (answer faithfulness, context relevance, or task completion rate) is tracked over time. A clear definition of what constitutes a quality regression that blocks deployment.

**Level 4 - Managed**
Continuous evaluation in production with weekly or more frequent automated measurement. Alerting on metric regression. RAGAS or equivalent framework applied systematically. Human feedback collected and incorporated into evaluation. Cost-per-query tracked alongside quality metrics.

**Level 5 - Optimising**
Real-time drift detection. Business-outcome metrics (task completion, escalation rate, user satisfaction scores) tracked alongside technical metrics. Evaluation data used to drive model selection, prompt engineering, and retrieval tuning decisions. Third-party AI risk assessed using the same evaluation framework.

---

## Named Patterns

### Pattern 1 - Eval-Before-Deploy Gate

**What it is:** A mandatory automated evaluation run as part of the deployment pipeline. If evaluation metrics fall below a defined threshold, deployment is blocked. This is the AI equivalent of a passing test suite as a deployment prerequisite.

**Why enterprises resist it:** It requires an evaluation baseline, which requires time to build. Teams under delivery pressure treat it as a future concern. The result is systems deployed without any quality baseline, making regression detection impossible.

**Implementation:**
- Define a golden dataset of representative queries with human-labelled expected answers (minimum 50 examples, ideally 200+)
- Run RAGAS or equivalent evaluation against the golden dataset before every deployment
- Define threshold values for key metrics (faithfulness > 0.85, answer relevance > 0.80)
- Block deployment if thresholds are not met
- Review and update the golden dataset quarterly or when the knowledge domain changes significantly

**Trade-offs:**
- Building the initial golden dataset requires significant human effort - 20-40 hours for a well-designed 100-question set
- Thresholds must be calibrated to the specific use case - a 0.85 faithfulness threshold is reasonable for a policy FAQ, potentially insufficient for a legal document review system
- False positives (blocking a good deployment due to edge cases in the golden dataset) require a human override process

---

### Pattern 2 - Continuous Production Evaluation

**What it is:** Automated evaluation that runs against production traffic on a regular schedule, measuring quality on real queries rather than a static golden dataset.

**When to use it:** Any system in production with more than 50 queries per day. Static golden datasets cannot capture the full distribution of user queries, especially for systems where usage patterns evolve over time.

**Implementation approaches:**
- LLM-as-judge: use a separate LLM to evaluate the quality of responses from the production system. Requires careful prompt engineering and validation that the judge model's assessments correlate with human judgments.
- User signal collection: implicit signals (query reformulation, escalation rates, session abandonment) and explicit signals (thumbs up/down, corrections) collected from the production interface.
- Sampling-based human review: a random sample of production queries reviewed by human evaluators on a weekly cadence.

**Tools:**

| Tool | Approach | Strengths | Weaknesses |
|---|---|---|---|
| LangSmith | LLM-as-judge, dataset management | Strong LangChain integration, good UI | LangChain ecosystem dependency |
| Langfuse | Open source, LLM-as-judge, tracing | Self-hostable, active development | Less mature than LangSmith |
| Arize AI | ML observability, drift detection | Strong production monitoring | Higher complexity, MLOps background required |
| Datadog LLM Observability | Infrastructure teams already using Datadog | Single pane of glass | Less RAG-specific than purpose-built tools |
| RAGAS | Metric framework | Open source, RAG-specific metrics | Framework only, not an observability platform |
| Custom | Full control | No vendor dependency | Build and maintenance cost |

**CloudKraft recommendation:** LangSmith for teams already in the LangChain/LangGraph ecosystem - it integrates evaluation, tracing, and dataset management in one tool. Langfuse for organisations requiring self-hosted evaluation with strong data residency requirements. RAGAS as the metric framework regardless of which observability platform you use - its metrics (faithfulness, answer relevance, context precision, context recall) are the industry standard for RAG evaluation.

---

### Pattern 3 - Drift Detection

**What it is:** Automated monitoring that detects when AI system behaviour has changed significantly from its baseline, triggering investigation before users are affected.

**What causes drift in enterprise AI systems:**
- Knowledge base staleness: source documents updated without re-ingestion
- Model updates: LLM provider silently updates the model behind the same endpoint
- Query distribution shift: user queries evolve over time, moving outside the distribution the system was evaluated on
- Embedding model changes: re-embedding with a different model changes retrieval behaviour

**Implementation:**
- Maintain rolling evaluation against a representative query sample
- Alert when key metrics move more than one standard deviation from the 30-day baseline
- Treat any model version change by a provider as a trigger for a full evaluation run

---

## Tool Selection Matrix - Evaluation & Trust Layer

### Evaluation Frameworks

| Framework | Best For | Key Metrics | Self-Hosted |
|---|---|---|---|
| RAGAS | RAG-specific evaluation | Faithfulness, answer relevance, context precision, context recall | Yes |
| DeepEval | General LLM evaluation, many metrics | Hallucination, bias, toxicity, custom | Yes |
| TruLens | LLM application evaluation, Snowflake integration | Groundedness, context relevance, answer relevance | Yes |
| PromptFoo | Prompt testing and red-teaming | Custom assertions, LLM-as-judge | Yes |

### Observability Platforms

| Platform | Best For | Hosting | Canadian Data Residency |
|---|---|---|---|
| LangSmith | LangChain ecosystem teams | Cloud | Limited |
| Langfuse | Self-hosted requirements | Cloud or self-hosted | Yes (self-hosted) |
| Arize AI | ML observability maturity | Cloud | Limited |
| Datadog LLM | Existing Datadog customers | Cloud | Yes (AWS ca-central-1) |

---

## Anti-Patterns

### Shipping Without an Evaluation Baseline

**What happens:** A system goes live with no measurement of its quality at launch. Three months later, a user reports that the system is giving wrong answers. The team has no way to know whether this is a regression from a previously working state or a bug that has been present since launch.

**Why it is dangerous:** Without a baseline, you cannot distinguish degradation from original failure. You cannot measure the impact of improvements. You cannot demonstrate to regulators that the system met quality standards at deployment.

**The fix:** Every system must have an evaluation baseline established before the first production deployment. No exceptions. The baseline does not need to be perfect - it needs to exist.

---

### Treating LLM Outputs as Deterministic

**What happens:** The team tests a prompt, gets a good result, and ships it. They assume the prompt will produce the same quality of output consistently. In production, the system produces high-quality outputs 70% of the time and plausible but incorrect outputs 30% of the time. Because the failures are plausible, they are not immediately obvious.

**Why it is dangerous:** LLM outputs are probabilistic, not deterministic. Temperature, context window variations, and model updates all affect output consistency. A system that appears to work in testing may behave unpredictably in production without systematic evaluation across a representative query distribution.

**The fix:** Evaluate across a distribution of queries, not a small set of handpicked examples. Use automated evaluation metrics rather than relying on human judgment of a handful of test cases.

---

### No Cost-Per-Query Visibility

**What happens:** The team optimises for answer quality without tracking the cost of achieving it. A retrieval pipeline that fetches 20 documents and passes them all to GPT-4o costs $0.15 per query. At 10,000 queries per day, that is $1,500/day. Nobody noticed because cost tracking was not part of the evaluation framework.

**The fix:** Track cost-per-query alongside quality metrics from the first day of production. Optimise for the best quality at an acceptable cost, not the best quality without cost constraints.

---

## Key Questions for a Client Engagement

1. Show me the evaluation results from the last deployment of your most important AI system. What metrics did you measure? What thresholds did they need to meet?

2. How would you know if the quality of your AI system degraded by 20% over the next 30 days? What would tell you?

3. What is your AI system's answer faithfulness score? If you do not know what that means or have not measured it, how do you know the system is not hallucinating?

4. When your LLM provider last updated their model, what did you do to verify that your system's quality was maintained?

5. What is the cost per query for your most-used AI workflow? Is that acceptable given the business value it delivers?

---

## Relationship to Other Layers

**Layer 4 depends on Layer 1:** Audit logs from the gateway provide the raw data for evaluation and compliance reporting. Without Layer 1 telemetry, evaluation is limited to synthetic test sets rather than production behaviour.

**Layer 4 depends on Layer 2:** Retrieval quality metrics (context precision, context recall) require visibility into what documents were retrieved for each query - data that must be captured at the retrieval layer.

**Layer 4 depends on Layer 3:** Orchestration execution data (step latencies, token counts, failure rates) feeds the evaluation and observability pipeline.

**Layer 4 informs Layer 1:** Evaluation results inform model selection decisions - if a model consistently underperforms on a specific use case, the Layer 1 model allowlist should be updated accordingly.

---

*Part of the [CloudKraft AI Control Tower framework](./README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this layer.*
