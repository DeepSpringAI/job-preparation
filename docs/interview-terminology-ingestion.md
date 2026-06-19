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

## NFR — Non-Functional Requirement

An **NFR** describes how the system should behave, not just what features it should provide.

Functional requirements describe capabilities, for example:

- Ingest documents from SharePoint.
- Extract text from PDFs.
- Generate embeddings.
- Serve retrieval results to a RAG application.

Non-functional requirements describe quality attributes, for example:

- The ingestion pipeline should process 1 million documents per day.
- Search results should respect source-system permissions.
- Retrieval latency should stay under a defined threshold.
- The platform should recover from worker failure without data loss.
- All document access should be auditable.

### Common NFR categories

| NFR category | What it means | Ingestion example |
| --- | --- | --- |
| Scalability | Can the system grow with load? | Handle more tenants, sources, documents, and embeddings. |
| Availability | Is the system usable when parts fail? | Retrieval remains available even if ingestion workers are delayed. |
| Reliability | Does it behave correctly over time? | Jobs are idempotent and can be retried safely. |
| Performance | Is it fast enough? | Extraction, indexing, and retrieval meet latency targets. |
| Security | Is access protected? | ACL-aware retrieval prevents unauthorized document exposure. |
| Privacy | Is sensitive data handled correctly? | PII is masked, encrypted, or excluded where required. |
| Compliance | Does it meet regulatory requirements? | Audit logs, retention policies, and data residency controls. |
| Observability | Can we detect and debug problems? | Metrics for freshness, failures, backlog, latency, and quality. |
| Maintainability | Can engineers safely evolve it? | Versioned processors, schemas, indexes, and ADRs. |
| Cost efficiency | Is cost controlled? | Batch embeddings, cache results, and avoid unnecessary reprocessing. |

### What the interviewer is really asking

When an interviewer asks about NFRs, they want to know whether you think like a production architect.

They are testing whether you consider:

- Scale
- Latency
- Security
- Reliability
- Availability
- Cost
- Compliance
- Operability
- Maintainability
- Failure recovery

### Good interview answer pattern

Use this structure:

1. **Separate functional from non-functional requirements**
   - Functional: what the system does.
   - Non-functional: how well, how safely, and under what constraints it does it.

2. **Tie NFRs to business risk**
   - Security prevents data leaks.
   - Freshness affects trust.
   - Latency affects user experience.
   - Availability affects adoption.

3. **Make NFRs measurable**
   - Avoid vague phrases like "fast" or "scalable."
   - Use measurable targets: latency, throughput, error rate, freshness, recovery time.

4. **Show architectural consequences**
   - NFRs drive queue design, partitioning, caching, indexing, monitoring, deployment strategy, and access control.

### Interview-ready framing

> NFRs are the production qualities of the system. For ingestion, I would define NFRs around scalability, freshness, reliability, security, permission correctness, retrieval latency, observability, and cost. For example, it is not enough to say the system ingests documents; we need to define how many documents per day, how fresh the index should be, how ACLs are enforced, what happens when workers fail, and how we detect bad extraction or indexing quality.

---

## SDLC — Software Development Life Cycle

**SDLC** describes the structured process used to plan, build, test, release, operate, and improve software.

A typical SDLC includes:

```text
Discovery / Requirements
  → Design
  → Implementation
  → Testing / Validation
  → Release / Deployment
  → Operations / Monitoring
  → Feedback / Iteration
```

In enterprise environments, SDLC often also includes governance, security reviews, architecture reviews, documentation, change management, release approvals, and post-release monitoring.

### SDLC in an ingestion or AI platform project

| SDLC phase | What happens | Ingestion example |
| --- | --- | --- |
| Discovery / Requirements | Understand business goals, users, constraints, and risks. | Which sources must be ingested? What freshness, compliance, and ACL requirements exist? |
| Design | Create HLSD, ADRs, dataflows, APIs, schemas, and deployment model. | Decide connector architecture, chunking strategy, vector DB, and permission model. |
| Implementation | Build the system components. | Build crawlers, parsers, workers, queues, metadata store, indexers, and retrieval APIs. |
| Testing / Validation | Verify correctness, quality, security, and performance. | Test extraction quality, ACL correctness, indexing completeness, latency, and failure handling. |
| Release / Deployment | Roll out safely. | Use feature flags, canaries, staged rollout, shadow indexes, and rollback plans. |
| Operations / Monitoring | Run the system in production. | Monitor backlog, freshness, failures, cost, latency, retrieval quality, and alerts. |
| Feedback / Iteration | Improve based on production signals. | Tune chunking, add connectors, improve extraction, reduce failures, and update ADRs. |

