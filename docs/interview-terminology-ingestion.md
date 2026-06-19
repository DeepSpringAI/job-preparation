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

## Dataflow

A **dataflow** describes how data moves through the system from source to destination.

In an ingestion system, dataflow usually means:

```text
Source System
  → Connector / Crawler
  → Raw Capture
  → Extraction / OCR
  → Cleaning / Normalization
  → Chunking
  → Metadata + ACL Capture
  → Embedding / Indexing
  → Storage
  → Retrieval / Serving
  → Monitoring / Feedback
```

### What the interviewer is really asking

When an interviewer asks about dataflows, they usually want to know whether you understand:

- Where data enters the system
- What transformations happen to it
- Where data is stored
- Which services depend on which outputs
- Where permissions and metadata are preserved
- Where failures, retries, and reprocessing happen
- How freshness and consistency are maintained

### Good interview answer pattern

Use this structure:

1. **Start with the source**
   - What system does the data come from?
   - Is it batch, streaming, webhook-based, or incremental sync?

2. **Describe the ingestion boundary**
   - How does the platform authenticate?
   - How are rate limits, pagination, and retries handled?

3. **Describe processing stages**
   - Extraction, OCR, parsing, cleaning, chunking, metadata enrichment, embeddings.

4. **Describe storage and indexes**
   - Raw store, metadata DB, search index, vector index, cache.

5. **Describe serving path**
   - Retrieval API, RAG service, access-control checks, observability.

6. **Mention failure handling**
   - Idempotency, retry, dead-letter queue, partial failure, replay, reprocessing.

### Interview-ready framing

> For dataflow, I would describe the path of the data end to end. For example, a document starts in SharePoint, is discovered by a connector, captured with metadata and permissions, extracted into text, cleaned, chunked, embedded, indexed, and then served through a retrieval API. I would also call out where access control is preserved, where failures are retried, and how we reprocess documents when the chunking strategy or embedding model changes.

---

## Blast Radius

**Blast radius** means the scope of impact when something fails or behaves incorrectly.

In system design, it answers:

- If this component fails, what breaks?
- How many users, tenants, documents, workflows, or downstream systems are affected?
- Can the failure be isolated?
- Can the system degrade gracefully?
- Can we stop, rollback, or contain the damage?

### Examples in ingestion

| Failure | Large blast radius | Smaller blast radius |
| --- | --- | --- |
| Bad parser release | All document types fail globally | Only one parser version / document type is affected |
| Embedding model bug | Entire vector index becomes unreliable | New embeddings are written to a versioned shadow index |
| Connector credential failure | All customers lose ingestion | One tenant/source is paused |
| Bad ACL handling | Users may see unauthorized documents | Retrieval is blocked if ACL validation fails |
| Queue backlog | Whole pipeline stops | Backlog isolated by tenant/source/priority queue |
| Reprocessing job bug | Existing production index is corrupted | Reprocessing writes to a new index and swaps after validation |

### What the interviewer is really asking

When an interviewer asks about blast radius, they are testing whether you design for operational safety, not just happy-path functionality.

They want to hear about:

- Isolation by tenant, source, region, queue, or index
- Feature flags and gradual rollout
- Versioned schemas, models, indexes, and processors
- Circuit breakers and kill switches
- Backpressure and rate limiting
- Safe retries and idempotency
- Shadow writes and validation before promotion
- Rollback strategy
- Monitoring and alerting

### Good interview answer pattern

Use this structure:

1. **Identify the failure mode**
   - Parser bug, bad embeddings, connector outage, permission issue, queue backlog, database outage.

2. **Define the impacted scope**
   - One document, one source, one tenant, one region, or the whole platform.

3. **Explain containment**
   - Tenant isolation, queue partitioning, feature flag, circuit breaker, versioned index.

4. **Explain recovery**
   - Retry, replay, rollback, reprocess, restore from raw data, switch index alias.

5. **Explain detection**
   - Metrics, alerts, dashboards, canaries, evaluation checks, audit logs.

### Interview-ready framing

> For blast radius, I would first identify what can fail and then explain how the design limits the impact. In ingestion, I would isolate work by tenant and source, use idempotent jobs, keep raw documents so we can reprocess, version parsers and embedding indexes, and roll out changes gradually with feature flags or shadow indexes. The goal is that a bad connector, parser, or model change affects only a limited slice of the system and can be rolled back or replayed safely.

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

11. **Blast-radius reduction**
    - How to isolate failures by tenant, source, pipeline stage, model version, index version, or deployment ring.

---

## Interview-ready framing

When asked about ingestion architecture, a strong answer is:

> I would start with an HLSD to define the end-to-end ingestion architecture: sources, connectors, sync model, processing pipeline, storage, indexing, access control, retrieval, monitoring, and operational flows. Then I would create ADRs for the major decisions, such as vector database selection, chunking strategy, embedding model, permission enforcement, and reprocessing strategy. The HLSD gives everyone the shared architecture picture, while ADRs preserve the reasoning behind important tradeoffs.

When asked specifically about dataflow:

> I would walk through the lifecycle of a document from the source system to the retrieval path, including connector discovery, extraction, metadata and ACL capture, chunking, embeddings, indexing, storage, and serving. I would also mention where failures are retried and how reprocessing works.

When asked specifically about blast radius:

> I would explain how the design limits the impact of failures. For example, isolate ingestion by tenant and source, use versioned processors and indexes, roll out changes gradually, keep raw data for replay, and make jobs idempotent so a failed or bad run can be safely retried or rolled back.

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
