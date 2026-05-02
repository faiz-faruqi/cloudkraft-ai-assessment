# Layer 2 - Data & Retrieval

**The question this layer answers:** How does the organisation connect enterprise knowledge to AI systems safely, accurately, and with enough freshness to be useful?

**Why this layer matters:** A language model without enterprise context is a general-purpose tool. A language model grounded in your policies, contracts, product documentation, and operational knowledge is a business capability. The Data & Retrieval layer is what makes the difference between an AI that hallucinates plausible-sounding answers and one that cites your actual documents.

**Primary stakeholders:** Chief Data Officer, Data Architecture, Information Security, Legal

---

## What Good Looks Like at Each Maturity Level

**Level 1 - Initial**
AI systems operate entirely on base model training data. When users ask questions about company policies, products, or operations, the model either hallucinates or declines. There is no systematic way to connect enterprise knowledge to AI systems.

**Level 2 - Developing**
Ad hoc document uploads to individual AI tools. Some teams have basic chatbots connected to SharePoint or Confluence. No data classification, no access control at the retrieval layer, no evaluation of answer quality.

**Level 3 - Defined**
A governed RAG system connects AI to a curated knowledge base. Documents are classified and access-controlled. A test set exists for evaluating retrieval quality. Freshness is managed through scheduled ingestion. At least one use case is in production with monitored quality.

**Level 4 - Managed**
Hybrid retrieval combining semantic search and keyword search with re-ranking. Multiple knowledge domains with domain-specific indices. Automated freshness management with staleness detection. Continuous evaluation with RAGAS or equivalent metrics tracked over time. Data lineage from source document to AI response is traceable.

**Level 5 - Optimising**
Multi-source retrieval federation across structured and unstructured data. Real-time or near-real-time knowledge synchronisation. Confidence scoring on every response with automated escalation for low-confidence answers. Business-outcome metrics (task completion, escalation rate, user satisfaction) tracked alongside technical metrics.

---

## Named Patterns

### Pattern 1 - Hybrid RAG

**What it is:** A retrieval pipeline that combines dense vector search (semantic similarity) with sparse keyword search (BM25 or equivalent), and uses Reciprocal Rank Fusion (RRF) or a learned re-ranker to combine results before passing context to the LLM.

**When to use it:** Always, for any production enterprise RAG system. Naive vector-only retrieval fails on exact-match queries (product codes, regulation numbers, clause references) where keyword search dominates. Keyword-only search fails on semantic queries where meaning matters more than exact terms. Hybrid retrieval handles both.

**Why vector-only RAG fails in regulated industries:**
A compliance officer asking "what does our policy say about Article 9 data under GDPR?" needs exact-match retrieval on "Article 9" combined with semantic understanding of "policy" in their specific context. Pure semantic search often returns documents about GDPR generally but misses the specific article. Hybrid retrieval catches both signals.

**Implementation approach:**
- Embed documents using a production-grade embedding model (see tool matrix)
- Store embeddings in a vector database with metadata for filtering
- Run parallel BM25 search on the same document corpus
- Combine results using RRF with a tuned k parameter (typically 60)
- Apply a cross-encoder re-ranker on the top 20 candidates if latency budget allows
- Pass top 5-10 re-ranked chunks as context to the LLM

**Trade-offs:**
- Higher complexity than vector-only retrieval - two retrieval paths, a fusion step, optional re-ranking
- Cross-encoder re-ranking adds 100-300ms latency - evaluate whether your use case requires it
- BM25 requires a separate index (Elasticsearch, OpenSearch, or in-memory) alongside your vector store

---

### Pattern 2 - Structured Data Grounding

**What it is:** Connecting AI systems to structured data sources (databases, data warehouses, APIs) rather than, or in addition to, unstructured documents.

**When to use it:** When users need precise numerical answers, current operational data, or information that changes frequently. Document RAG cannot answer "what was our revenue in Q3?" or "how many open tickets does customer X have?" - structured grounding can.

**Implementation approaches:**
- Text-to-SQL: LLM generates SQL queries from natural language, executes against a read-only database replica
- API tool calling: LLM is given tool definitions for internal APIs and calls them to retrieve current data
- Semantic layer: Expose a governed semantic layer (dbt metrics, Cube, Looker) as the AI data source

**Trade-offs:**
- Text-to-SQL is powerful but requires careful guardrailing - an LLM with unrestricted SQL access is a security risk
- API tool calling requires well-documented, stable internal APIs - most enterprises do not have these
- All structured grounding requires read-only access with the same data classification controls as direct database access

---

### Pattern 3 - Multi-Index Federation

**What it is:** Separate vector indices for different knowledge domains, with a router that directs each query to the appropriate index (or queries multiple indices and merges results).

**When to use it:** Organisations with distinct knowledge domains that have different access control requirements, different freshness needs, or different retrieval characteristics. A bank might have separate indices for regulatory documents, product specifications, internal policies, and client-facing FAQs - each with different access rules.

**Trade-offs:**
- Query routing adds complexity - the router must correctly classify each query
- Merging results from multiple indices requires careful score normalisation
- Each index has its own ingestion pipeline and maintenance burden

---

## Tool Selection Matrix - Data & Retrieval Layer

### Vector Databases

| Tool | Best For | Self-Hosted | Canadian Data Residency | Limitations |
|---|---|---|---|---|
| Qdrant | Performance, filtering, self-hosted | Yes | Yes | Less mature managed cloud |
| Weaviate | Hybrid search built-in, GraphQL API | Yes | Limited | Complex configuration |
| Pinecone | Managed simplicity, fast setup | No | Limited | Vendor lock-in, cost at scale |
| pgvector | Existing PostgreSQL, simple use cases | Yes | Yes | Performance limits at scale |
| Azure AI Search | Azure-committed, hybrid search built-in | No | Yes (canadacentral) | Azure-only |
| OpenSearch | Existing OpenSearch/Elasticsearch, hybrid search | Yes | Yes | Operational complexity |

