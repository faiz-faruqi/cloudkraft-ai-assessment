# Layer 2 - Architecture Patterns

**The question this layer answers:** How are RAG systems, multi-agent workflows, and AI integrations designed - as reusable, governed patterns, or as one-off builds that nobody can maintain?

**Why this layer matters:** Architecture decisions made in the first AI pilot become the default pattern for the next ten, whether or not they deserve to be. Without named, governed patterns, every team reinvents retrieval and orchestration independently - at different quality levels, with different failure modes, and with no shared vocabulary for architecture review.

**Primary stakeholders:** VP Engineering, Platform Engineering, Solutions Architecture

---

## What Good Looks Like at Each Maturity Level

**Level 1 - Initial**
AI is used in isolated, single-step interactions with no retrieval grounding beyond base model knowledge. Where RAG exists, it is a single-source implementation built ad hoc for one use case. There is no shared vocabulary for architecture across teams.

**Level 2 - Developing**
Basic chaining exists in individual applications but with no shared framework. A single RAG pattern may be reused informally, but retrieval quality - chunking strategy, reranking - is unoptimised and undocumented. System topology exists only in individual engineers' heads.

**Level 3 - Defined**
A reusable RAG pattern and a shared orchestration framework exist, though agent roles and handoffs are loosely defined. A recommended integration pattern exists but adoption across teams is inconsistent. A current topology diagram exists but is not systematically maintained.

**Level 4 - Managed**
A governed hybrid RAG pattern (dense + sparse retrieval with reranking) is standard across use cases. Multi-agent workflows use a governed framework with clearly defined roles, handoffs, and state management. Architecture Decision Records are used for significant design choices, and topology is kept current as part of the review process.

**Level 5 - Optimising**
Hybrid retrieval with continuous relevance evaluation and multi-source fusion is standard practice. Multi-agent orchestration supports dynamic role assignment and supervision, with patterns reused across teams by default. Topology is modelled as living architecture documentation, auto-updated and used to assess the blast radius of proposed changes.

---

## Named Patterns

### Pattern 1 - Hybrid RAG with Reranking

**What it is:** A retrieval pipeline that combines dense (embedding-based) and sparse (keyword-based) retrieval, then reranks the merged candidate set before passing it to the model - rather than relying on a single retrieval method.

**When to use it:** The default pattern for any production RAG system. Dense retrieval alone misses exact-match terms (product codes, names, acronyms); sparse retrieval alone misses semantic similarity. The combination consistently outperforms either alone.

**How it works:**
- The query is run against both a vector index and a keyword index (e.g. BM25) in parallel
- Candidate results from both are merged and deduplicated
- A reranking model scores the merged set for relevance to the specific query
- The top-N reranked results are passed to the generation model as context

**Trade-offs:**
- Adds latency and infrastructure complexity versus single-method retrieval - justified once retrieval accuracy directly affects business or compliance outcomes
- Reranking models add cost per query - mitigated by only reranking the merged candidate set, not the full corpus

**Tools that implement this pattern:**

| Tool | Best For | Limitations |
|---|---|---|
| LlamaIndex | Rapid RAG pipeline assembly with built-in hybrid retrieval | Abstraction can obscure what is actually happening at query time |
| LangChain / LangGraph | Composable retrieval + orchestration in one framework | Steeper learning curve for teams new to the ecosystem |
| Elasticsearch / OpenSearch (hybrid mode) | Organisations already running Elastic for search | Requires tuning to get hybrid scoring right |
| Cohere Rerank / Voyage Rerank | Drop-in reranking for an existing retrieval pipeline | Adds an external API dependency and per-query cost |
| Custom retrieval pipeline | Full control, specific latency or compliance requirements | Build and maintenance cost |

**CloudKraft recommendation:** For most enterprises standing up their first governed RAG system, LlamaIndex or LangGraph plus a managed reranking API is the fastest path to production-grade hybrid retrieval. Custom pipelines are justified once query volume or latency requirements exceed what packaged tools handle well.

---

### Pattern 2 - Multi-Agent Orchestration with Defined Roles

**What it is:** Multi-step AI workflows structured as named agents with explicit responsibilities, handoff contracts, and shared state - rather than a single long prompt chain doing everything.

**When to use it:** Any workflow with more than two or three distinct reasoning steps, or where different steps benefit from different tools, context, or model choices.

**Trade-offs:**
- More moving parts to monitor and debug than a single-agent chain - worthwhile once workflow complexity would otherwise produce an unmaintainable prompt
- Requires explicit state management design up front - skipping this is the most common source of orchestration failures

**CloudKraft assessment:** Teams frequently reach for multi-agent frameworks before a single well-structured agent with good tool access would suffice. Multi-agent orchestration earns its complexity when steps genuinely require different context, tools, or specialisation - not by default.

