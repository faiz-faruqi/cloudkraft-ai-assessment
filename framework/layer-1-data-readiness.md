# Layer 1 - Data Readiness

**The question this layer answers:** Is the data feeding your AI systems accurate, traceable, current, and safe to expose - before a model ever sees it?

**Why this layer comes first:** Every downstream AI failure - hallucination, stale answers, compliance exposure, silent quality drift - traces back to what the model was given to work with. You cannot govern retrieval, orchestration, or trust if the underlying data was never made trustworthy. Layer 1 is the prerequisite for every other layer in this framework.

**Primary stakeholders:** CDO, Data Architecture, Data Engineering, Head of AI

---

## What Good Looks Like at Each Maturity Level

**Level 1 - Initial**
AI systems consume whatever data happens to be available, with no quality checks, no lineage, and no classification. Embeddings are generated ad hoc per project in whatever vector store a developer chose. Nobody can say where a given AI answer's source content came from.

**Level 2 - Developing**
Quality issues are found reactively, usually when an AI output looks obviously wrong. Lineage is known informally by the team that built the pipeline but is not documented. A vector store exists but has no versioning or access control.

**Level 3 - Defined**
Manual quality reviews happen before major AI data sources go live. Lineage is documented for major sources, though not always current. Data contracts exist for some sources with manual change notification. Sensitive data is classified for major sources with manual masking applied.

**Level 4 - Managed**
Automated data quality checks run on a schedule with defined thresholds. Lineage tracking is automated for most AI-accessible data. Vector stores are versioned, access-controlled, and refreshed on a defined schedule. Automated classification and masking run before data reaches AI-accessible stores.

**Level 5 - Optimising**
Continuous automated quality monitoring gates ingestion into every AI-accessible store. End-to-end lineage runs from source system through vector store to AI output, queryable for audit and impact analysis. Data contracts are enforced as code - breaking changes are blocked before they reach an AI pipeline. Vector store governance includes drift detection and automated re-indexing.

---

## Named Patterns

### Pattern 1 - Governed Data Contract Layer

**What it is:** A formal, versioned contract between each data-producing system and the AI pipelines that consume it, enforced through schema validation rather than tribal knowledge.

**When to use it:** Any organisation where more than one team owns data that AI systems depend on. It is the only pattern that prevents an upstream schema change from silently breaking a downstream AI system.

**How it works:**
- Data producers publish a schema and quality contract alongside the data itself
- AI pipelines declare which contract version they depend on
- Schema validation runs on every ingestion; violations block the pipeline rather than corrupting the AI-accessible store
- Breaking changes require a version bump and a migration window, not a silent overwrite

**Trade-offs:**
- Adds process overhead for data producers - acceptable for any source feeding a production AI system, excessive for one-off experiments
- Requires organisational buy-in from teams who do not think of themselves as "data producers for AI" - the biggest adoption blocker in practice
- Strict contracts can slow down legitimate schema evolution if change management is too heavyweight

**Tools that implement this pattern:**

| Tool | Best For | Limitations |
|---|---|---|
| dbt (contracts + tests) | Warehouse-native data feeding AI pipelines | Requires a dbt-modelled warehouse layer |
| Great Expectations | Flexible quality and schema validation across sources | Requires engineering effort to integrate into pipelines |
| Soda | Declarative data quality checks with alerting | Best for tabular/warehouse data, weaker for unstructured content |
| Monte Carlo / Bigeye | Automated data observability and anomaly detection | Cost scales with data volume; overkill for small AI programmes |
| Custom validation layer | Full control, specific compliance requirements | Build and maintenance cost |

**CloudKraft recommendation:** For organisations with a warehouse-centric data stack, dbt contracts plus Great Expectations checks are the fastest path to enforceable data contracts. For unstructured document sources feeding RAG systems, a custom validation layer at the ingestion boundary is usually required - no packaged tool fully covers both structured and unstructured AI inputs today.

---

### Pattern 2 - Vector Store Governance

**What it is:** Treating the vector store as a governed system of record rather than a disposable cache - versioned, access-controlled, and traceable back to source documents.

**When to use it:** Every organisation running a production RAG system. An ungoverned vector store is the most common source of stale or untraceable AI answers.

**Trade-offs:**
- Versioning and re-indexing add storage and compute cost - justified once a RAG system is in production and answers are relied upon
- Access control at the vector store level requires the same rigour as access control on the source documents - easy to under-invest in because it feels like "just an index"

**CloudKraft assessment:** Teams routinely govern the source document store carefully and then treat the derived vector index as an afterthought. The index deserves the same governance as the data it was built from.

---

### Pattern 3 - Automated Quality Gating