**CloudKraft recommendation:** Qdrant for self-hosted regulated environments (strong filtering, excellent performance, active development). Azure AI Search for Azure-committed organisations (hybrid search native, no separate BM25 index needed). Avoid Pinecone for data-sensitive industries due to limited data residency options.

### Embedding Models

| Model | Dimensions | Quality | Cost | Self-Hosted |
|---|---|---|---|---|
| text-embedding-3-large (OpenAI) | 3072 | Excellent | $0.13/1M tokens | No |
| text-embedding-3-small (OpenAI) | 1536 | Very Good | $0.02/1M tokens | No |
| Cohere Embed v3 | 1024 | Excellent | $0.10/1M tokens | No |
| all-MiniLM-L6-v2 | 384 | Good | Free | Yes |
| BGE-M3 | 1024 | Excellent | Free | Yes |
| Titan Embeddings (AWS) | 1536 | Good | $0.02/1M tokens | No |

**CloudKraft recommendation:** text-embedding-3-small for cost-sensitive production workloads with OpenAI already in the stack. BGE-M3 for self-hosted regulated environments requiring data residency. Never use a self-hosted embedding model in production without an evaluation baseline - quality varies significantly by domain.

### Orchestration Frameworks for RAG

| Tool | Strengths | Weaknesses |
|---|---|---|
| LangChain | Broad ecosystem, many integrations | Abstraction complexity, frequent breaking changes |
| LlamaIndex | RAG-specialised, strong indexing | Narrower than LangChain for non-RAG use cases |
| Haystack | Production-grade, strong evaluation | Steeper learning curve |
| Custom FastAPI | Full control, no framework overhead | Build and maintenance cost |

**CloudKraft recommendation:** LlamaIndex for RAG-first systems. LangChain when the system needs to combine RAG with agents and tool calling. Custom implementation for systems where framework abstraction adds more friction than value.

---

## Anti-Patterns

### Naive Top-K Retrieval Without Re-Ranking

**What happens:** The RAG system retrieves the top 5 most semantically similar chunks and passes them directly to the LLM. No re-ranking. No hybrid search. No confidence scoring.

**Why it is dangerous:** Semantic similarity is not the same as relevance. A chunk that is semantically close to the query may be from an outdated document, may be from a section the user does not have access to, or may simply be adjacent to the relevant section rather than containing it. Without re-ranking, these near-misses become hallucination fuel.

**Symptoms:** Users report that the AI gives answers that are "close but not quite right." The system performs well on broad questions but fails on specific ones. Evaluation shows high recall but low precision.

**The fix:** Hybrid retrieval plus a cross-encoder re-ranker. Evaluate with RAGAS or equivalent. Treat retrieval quality as a first-class engineering concern, not a default setting.

---

### Embedding Model Lock-In

**What happens:** The organisation builds a production RAG system on a specific embedding model. Later, a better model is released, or the provider changes pricing, or the organisation needs to move to a self-hosted model for compliance reasons. Switching models requires re-embedding the entire knowledge base and re-evaluating retrieval quality from scratch.

**Why it happens:** Embedding model selection is treated as a one-time implementation decision rather than an architectural concern.

**The fix:** Abstract the embedding model behind an interface. Document the model selection as an ADR with explicit criteria for when the decision should be revisited. Maintain a golden evaluation dataset that allows you to benchmark a new model before migrating.

---

### Ignoring Structured Data Sources

**What happens:** The RAG system is built on unstructured documents. Users quickly discover that it cannot answer questions about current operational data, customer records, or financial figures. They stop trusting it for anything time-sensitive.

**Why it happens:** Unstructured RAG is easier to implement. Structured data grounding requires database access, schema understanding, and careful security design.

**The fix:** Explicitly scope what the system will and will not answer. If structured data is in scope, implement text-to-SQL or API tool calling with appropriate guardrails. If it is not in scope, be explicit with users about the boundary - "this system answers questions about policy documents, not operational data."

---

## Key Questions for a Client Engagement

1. Show me the most recent document in your AI knowledge base. When was it ingested? When was the source document last updated? How would the system know if it became stale?

2. If I asked your AI system a question about a policy that only certain employees are allowed to access, what would happen? Would it answer? Would it tell me I do not have access? Would it not know the document exists?

3. How do you know your RAG system is giving accurate answers today compared to three months ago? What would tell you if quality had degraded?

4. Walk me through what happens when a new policy document is published. How long until the AI system knows about it?

5. What is the most important question users ask this system that it frequently gets wrong? What is your hypothesis for why?

---

## Relationship to Other Layers

**Layer 2 depends on Layer 1:** Data classification at the retrieval layer must be enforced at the access layer. If Layer 1 does not control which models can access which data, Layer 2 classification is advisory rather than enforced.

**Layer 2 feeds Layer 4:** Retrieval quality metrics (context precision, answer faithfulness, answer relevance) are the primary inputs to the Layer 4 evaluation pipeline. Without retrieval quality data, evaluation is incomplete.

**Layer 2 connects to Layer 5:** The ingestion pipeline that keeps the knowledge base fresh is an integration concern - it connects source systems (SharePoint, Confluence, databases) to the vector store through the enterprise integration layer.

---

*Part of the [CloudKraft AI Control Tower framework](./README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this layer.*