### What the interviewer is really asking

When an interviewer asks about SDLC, they are usually testing whether you can work in a mature engineering organization, not just build a prototype.

They want to hear that you understand:

- Requirements gathering
- Design review
- Documentation
- Implementation discipline
- Testing strategy
- Security review
- CI/CD
- Release management
- Monitoring and incident response
- Feedback loops
- Continuous improvement

### SDLC for AI systems

For AI and RAG systems, SDLC needs additional validation steps:

- Dataset or document-corpus analysis
- Evaluation set creation
- Retrieval-quality evaluation
- Groundedness and hallucination checks
- Permission-leakage tests
- Regression tests for prompts, models, chunking, and retrievers
- Human review for high-risk workflows
- Monitoring for quality drift and data drift
- Model, prompt, embedding, and index versioning

### Good interview answer pattern

Use this structure:

1. **Start from requirements and risks**
   - Business goal, users, data sources, security needs, scale, and compliance.

2. **Move to design artifacts**
   - HLSD for architecture, ADRs for major decisions, LLDs for implementation details.

3. **Explain validation**
   - Unit tests, integration tests, load tests, security tests, data-quality checks, retrieval-quality evals.

4. **Explain safe release**
   - CI/CD, feature flags, canary deployment, staged rollout, rollback.

5. **Explain production operation**
   - Monitoring, alerts, runbooks, incident response, retrospectives, continuous improvement.

### Interview-ready framing

> SDLC is the end-to-end process for delivering production software. For an ingestion platform, I would start with requirements and NFRs, then create an HLSD, capture major tradeoffs in ADRs, build the implementation, validate extraction quality and permission correctness, release gradually with feature flags or canaries, and operate it with monitoring, runbooks, and incident review. For AI systems, I would also include retrieval evals, groundedness checks, regression tests for chunking and prompts, and quality monitoring after release.

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

12. **NFR tradeoffs**
    - How scalability, latency, availability, security, compliance, and cost shape the architecture.

13. **SDLC and release safety**
    - How the team moves from requirements to design, implementation, validation, deployment, monitoring, and iteration.

---

## Interview-ready framing

When asked about ingestion architecture, a strong answer is:

> I would start with an HLSD to define the end-to-end ingestion architecture: sources, connectors, sync model, processing pipeline, storage, indexing, access control, retrieval, monitoring, and operational flows. Then I would create ADRs for the major decisions, such as vector database selection, chunking strategy, embedding model, permission enforcement, and reprocessing strategy. The HLSD gives everyone the shared architecture picture, while ADRs preserve the reasoning behind important tradeoffs.

When asked specifically about dataflow:

> I would walk through the lifecycle of a document from the source system to the retrieval path, including connector discovery, extraction, metadata and ACL capture, chunking, embeddings, indexing, storage, and serving. I would also mention where failures are retried and how reprocessing works.

When asked specifically about blast radius:

> I would explain how the design limits the impact of failures. For example, isolate ingestion by tenant and source, use versioned processors and indexes, roll out changes gradually, keep raw data for replay, and make jobs idempotent so a failed or bad run can be safely retried or rolled back.

When asked specifically about NFRs:

> I would separate what the system does from how well it must operate. For ingestion, NFRs include scale, freshness, reliability, security, ACL correctness, latency, observability, cost, and compliance. These requirements directly shape the architecture because they determine queueing, partitioning, storage, indexing, monitoring, and release strategy.

When asked specifically about SDLC:

> I would describe the full delivery process: requirements and NFRs, HLSD and ADRs, implementation, validation, release, monitoring, and iteration. For AI ingestion systems, I would add retrieval-quality evals, permission-leakage tests, groundedness checks, and versioning for prompts, models, chunking, embeddings, and indexes.

---

## Terms to add later

- LLD — Low-Level Design
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