---

### Pattern 3 - Event-Driven Integration Topology

**What it is:** AI components as event producers and consumers within existing enterprise messaging infrastructure, rather than direct point-to-point API calls between every system and every AI component.

**When to use it:** Organisations with more than a handful of AI touchpoints across the enterprise, where point-to-point integration would otherwise produce an unmanageable mesh of direct dependencies.

**Trade-offs:**
- Requires existing or new event infrastructure - a bigger up-front investment than direct API calls
- Decouples systems effectively, at the cost of making end-to-end tracing harder without good observability (see Layer 5)

---

## Tool Selection Matrix - Architecture Patterns Layer

| Capability | LangGraph | LlamaIndex | Semantic Kernel | CrewAI | Custom |
|---|---|---|---|---|---|
| Hybrid RAG support | Good (composable) | Excellent | Good | Limited | Full control |
| Multi-agent orchestration | Excellent | Limited | Good | Good | Full control |
| State management | Explicit, strong | Basic | Good | Basic | Custom |
| Enterprise .NET/Java fit | Limited | Limited | Excellent | Limited | Full control |
| Maturity / ecosystem | Growing fast | Mature | Mature (Microsoft) | Newer | N/A |
| Setup complexity | Medium | Low-Medium | Medium | Low | High |

---

## Anti-Patterns

### The Monolithic Prompt Chain

**What happens:** A single, sprawling prompt (or prompt chain with no defined boundaries) tries to handle retrieval, reasoning, formatting, and tool use all at once. It works in the demo and becomes unmaintainable within a few months as more logic is bolted on.

**Why it happens:** It is the fastest way to get a first version working. Structuring a proper multi-step architecture up front feels like premature engineering - until the prompt hits its complexity ceiling.

**Why it is dangerous:** Debugging a failure means guessing which part of a single opaque prompt is responsible. Every new requirement risks breaking unrelated behaviour elsewhere in the same prompt.

**The fix:** Decompose into named steps or agents with defined responsibilities and explicit handoffs, even if the first version is a single agent with clearly separated internal stages.

---

### Retrieval Without Reranking

**What happens:** A RAG system retrieves the top-K results by vector similarity alone and passes them directly to the model. Superficially similar but irrelevant content crowds out the genuinely relevant passage, and answer quality suffers in ways that are hard to diagnose.

**Why it happens:** Single-method vector retrieval is the fastest thing to stand up and looks fine in early testing with easy queries.

**Why it is dangerous:** Retrieval quality problems masquerade as model quality problems. Teams spend effort tuning prompts or switching models when the actual defect is upstream in retrieval.

**The fix:** Hybrid retrieval with reranking (Pattern 1), and retrieval-quality evaluation as a distinct, measured step separate from end-to-end answer evaluation.

---

### Tight Coupling Between AI Components

**What happens:** Every AI component calls every other AI component and business system directly. Changing one component requires understanding and testing every system that calls it.

**Why it happens:** Direct calls are the simplest thing to build for the first two or three integrations.

**Why it is dangerous:** The integration mesh becomes exponentially harder to reason about and change as AI touchpoints multiply across the enterprise.

**The fix:** Event-driven integration topology (Pattern 3) with clear producer/consumer contracts, adopted before the mesh becomes unmanageable rather than after.

---

## Key Questions for a Client Engagement

1. Walk me through what happens end-to-end when a user asks your RAG system a question - what actually determines which content it sees?

2. If two teams both need multi-step AI workflows, would they build them the same way, or independently from scratch?

3. Can you produce a current diagram of how your AI systems, agents, and data sources connect to each other?

4. How do you decide whether a workflow needs one agent or several?

5. What is your process for reviewing and recording a significant AI architecture decision?

The quality of the answers to these five questions is a reliable predictor of the overall Layer 2 maturity score.

---

## Relationship to Other Layers

**Layer 2 depends on Layer 1 (Data Readiness):** Even a well-designed hybrid RAG pattern produces poor answers if the underlying data lacks quality, lineage, or freshness controls.

**Layer 2 is constrained by Layer 3 (Governance & Risk):** Access control and audit logging requirements shape which integration and orchestration patterns are permissible for a given data sensitivity level.

**Layer 2 is enabled by Layer 4 (Operating Model):** Shared, governed patterns only emerge when there is an operating model - a central team or CoE - with the mandate to define and enforce them across business units.

**Layer 2 feeds Layer 5 (Integration & Scale):** The architecture patterns chosen here determine how much re-architecture is required when a pilot needs to scale to production load.

---

*Part of the [CloudKraft AI Control Framework](./README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this layer.*
