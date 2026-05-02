# Pattern: Hybrid RAG

**Layer:** 2 — Data & Retrieval
**Problem:** Vector-only RAG fails on exact-match queries (product codes, regulation numbers, clause references) while keyword-only search misses semantic meaning. Neither alone is sufficient for production enterprise knowledge retrieval.
**Solution:** A retrieval pipeline that runs dense vector search and sparse keyword search in parallel, then fuses results using Reciprocal Rank Fusion (RRF) before passing context to the LLM.

---

## Context

Pure vector search is the default choice because it is the simplest to implement. It fails in enterprise environments for a predictable set of reasons:

- A compliance officer asks about "Article 9 data" — the model retrieves documents about GDPR broadly, misses the specific article
- A procurement analyst searches for contract number "MSA-2024-0041" — semantic search returns unrelated contracts with similar language
- An engineer asks about error code "E-4092" — embedding similarity is near-useless for opaque identifiers
- A user asks a question whose answer is in a section with domain-specific terminology that does not appear in the query

Keyword search catches all of these. But keyword search alone misses synonyms, reformulations, and questions where the user's phrasing differs from the document language.

Hybrid RAG is not an optimisation — it is the minimum viable retrieval architecture for a production enterprise RAG system.

---

## How It Works

```
Query
  ├─ Dense Retrieval (embeddings + vector DB)  → top-20 candidates
  └─ Sparse Retrieval (BM25 / keyword index)   → top-20 candidates
                      ↓
              RRF Fusion (k=60)
                      ↓
              Cross-Encoder Re-Ranker (optional, top-10 candidates)
                      ↓
              Top-5 chunks → LLM context window
```

**Step 1 — Dense retrieval**
Embed the query using the same model used to embed the corpus. Query the vector database for the top-20 semantically similar chunks. Apply metadata filters (document type, access tier, recency) before or after the vector search depending on your vector database's filtering architecture.

**Step 2 — Sparse retrieval**
Run a BM25 query against the same document corpus. This requires a separate keyword index (Elasticsearch, OpenSearch, or an in-process BM25 library like `rank_bm25` for smaller corpora). Return the top-20 results.

**Step 3 — RRF fusion**
Combine the two ranked lists using Reciprocal Rank Fusion. For a document d appearing at rank r in a list, its score contribution is 1/(k+r) where k=60 is the standard default. Sum contributions across both lists. Re-rank by combined score.

```python
def reciprocal_rank_fusion(results_dense, results_sparse, k=60):
    scores = {}
    for rank, doc in enumerate(results_dense):
        scores[doc.id] = scores.get(doc.id, 0) + 1 / (k + rank + 1)
    for rank, doc in enumerate(results_sparse):
        scores[doc.id] = scores.get(doc.id, 0) + 1 / (k + rank + 1)
    return sorted(scores.items(), key=lambda x: x[1], reverse=True)
```

**Step 4 — Cross-encoder re-ranking (recommended for high-stakes use cases)**
Pass the top 10–20 fused candidates to a cross-encoder re-ranker. Unlike bi-encoder embeddings that score query and document independently, cross-encoders take the query and document together, producing more accurate relevance scores at the cost of latency. Use `cross-encoder/ms-marco-MiniLM-L-6-v2` (self-hosted, fast) or Cohere Rerank (managed, higher quality).

**Step 5 — Context assembly**
Pass the top 5 re-ranked chunks to the LLM with explicit source citations. Instruct the model to cite which chunk each claim comes from. This enables answer faithfulness evaluation downstream.

---

## Tool Selection

### Vector Databases

| Tool | Hybrid Search Support | Self-Hosted | Canadian Residency | Best For |
|---|---|---|---|---|
| Qdrant | Yes (native sparse + dense) | Yes | Yes | Self-hosted regulated environments |
| Azure AI Search | Yes (native) | No | Yes (canadacentral) | Azure-committed organisations |
| Weaviate | Yes (BM25 built-in) | Yes | Limited | GraphQL API users |
| OpenSearch | Yes (via k-NN plugin) | Yes | Yes | Existing OpenSearch users |
| Pinecone | Sparse + dense separate | No | Limited | Managed simplicity, non-regulated |
| pgvector | No native hybrid | Yes | Yes | Simple use cases on existing PostgreSQL |

