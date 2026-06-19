# Interview Terminology: Ingestion and System Design

This document captures practical terminology that often appears in AI platform, enterprise ingestion, data engineering, and system-design interviews.

## ADR — Architecture Decision Record

An **ADR** is a short document that records an important architecture or technical decision.

It answers:

- What problem were we solving?
- What decision did we make?
- What alternatives did we consider?
- What tradeoffs or consequences did we accept?

### Why interviewers care

ADRs show that you can reason beyond code implementation. They demonstrate structured thinking, tradeoff analysis, and long-term ownership of architecture decisions.

### Example in an ingestion system

**ADR: Vector Database Selection**

- **Context:** The ingestion platform needs semantic search over a large document corpus.
- **Decision:** Use OpenSearch vector search for the first production version.
- **Alternatives considered:** Pinecone, Weaviate, Milvus, PostgreSQL with pgvector.
- **Consequences:**
  - Lower platform cost if OpenSearch is already available.
  - More operational responsibility than a fully managed vector database.
  - Better fit if keyword search and semantic search need to live together.

### How to explain it in an interview

> An ADR is the written memory of an architecture decision. For an ingestion platform, I would use ADRs for decisions such as crawler strategy, chunking approach, embedding model, vector database, metadata schema, access-control model, and reprocessing strategy. The value is that future engineers can understand why the decision was made, not just what the system currently does.

---

## HLSD — High-Level Solution Design

An **HLSD** is a high-level design document that explains the overall solution architecture before detailed implementation.

It usually covers:

- Business goals
- Functional requirements
- Non-functional requirements
- Major components
- Integrations
- Data flows
- Security and access control
- Deployment architecture
- Operational concerns
- Major assumptions, risks, and open questions

### Why interviewers care

HLSDs show that you can design the full system, communicate it clearly, and connect technical architecture to business requirements.

### Example in an ingestion system

```text
Enterprise Sources
  ├─ SharePoint
  ├─ Google Drive
  ├─ Confluence
  ├─ Slack
  └─ Email

        ↓

Ingestion Layer
  ├─ Connectors / Crawlers
  ├─ Incremental Sync
  ├─ Permissions Capture
  └─ Change Detection

        ↓

Processing Layer
  ├─ Text Extraction
  ├─ OCR
  ├─ Cleaning / Normalization
  ├─ Chunking
  ├─ Metadata Extraction
  └─ Embedding Generation

        ↓

Storage Layer
  ├─ Raw Object Store
  ├─ Metadata Store
  ├─ Search Index
  └─ Vector Index

        ↓

Serving Layer
  ├─ Retrieval API
  ├─ RAG Service
  ├─ Access-Control Enforcement
  └─ Observability / Evaluation
```

### How to explain it in an interview

> An HLSD describes the overall shape of the solution. For ingestion, I would include source connectors, sync strategy, processing pipeline, metadata model, indexing, storage, retrieval, security, monitoring, and reprocessing. It is not the same as detailed code design; it is the architecture-level blueprint that aligns engineering, product, security, and operations.

---

## Relationship Between HLSD, ADR, and LLD

| Artifact | Purpose | Example in ingestion |
| --- | --- | --- |
| HLSD | Describes the overall solution architecture | The system has connectors, a processing pipeline, metadata store, vector index, and retrieval API. |
| ADR | Explains a specific architecture decision | We selected OpenSearch instead of Pinecone because we need hybrid search and already operate OpenSearch. |
| LLD | Describes implementation-level details | Index schema, API contracts, retry logic, queue topics, database tables, and worker behavior. |

A good sequence is:

```text
Requirements → HLSD → ADRs → LLDs → Implementation → Runbooks → Operational Reviews
```

---

## Common ingestion decisions worth capturing as ADRs

1. **Connector strategy**
   - Build custom connectors, use third-party connectors, or use MCP/connectors from a platform.

2. **Full sync vs incremental sync**
   - Crawl everything repeatedly, or track deltas, timestamps, webhooks, and change tokens.

3. **Raw data retention**
   - Store original files, extracted text only, or both.

4. **Chunking strategy**
   - Fixed-size chunks, semantic chunks, section-aware chunks, page-aware chunks, or hierarchical chunks.

5. **Embedding model selection**
   - Cloud embedding API, open-source model, multimodal embedding, domain-specific embedding, or hybrid approach.

6. **Indexing strategy**
   - Vector-only retrieval, keyword-only retrieval, hybrid retrieval, reranking, or graph-enhanced retrieval.

7. **Permission model**
   - Enforce access control at ingestion time, retrieval time, or both.

8. **Reprocessing strategy**
   - How to handle model upgrades, schema changes, chunking changes, and embedding regeneration.

9. **Failure handling**
   - Retry, dead-letter queue, partial ingestion, poison-document handling, and manual repair workflows.

10. **Observability and evaluation**
    - Track freshness, coverage, latency, extraction failures, retrieval quality, and answer quality.

---

## Interview-ready framing

When asked about ingestion architecture, a strong answer is:

> I would start with an HLSD to define the end-to-end ingestion architecture: sources, connectors, sync model, processing pipeline, storage, indexing, access control, retrieval, monitoring, and operational flows. Then I would create ADRs for the major decisions, such as vector database selection, chunking strategy, embedding model, permission enforcement, and reprocessing strategy. The HLSD gives everyone the shared architecture picture, while ADRs preserve the reasoning behind important tradeoffs.

---

## Terms to add later

- LLD — Low-Level Design
- NFR — Non-Functional Requirement
- SLO / SLA / SLI
- RTO / RPO
- Data lineage
- Backfill
- Incremental sync
- Dead-letter queue
- Idempotency
- Reprocessing
- Schema evolution
- Hybrid retrieval
- Reranking
- ACL-aware retrieval
- Observability
- Runbook