**What it is:** Quality checks that run as a gate, not a report - failing data is blocked from reaching an AI-accessible store rather than flagged after the fact.

**When to use it:** Any AI system where a wrong or stale answer has real business or compliance consequences.

**Typical findings:** In CloudKraft assessments, organisations without automated gating typically discover data quality problems only after an AI output is questioned by a user or auditor - by which point the bad data has often already propagated into multiple downstream systems.

---

## Tool Selection Matrix - Data Readiness Layer

| Capability | dbt | Great Expectations | Monte Carlo | Collibra / Atlan | Custom |
|---|---|---|---|---|---|
| Structured data contracts | Excellent | Good | Limited | Good (catalog-level) | Full control |
| Unstructured/document validation | No | Limited | Limited | Limited | Full control |
| Lineage tracking | Good (warehouse) | No | Good | Excellent | Custom |
| Automated anomaly detection | No | Limited | Excellent | Limited | Custom |
| Vector store integration | No | No | No | No | Yes (build required) |
| Setup complexity | Low-Medium | Medium | Medium | Medium-High | High |
| Canadian data residency | Depends on warehouse | Self-hosted option | Limited | Depends on plan | Full control |

---

## Anti-Patterns

### The Ungoverned Embedding Pipeline

**What happens:** Documents are chunked and embedded into a vector store with no record of which source document, version, or chunking strategy produced each vector. When an AI answer turns out to be wrong or outdated, nobody can trace it back to a specific source or determine whether the fix requires re-embedding.

**Why it happens:** Getting a RAG prototype working is fast if you skip lineage. Retrofitting it later means re-processing the entire corpus.

**Why it is dangerous:** Without lineage, you cannot selectively update stale content, cannot prove to an auditor what data an AI answer was grounded in, and cannot debug quality regressions with any precision.

**The fix:** Every embedding job records source document ID, version, chunking parameters, and embedding model version alongside the vector. Re-embedding is triggered by source changes, not by manual discovery of staleness.

---

### Stale Vector Drift

**What happens:** The vector store is populated once at project launch and never systematically resynced with the source-of-truth systems. Months later, the AI confidently answers questions using content that has since been corrected, superseded, or deleted upstream.

**Why it happens:** Re-embedding is treated as a one-time migration task rather than an ongoing operational process.

**Why it is dangerous:** Users trust AI answers as current by default. Silent staleness is worse than an obvious error because nobody thinks to question it.

**The fix:** Automated freshness tracking with defined re-sync cadence, and staleness alerts when a source document changes without a corresponding re-embed.

---

### The Schema-less Ingestion

**What happens:** AI pipelines ingest data directly from source systems with no contract. When an upstream team changes a field name or format, the AI pipeline breaks silently or, worse, keeps running on corrupted data.

**Why it happens:** Nobody owns the interface between the data-producing team and the AI-consuming team.

**Why it is dangerous:** Silent corruption is far more damaging than a loud failure - it erodes trust in AI outputs long before anyone notices the root cause.

**The fix:** A governed data contract layer with automated schema validation at the ingestion boundary, and a defined owner on both sides of every AI-facing data interface.

---

## Key Questions for a Client Engagement

These are the questions CloudKraft asks in every Layer 1 assessment. They expose maturity gaps faster than any survey.

1. If an AI system gave a wrong answer today, how would you trace it back to the specific source document and version it was grounded in?

2. Walk me through what happens when an upstream system changes a data schema. Does anything break, and would you know before a user did?

3. How old is the oldest content in your production vector store, and how would you know if it were stale?

4. Who is the named owner of data quality for the sources your AI systems depend on?

5. If a regulator asked what data your AI systems have access to and how it is classified, what would you show them?

The quality of the answers to these five questions is a reliable predictor of the overall Layer 1 maturity score.

---

## Relationship to Other Layers

**Layer 1 enables Layer 2 (Architecture Patterns):** RAG and retrieval architecture is only as good as the data readiness beneath it - poor chunking, missing lineage, or stale embeddings undermine even a well-designed retrieval pattern.

**Layer 1 enables Layer 3 (Governance & Risk):** Data classification at this layer determines which AI systems are permitted to handle which data - a control that Layer 3 enforces and audits.

**Layer 1 is informed by Layer 4 (Operating Model):** Data ownership and contract enforcement require a clear operating model - without a named data owner, contracts have no one to enforce them.

**Layer 1 is informed by Layer 5 (Integration & Scale):** Production-scale AI systems place sustained load on data pipelines; readiness practices that work for a pilot often need re-architecting to hold up at scale.

---

*Part of the [CloudKraft AI Control Framework](./README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this layer.*