**CloudKraft recommendation:** Qdrant for self-hosted hybrid RAG — it supports sparse and dense vectors natively in a single query, eliminating the need for a separate BM25 index. Azure AI Search for Azure-committed organisations — hybrid search is native and requires no additional infrastructure. For any other vector database, pair it with OpenSearch or Elasticsearch for BM25.

### Embedding Models

| Model | Quality | Self-Hosted | Cost | Notes |
|---|---|---|---|---|
| text-embedding-3-large | Excellent | No | $0.13/1M tokens | Best for OpenAI-committed stacks |
| text-embedding-3-small | Very Good | No | $0.02/1M tokens | Best cost/quality for OpenAI stacks |
| BGE-M3 | Excellent | Yes | Free | Best self-hosted option, multilingual |
| Cohere Embed v3 | Excellent | No | $0.10/1M tokens | Best multi-language support |
| all-MiniLM-L6-v2 | Good | Yes | Free | Development/prototyping only |

### Re-Rankers

| Model | Quality | Latency | Self-Hosted | Notes |
|---|---|---|---|---|
| Cohere Rerank 3.5 | Best | +100–200ms | No | Managed, highest quality |
| cross-encoder/ms-marco-MiniLM-L-6-v2 | Good | +50–100ms | Yes | Good self-hosted option |
| BGE-Reranker-Large | Very Good | +80–150ms | Yes | Best self-hosted quality |
| Jina Reranker v2 | Good | +60–120ms | Yes | Active development |

---

## Trade-offs

**What you gain:**
- Significantly better retrieval on exact-match queries (codes, identifiers, named entities)
- Better recall on semantic queries than keyword search alone
- More robust to query phrasing variation than either approach individually
- Industry-validated: every serious enterprise RAG evaluation shows hybrid outperforms pure vector search on recall@5

**What it costs:**
- Two retrieval paths to maintain instead of one
- Requires a keyword index alongside the vector database (unless your vector DB supports both natively)
- Cross-encoder re-ranking adds 50–300ms latency — evaluate whether your use case requires it
- Slightly higher infrastructure cost for the keyword index

---

## Ingestion Pipeline

Retrieval quality depends entirely on ingestion quality. The retrieval pipeline is only as good as what was put in.

**Chunking strategy matters more than most teams realise:**
- Fixed-size chunking (512 tokens, 128 token overlap) is the default and is frequently wrong
- Semantic chunking (split at sentence boundaries, paragraph boundaries, or semantic shifts) produces more coherent chunks
- Hierarchical chunking (embed both a paragraph summary and the full paragraph) improves cross-encoder re-ranking accuracy
- Document-specific chunking (different strategy for contracts vs. emails vs. technical specs) outperforms a universal strategy

**Metadata matters:**
Every chunk should carry: source document ID, section title, document type, creation date, last-modified date, access classification, and author/owner. These metadata fields power the filters that prevent returning expired or unauthorised content.

**Freshness management:**
Stale retrieval is a silent failure mode. A document that was accurate when ingested but has since been superseded will continue to be retrieved until the old version is deleted and the new version is re-ingested. Schedule incremental ingestion daily for high-velocity document sources. Detect and flag documents older than a threshold. Surface staleness warnings to users when retrieved content is more than N days old.

---

## Evaluation

Hybrid RAG must be evaluated with RAG-specific metrics. General LLM benchmarks do not capture retrieval quality.

**RAGAS metrics for RAG evaluation:**
- **Context Precision** — what fraction of the retrieved context was actually relevant to the answer?
- **Context Recall** — what fraction of the information needed to answer the question was present in the retrieved context?
- **Answer Faithfulness** — how much of the generated answer is grounded in the retrieved context (not hallucinated)?
- **Answer Relevance** — how relevant is the generated answer to the original question?

Establish a baseline on all four metrics before the first production deployment. Alert on regressions of more than 5 percentage points on any metric.

---

## Relationship to Other Patterns

**Depends on:**
- [Centralised AI Gateway](./centralised-ai-gateway.md) — data classification at the retrieval layer must be enforced at the access layer

**Feeds into:**
- [Continuous Evaluation](./continuous-evaluation.md) — context precision and context recall from the retrieval pipeline are primary inputs to the evaluation framework
- [Human-in-the-Loop](./human-in-the-loop.md) — low-confidence retrieval is a valid trigger for human escalation

---

*Part of the [CloudKraft AI Control Tower framework](../framework/README.md). See [Layer 2 — Data & Retrieval](../framework/layer-2-data-retrieval.md) for the full layer context.*
