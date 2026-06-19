# Insight Global — Principal AI Engineer — Technical Interview Q&A

Companion prep doc for `2026-Insight-Global-Principal-AI-Engineer.md`.

Answers are written in first person for interview use. They emphasize Hossein's production enterprise AI experience at Intapp, platform engineering background, and honest positioning on full-stack (strong backend/API/orchestration; lighter on Angular-specific front-end depth).

---

## 1. LLM Systems & "Ask" Platform Design

### Q1. How would you design an internal LLM "Ask" feature for architects and engineers?

**A:** I would treat it as three layers: ingestion, orchestration, and presentation.

1. **Ingestion** — continuously index architecture docs, ADRs, HLSDs, OpenAPI specs, repo metadata, Jira epics, and dependency/config artifacts into searchable stores (vector index + structured metadata in Postgres; graph relationships in Neo4j where useful).
2. **Orchestration** — on each question, classify intent, retrieve grounded context, optionally invoke tools (repo search, schema lookup, graph traversal), generate an answer with citations, and run output validation.
3. **Presentation** — chat UI with source links, confidence indicators, and feedback capture.

At Intapp I built a similar pattern for a natural-language CRM interface used by 10,000+ customers: user question → intent + schema grounding → secure API/tool execution → structured + natural-language response. The same architecture applies here, except the "tools" are architecture-discovery APIs instead of CRM APIs.

### Q2. What context do you retrieve before answering a question about internal systems, requirements, or architecture docs?

**A:** I retrieve in ranked tiers:

- **Direct matches** — docs/chunks semantically similar to the query (hybrid keyword + embedding search).
- **Structural context** — parent doc sections, linked ADRs, related Jira tickets, service owner metadata.
- **Graph context** — upstream/downstream dependencies if the question is about dataflows or blast radius.
- **Freshness/s authority signals** — prefer recently updated docs and official architecture sources over stale wiki pages.

I always attach source IDs so the model can cite and so we can debug bad answers. Enterprise AI is a context problem: the quality of retrieval matters more than model choice.

### Q3. How do you decide between RAG, fine-tuning, and tool use?

**A:**

| Approach | When I use it |
|---|---|
| **RAG** | Default for internal knowledge that changes frequently (docs, code, tickets). Keeps answers grounded and updatable without retraining. |
| **Tool use** | When the answer requires live or structured data: dependency graphs, repo search, schema lookup, ticket status. |
| **Fine-tuning** | Only when we need consistent output format, domain phrasing, or classification behavior that prompts can't stabilize — and when we have enough curated examples. |

For an internal architecture assistant, I would start with **RAG + tools**. Fine-tuning is a later optimization, not day-one.

### Q4. How would you handle questions the system cannot answer confidently?

**A:** I use explicit abstention rather than guessing:

- Set retrieval confidence thresholds; if top chunks score below threshold, say "I don't have enough reliable context."
- Ask a clarifying question when intent is ambiguous ("Do you mean Service A in prod or the legacy batch pipeline?").
- Return partial answers with clear boundaries: "Based on these 2 sources, here's what I know; I could not verify X."
- Log low-confidence queries for content-gap analysis — they tell us what to ingest or document next.

At Intapp, workflow validation and guardrails prevented the system from executing ungrounded API calls; the same principle applies to internal architecture answers.

### Q5. How do you prevent hallucinations when answering questions about system dataflows, APIs, or architecture decisions?

**A:**

1. **Source-backed generation** — require citations for factual claims; post-process to drop unsupported statements.
2. **Structured retrieval first** — for dataflow questions, prefer graph/dependency query results over free-form generation.
3. **Tool-verified facts** — "Service X calls Service Y" should come from OpenAPI, code analysis, or an approved graph edge, not model inference alone.
4. **Output validation** — schema-check generated API summaries against known specs.
5. **Human review for high-risk outputs** — HLSD drafts are suggestions, not auto-published truth.

Hallucination control is architectural, not just prompt engineering.

### Q6. How would you support multi-turn conversations while keeping context bounded?

**A:** I maintain a conversation state object with:

- Original user goal (summarized every few turns to prevent drift).
- Resolved entities (service names, doc IDs, ticket keys).
- Retrieved evidence from prior turns (deduplicated).
- Open questions/clarifications.

Each new turn re-retrieves rather than relying only on chat history — docs may have been updated. I cap token budget by summarizing older turns and keeping the last N evidence chunks. This mirrors how I handled multi-step legal/CRM workflows at Intapp where context had to stay permission-aware and traceable.

### Q7. What metadata would you attach to retrieved chunks?

**A:** Minimum metadata:

- `source_type` (ADR, HLSD, OpenAPI, code, Jira, Confluence)
- `source_id`, `title`, `url/path`
- `owner_team`, `last_updated`, `environment` (prod/dev)
- ` sensitivity / access_class`
- `section_hierarchy` (parent headings)
- `related_entities` (service names, system IDs)
- `embedding_model_version`, `chunk_index`

This enables filtering ("only official ADRs"), permission checks, freshness ranking, and debugging.

### Q8. How would you evaluate whether the "Ask" feature is actually useful?

**A:** I track both offline and online metrics:

**Offline:** golden Q&A set from architects; precision/recall of retrieval; citation accuracy; hallucination rate on known-fact questions.

**Online:** thumbs up/down, "was this cited source helpful?", task completion (did user open linked doc / export HLSD draft?), repeat usage by team, time-to-answer vs manual search.

**Qualitative:** monthly review of failed queries with architecture leads — the best eval signal for internal tools.

### Q9. How do you version prompts, models, and retrieval configs in production?

**A:** Treat them like code:

- Prompt templates and retrieval configs in git with semver tags.
- Model routing config (model name, temperature, max tokens) in versioned config files.
- Deployment via feature flags / canary — 5% traffic on new prompt+retrieval bundle.
- Store `prompt_version`, `model_id`, `retrieval_config_id` with every response for replay and regression analysis.
- CI job runs golden-set eval when prompt or retrieval config changes.

### Q10. How would you route queries across different knowledge domains?

**A:** Lightweight intent classifier (LLM or fine-tuned classifier) maps queries to domains: requirements, architecture/design, dataflows, troubleshooting, SDLC process.

Each domain has:

- Preferred source filters (Jira for requirements, OpenAPI/code for APIs, Neo4j for dependencies).
- Domain-specific tools.
- Domain-specific answer templates (troubleshooting answers include dependency chain; HLSD questions include component diagram suggestions).

Router confidence below threshold → ask clarifying question or search broadly with domain tags in results.

---

## 2. AI Agents & Tool Orchestration

### Q11. Design an AI agent that helps an architect draft an HLSD from requirements and existing system context.

**A:** Agent workflow:

1. **Intake** — parse Jira epic / requirement doc; extract capabilities, constraints, NFRs.
2. **Context gather** — retrieve related ADRs, existing services, data stores, integration patterns, security standards.
3. **Gap analysis** — identify missing info; ask user or create clarification tasks.
4. **Draft HLSD sections** — context diagram, component list, dataflows, API contracts, risks — each section grounded in retrieved sources.
5. **Validate** — check components against approved reference architectures and naming standards.
6. **Human review** — architect edits/approves; feedback stored for eval.

Tools: `search_docs`, `get_jira_epic`, `query_dependency_graph`, `list_reference_architectures`, `validate_hlsd_schema`, `save_draft`.

I designed a similar agentic reference architecture at Intapp covering retrieval, tool execution, audit trails, and human-in-the-loop approval.

### Q12. How do you structure agent planning vs execution for reliability?

**A:** I separate them explicitly:

- **Planner** produces a structured plan (steps, tools, success criteria) — often visible to the user for high-stakes tasks.
- **Executor** runs one step at a time with typed inputs/outputs.
- **Checker** validates step output before proceeding.

For internal SDLC tools, I prefer **constrained plans** (max steps, allowed tool list) over open-ended ReAct loops. Long-horizon autonomy is less important than predictable, auditable behavior.

### Q13. What's your approach to tool selection when an agent has many APIs?

**A:**

- Register tools with clear descriptions and JSON schemas — model selects via function calling.
- Pre-filter tools by domain/intent so the model sees 5–10 relevant tools, not 50.
- Use a two-stage approach for complex tasks: planner picks tool category, executor picks specific tool.
- Fallback: if tool call fails validation, retry with narrowed schema or escalate to user.

At Intapp I standardized secure tool-invocation patterns with validation before execution — same pattern here.

### Q14. How do you handle permission-aware tool execution?

**A:** Every request carries user identity and team scopes. Before any tool call:

1. Policy engine checks user role against resource ACL.
2. Retrieval filters exclude documents/tools the user cannot access.
3. Tool responses are redacted if they would leak cross-team data.
4. Audit log records who asked, what tools ran, what data was accessed.

This is non-negotiable for internal enterprise tools. I implemented permission-aware context retrieval and API execution at Intapp for multi-tenant CRM schemas.

### Q15. How do you implement human-in-the-loop approval?

**A:** Classify actions by risk:

- **Read-only** (search, summarize) — auto-execute.
- **Draft/create** (HLSD draft, requirement summary) — show preview, user confirms save.
- **Write/external** (create Jira, post comment, open PR) — explicit approval with diff preview.

Store approval decisions as training/eval signal. Timeout and cancel long-running agent runs.

### Q16. What failure modes have you seen with agents and how do you guard against them?

**A:** Common failures:

| Failure | Guard |
|---|---|
| Infinite tool loops | Max steps, duplicate-call detection |
| Wrong tool / args | Schema validation, dry-run mode |
| Overconfident answers | Require citations, abstain on low retrieval score |
| Context overflow | Summarize + re-retrieve per step |
| Stale knowledge | Freshness metadata, periodic re-index |

Production agent systems need circuit breakers the same way microservices do.

### Q17. LangChain vs custom orchestration vs lightweight function-calling?

**A:**

- **Function calling (OpenAI/Anthropic native)** — best for straightforward tool use; minimal abstraction.
- **LangChain / LlamaIndex** — useful for rapid prototyping, standard RAG chains; watch for opacity and version churn at scale.
- **Custom orchestration** — my preference for production internal tools where we need strict audit, policy hooks, and testability.

I would prototype with LlamaIndex or LangChain, then harden critical paths in a thin custom orchestration layer.

### Q18. How would an agent help with dataflow discovery across microservices?

**A:** Combine static and dynamic signals:

- **Static:** OpenAPI specs, async event schemas, DB migration files, infra-as-code, service mesh config.
- **Dynamic (optional v2):** sampled trace/log analysis for actual call paths.

Agent tools: `parse_openapi`, `scan_repo_imports`, `query_graph`, `compare_static_vs_runtime`.

Output: proposed dataflow diagram with evidence links per edge. Architect confirms before graph edge is "approved."

### Q19. How do you log and audit agent actions?

**A:** Structured audit events: `session_id`, `user`, `plan`, `tool_name`, `inputs_hash`, `outputs_summary`, `sources`, `latency`, `model_version`, `approval_status`.

Store full payloads in secure object storage with retention policy. Dashboards for error rate, tool latency, approval rejection reasons.

### Q20. How would you test agent behavior before rollout?

**A:**

- **Unit tests** for tool wrappers and validators.
- **Scenario tests** — scripted multi-step tasks with expected tool sequence.
- **Golden trajectories** — approved plans for standard tasks (draft HLSD from epic X).
- **Red-team prompts** — permission bypass attempts, prompt injection via doc content.
- **Shadow mode** — agent runs but output not shown to users; compare to baseline.

---

## 3. Context Engineering & Knowledge Systems

### Q21. What is your definition of context engineering for enterprise AI systems?

**A:** Context engineering is the work of transforming fragmented enterprise data — documents, schemas, APIs, permissions, workflow state, metadata — into a **bounded, trustworthy context package** an LLM can use to reason and act safely.

It includes ingestion, chunking, enrichment, retrieval, ranking, validation, and lifecycle management. Models are interchangeable; context quality is the durable moat. This was the core of my Intapp work for legal, CRM, and financial workflows.

### Q22. How would you ingest content from Confluence, Git, ADRs, HLSDs, and Jira?

**A:** Source-specific connectors → normalized document model → enrichment pipeline → indexes.

- **Confluence/Jira:** API crawl with webhook or scheduled delta sync.
- **Git:** clone/pull, parse Markdown/ADRs, OpenAPI, proto files, CI configs.
- **HLSD/PDF:** structure-aware parsing (headings, tables, diagrams as attachments).

Normalized fields: title, body, author, timestamps, links, entity tags (service names). Push to vector index + Postgres metadata + optional Neo4j entity extraction.

I've built similar ingestion patterns over Kafka/Spark/Airflow at Genapsys and Velti scale.

### Q23. How do you chunk long architecture documents without losing structure?

**A:**

- Split on heading hierarchy (H1/H2/H3) first — keep section path as metadata.
- Tables and code blocks stay intact in single chunks where possible.
- Overlap adjacent chunks slightly (1–2 sentences) for boundary context.
- Store parent-child links so UI can expand full section.
- For diagrams: store alt text + caption + linked components as separate searchable nodes.

Naive fixed-token splitting loses architecture doc semantics quickly.

### Q24. How do you represent relationships between systems, services, data stores, and teams?

**A:** Dual representation:

- **Graph (Neo4j):** `Service -[:CALLS]-> Service`, `Service -[:READS]-> Database`, `Team -[:OWNS]-> Service`.
- **Relational (Postgres):** authoritative registry tables with foreign keys, tags, environment, SLA tier.

Graph for traversal queries; relational for CRUD, permissions, and reporting. Sync graph edges from approved docs and automated parsers; mark unverified edges explicitly.

### Q25. When would you use a vector store vs a graph database vs both?

**A:**

| Store | Best for |
|---|---|
| **Vector** | "Find docs similar to this question" — unstructured search |
| **Graph** | "What depends on X?" — relationship traversal |
| **Both** | Real architecture questions almost always need both: semantic search to find relevant services, graph to show dependencies |

I would not force all queries through one store.

### Q26. How do you keep the knowledge base fresh?

**A:**

- Event-driven re-index on git push, doc update, Jira change.
- Nightly reconciliation job compares source timestamps vs index timestamps.
- Stale content flagged in UI ("last verified 18 months ago").
- Owner notifications for docs referenced often but outdated.

### Q27. How would you deduplicate or reconcile conflicting documentation?

**A:**

- Canonical source hierarchy: ADR > official HLSD > Confluence draft > auto-inferred.
- Show conflicts to users: "Source A says Kafka; Source B says SQS."
- Never silently merge — architects resolve conflicts; resolution stored as override metadata.
- Dedup near-duplicate chunks via embedding similarity + title matching at ingest.

### Q28. What's your approach to schema grounding for natural-language questions over enterprise models?

**A:** At Intapp I mapped customized CRM schemas to domain concepts so NL queries translated to valid API calls. Same pattern here:

1. Maintain machine-readable schema catalog (services, entities, fields, relationships).
2. Retrieve relevant schema subset for the question.
3. Constrain generation to valid entity/field names.
4. Validate generated queries/API calls against schema before execution.

Schema grounding reduces both hallucinations and bad tool calls.

### Q29. Embedding quality, reranking, and hybrid search?

**A:** Default stack:

- **Hybrid retrieval** — BM25/keyword + dense embeddings (reciprocal rank fusion).
- **Reranker** — cross-encoder or lightweight rerank model on top-20 candidates.
- **Filters** — domain, freshness, access class applied pre-ranking.

Evaluate retrieval with MRR/recall@k on architect-provided golden queries. Upgrade embedding model only after offline eval proves gain.

### Q30. How would you build a context graph linking requirements → components → dataflows → owners?

**A:** Extraction pipeline:

- From Jira: requirement IDs, epics, acceptance criteria.
- From HLSD/ADR: components, interfaces.
- From code/OpenAPI: realized dataflows.
- From CMDB/team registry: owners.

Neo4j nodes/edges with provenance (`source_doc`, `confidence`, `verified_by`). LLM assists extraction but human/architect approval promotes edges to "trusted."

---

## 4. Knowledge Graphs & Neo4j

### Q31. What entities and relationships would you model for enterprise architecture dataflow discovery?

**A:** Core entities: `System`, `Service`, `API`, `Topic/Queue`, `Database`, `Team`, `Document`, `Requirement`.

Relationships: `CALLS`, `PUBLISHES_TO`, `SUBSCRIBES_TO`, `READS`, `WRITES`, `OWNED_BY`, `DOCUMENTED_IN`, `IMPLEMENTS`.

Properties: environment, protocol, data classification, last_verified.

### Q32. When is Neo4j the right choice vs Postgres with JSON/graph extensions?

**A:** Neo4j when:

- Frequent multi-hop traversals (3+ hops).
- Architects explore dependencies interactively.
- Graph visualization is first-class.

Postgres when:

- Simple parent-child or 1-hop joins dominate.
- Team prefers one operational database.
- Graph queries are batch, not interactive.

For this role, Neo4j is justified if dataflow discovery and traversal UX are core features.

### Q33. How would you populate a knowledge graph automatically?

**A:** Parsers emit candidate triples:

- OpenAPI → service-to-service calls.
- Terraform/k8s manifests → service-to-database links.
- Kafka topic schemas → pub/sub edges.
- Import statements / RPC stubs in code.

Each edge gets `confidence` and `source`. Promotion workflow: auto-edges visible as "inferred"; architect confirms → "verified."

### Q34. How do you query a graph to answer "What downstream systems depend on Service X?"

**A:** Cypher example pattern:

```cypher
MATCH (s:Service {name: $X})<-[:CALLS|SUBSCRIBES_TO|READS*1..5]-(downstream)
RETURN DISTINCT downstream.name, relationships(path)
```

Return results with path evidence to LLM for natural-language explanation. Cap hop depth to avoid explosion; filter by environment.

### Q35. How do you reconcile graph data with unstructured retrieval results?

**A:** Orchestration layer merges:

1. Graph query returns structured dependency list.
2. Vector search returns explanatory docs.
3. LLM synthesizes answer using graph as source of truth for topology and docs for rationale/constraints.

If they conflict, surface conflict explicitly — don't blend silently.

### Q36. Common pitfalls when building graph-based internal tools?

**A:**

- Treating inferred edges as truth too early.
- Graph sprawl without ownership/governance.
- No freshness → misleading blast-radius analysis.
- Over-modeling before users validate usefulness.
- Letting LLM invent nodes/edges without provenance.

### Q37. How would you expose graph queries to an LLM safely?

**A:** Wrap graph access in typed tools — `get_downstream_dependencies(service, env, max_hops)` — not raw Cypher from the model. Validate inputs against allowlisted service names. Limit result size. Apply user permission filters in the wrapper.

### Q38. How do you handle stale or incomplete graph edges?

**A:** Edge metadata: `last_verified`, `source`, `status`. Stale edges decay in ranking or show warnings. Incomplete coverage dashboard by team/service. Tie verification to SDLC events (service deploy triggers re-validation task).

### Q39. How would you visualize graph-derived dataflows for architects?

**A:** Interactive graph UI (e.g., force-directed or layered dataflow view) with:

- Filter by domain, environment, team.
- Color by verified vs inferred.
- Click node → docs, owners, recent incidents.
- Export to PlantUML/Mermaid for HLSD inclusion.

Angular front-end can host this; graph layout libs exist (Cytoscape, vis.js).

### Q40. How do you migrate/evolve a graph schema as the platform grows?

**A:** Version node/edge labels; migration scripts in git; dual-write during transitions; backward-compatible tool APIs. Document schema changes in platform changelog. Eval pipelines ensure existing golden queries still work after schema migration.

---

## 5. Data Pipelines & Ingestion

### Q41. Design an ingestion pipeline for continuously indexing internal architecture artifacts.

**A:**

```
Sources → Connectors → Raw store (S3) → Parsers → Normalized docs → Enrichment → Vector index + Postgres + Neo4j
                ↑                                                              ↓
           Event bus (webhooks, git push)                              Monitoring / DLQ
```

Workers on Kubernetes or queue consumers; Airflow/Prefect for scheduled reconciliation. Idempotent processing keyed by `(source, doc_id, content_hash)`.

I've built comparable pipelines with Kafka, Spark, and Kubernetes at Genapsys and Velti.

### Q42. What triggers re-indexing?

**A:**

- **Event-driven:** git push, Confluence/Jira webhooks.
- **Scheduled:** nightly full delta scan for missed events.
- **Manual:** architect marks doc as authoritative → priority re-index.
- **Downstream:** schema/catalog change triggers dependent doc re-enrichment.

### Q43. How do you parse OpenAPI, Markdown, PlantUML, PDFs, SQL schemas?

**A:** Dedicated parsers per type:

- OpenAPI/Swagger → structured endpoint + schema objects.
- Markdown/ADRs → heading-aware chunks.
- PlantUML/Mermaid → extract declared components/edges (best-effort).
- PDF → layout-aware extraction; diagrams often need manual alt text.
- SQL DDL → table/column/FK graph contributions.

Parser outputs land in a common `NormalizedDocument` schema.

### Q44. How would you build a pipeline to infer dataflows from code, configs, and logs?

**A:** Phase 1 static only (lower risk):

- Static call graph from OpenAPI + RPC imports + message schema links.
- Config parsers for queue/topic bindings.

Phase 2 optional dynamic:

- Sample distributed traces (Jaeger/Tempo) to validate or discover runtime-only paths.

Label inferred vs observed edges separately in the graph.

### Q45. Large repos and incremental updates?

**A:** Track `content_hash` per file; only re-embed changed files. Shallow fetch + diff against last indexed commit. Parallelize by repo/module. Rate-limit API calls to source systems. Maintain tombstone index for deleted files.

### Q46. Idempotency and retry strategy?

**A:** Each job keyed by `(source, artifact_id, version)`. Writes are upserts. Retries with exponential backoff; poison messages to DLQ after N failures with alert. Reprocessing DLQ is safe because upserts are idempotent.

### Q47. PII/secrets during ingestion?

**A:** Pre-index scanners for API keys, tokens, SSN patterns; block or redact before embedding. Respect source-system ACLs at ingest time — don't index what the search user shouldn't see. Secrets scanning in CI for connector code too.

### Q48. Monitor pipeline health?

**A:** Metrics: lag per source, docs/min throughput, parse failure rate, embedding queue depth, index freshness SLA, DLQ size. Alerts on staleness > 24h for critical sources. Dashboard per connector.

### Q49. Postgres vs object storage vs search index — what lives where?

**A:**

| Data | Store |
|---|---|
| Raw artifacts | Object storage (S3) |
| Metadata, ACLs, job state | Postgres |
| Embeddings + text for search | Vector DB or pgvector |
| Relationships | Neo4j |
| Audit logs | Postgres + log aggregator |

### Q50. Backfill historical docs without overloading systems?

**A:** Throttled batch jobs off-peak; prioritize high-value corpora first (ADRs, official HLSDs). Checkpoint progress. Separate backfill queue from real-time delta queue. Monitor source API rate limits.

---

## 6. SDLC Automation & Architecture Tooling

### Q51. How would AI help with requirement definition without vague specs?

**A:** AI drafts **structured** requirements from epics/interviews:

- User stories with acceptance criteria templates.
- Explicit open questions section.
- Trace links to source Jira text — no unsupported additions.
- Architect edits before publish.

Constraint prompts: "Do not invent NFRs; mark unknowns as TBD."

### Q52. What inputs for a first-draft HLSD?

**A:** Jira epic, related ADRs, reference architecture library, dependency graph for affected services, security/compliance standards, existing HLSDs for similar systems.

### Q53. Traceability from generated architecture docs to requirements?

**A:** Each HLSD section carries `requirement_ids[]` and `source_doc_refs[]`. Generated content stored as draft with provenance. Diff view shows what came from which source. Matches compliance expectations from my regulated enterprise AI work at Intapp.

### Q54. Identify gaps between documented architecture and implementation?

**A:** Compare:

- HLSD components vs repo/service registry.
- OpenAPI declared endpoints vs code handlers.
- Graph inferred edges vs documented edges.

Report: documented-but-not-found, implemented-but-not-documented.

### Q55. AI during architecture review vs initial design?

**A:**

- **Initial design:** brainstorming, draft components, pattern suggestions, similarity to past systems.
- **Review:** checklist against standards, missing NFRs, dependency risk, security gaps, inconsistency detection.

Review mode should be more conservative — flag issues with evidence, not redesign everything.

### Q56. AI-accelerated troubleshooting via dependencies?

**A:** Given symptom (latency, errors), agent retrieves recent changes, upstream/downstream graph, runbooks, past incidents. Output: ranked hypotheses with evidence links — not a single guessed root cause.

Similar to event-dispatch troubleshooting I worked on at AutoGrid in high-reliability environments.

### Q57. Integrate with Jira, GitHub, Confluence, CI/CD?

**A:** OAuth/service accounts per system; read-first integrations initially. Webhooks from git/Jira drive index freshness. Optional write-back (create sub-tasks, link ADRs) behind approval. Respect enterprise SSO and token rotation policies.

### Q58. Guardrails against unsafe architectural shortcuts?

**A:** Policy rules encoded: no public data exposure, approved patterns only, mandatory security controls for PII systems. Validator checks generated designs against rule engine before export. High-risk violations block save until acknowledged.

### Q59. Actionable suggestions for senior vs junior engineers?

**A:** Adjustable detail level:

- **Senior:** concise deltas, risk flags, alternatives.
- **Junior:** step-by-step explanations, links to reference architectures, glossary.

Same backend; different presentation templates.

### Q60. Measure productivity impact on SDLC?

**A:** Baseline time for HLSD draft, requirement clarification cycles, mean time to find architecture info. Post-rollout: same metrics + adoption + qualitative architect interviews. A/B teams if possible.

---

## 7. Full-Stack & API Design (JS/TS, Node, Angular)

### Q61. Architect the backend for an internal AI platform using Node.js/TypeScript?

**A:** Services:

- **API gateway** — auth, rate limit, request routing.
- **Chat/orchestration service** — session management, LLM calls, tool routing.
- **Retrieval service** — hybrid search, rerank.
- **Ingestion workers** — async indexing.
- **Graph service** — Neo4j queries behind typed API.

Postgres for sessions/metadata; Redis for rate limits/caches; message queue for async jobs. TypeScript for shared types between API and tooling clients.

My strongest day-to-day implementation is Python/FastAPI, but the service boundaries and API contracts transfer directly — I've built production REST/gRPC services in Java/Spring and Python for years.

### Q62. API shape for chat, retrieval, tool invocation, feedback?

**A:** Example endpoints:

- `POST /v1/sessions/{id}/messages` — stream chat response (SSE/WebSocket).
- `POST /v1/retrieve` — debug retrieval (internal).
- `POST /v1/tools/{toolName}/invoke` — guarded tool exec.
- `POST /v1/feedback` — thumbs, citation helpfulness.
- `GET /v1/documents/{id}` — fetch source for citation panel.

All responses include `trace_id`, `model_version`, `citations[]`.

### Q63. Streaming responses to the UI?

**A:** SSE or WebSocket from orchestration service. Stream tokens plus structured events (`citation`, `tool_start`, `tool_end`, `done`). Client renders incrementally; cancel button aborts upstream LLM call.

### Q64. What the Angular frontend needs?

**A:** Chat pane, citation sidebar, source preview, confidence/abstention states, approval modals for agent actions, feedback widgets, optional graph visualization tab. State management (NgRx or signals) for session + streaming partials.

I'm lighter on Angular-specific implementation than backend AI orchestration; I'd partner with a front-end engineer or ramp quickly on component patterns while owning API/orchestration.

### Q65. Auth/session with corporate SSO?

**A:** OIDC/SAML via enterprise IdP; JWT or session cookie for API auth. Propagate user groups/roles to retrieval ACL filters and tool policies. Short-lived tokens; refresh handled server-side.

### Q66. Monorepo or service split?

**A:** Monorepo early for velocity (shared types, single CI). Split ingestion workers when scale/isolation demands. Keep orchestration + API together initially to reduce deployment complexity.

### Q67. REST vs gRPC vs WebSockets?

**A:**

- **REST/JSON** — external API, CRUD, feedback.
- **gRPC** — internal service-to-service if low latency needed.
- **WebSockets/SSE** — streaming chat.

I've used REST and gRPC extensively; choose based on team familiarity and client needs.

### Q68. APIs consumable by agents and humans?

**A:** Same core APIs; agents call typed tool wrappers that hit internal REST endpoints. Human UI uses identical backend — no duplicate business logic. OpenAPI spec published for both.

### Q69. Caching strategy?

**A:** Cache retrieval results keyed by `(query_hash, user_scope, index_version)` with short TTL. Cache graph traversals for hot services. Invalidate on re-index events for affected docs. Never cache across permission boundaries.

### Q70. Frontend sync with long-running agent tasks?

**A:** Job ID returned immediately; WebSocket pushes step progress (`planning`, `retrieved 5 docs`, `drafting section 2`). Poll fallback if WebSocket drops. Persist job state in Postgres for refresh/resume.

---

## 8. Databases (Postgres, SQL Server, Neo4j)

### Q71. Relational schema for conversations, citations, feedback?

**A:** Core tables:

- `users`, `sessions`, `messages` (role, content, model_version)
- `citations` (message_id, doc_id, chunk_id, span)
- `feedback` (message_id, rating, comment)
- `documents` (source metadata, hash, indexed_at)
- `ingestion_jobs` (status, error)

Foreign keys + indexes on `session_id`, `user_id`, `created_at`.

### Q72. Store embeddings — pgvector vs separate vector DB?

**A:** **pgvector** — simpler ops, good to ~few million chunks, strong metadata joins.

**Dedicated vector DB (Pinecone, Weaviate, etc.)** — higher scale or advanced hybrid features.

I'd start pgvector + Postgres metadata unless scale tests say otherwise — aligns with role's Postgres requirement.

### Q73. Query across relational metadata and graph efficiently?

**A:** Application layer orchestrates: Postgres lookup for doc ACL + Neo4j traversal for dependencies. Don't try one SQL query across both. Cache hot graph results. Optional materialized views in Postgres for common graph summaries.

### Q74. Indexing strategy for document search + metadata filters?

**A:** GIN/BTREE on metadata filters (source_type, team, updated_at). pgvector HNSW/IVFFlat for embeddings. Composite filter-then-search pattern: narrow in SQL, vector search within candidate set for better performance and ACL enforcement.

### Q75. Migrations in a fast-evolving internal tool?

**A:** Alembic/Flyway migrations in git; backward-compatible expand-contract pattern; feature flags for code depending on new columns. Weekly migration review so debt doesn't accumulate.

### Q76. Model user permissions in Postgres for multi-team enterprise use?

**A:** RBAC: `users`, `roles`, `teams`, `resources`, `resource_grants`. Optional ABAC attributes (env, data_class). Enforce in retrieval layer and API middleware — single policy module shared by chat and tools.

### Q77. When denormalize for read performance?

**A:** Denormalize read-heavy dashboards: popular docs, service summary cards, precomputed dependency counts. Keep writes normalized. Refresh async on index updates.

### Q78. Transactional consistency between ingestion state and search index?

**A:** Outbox pattern: commit doc metadata + outbox event in one Postgres transaction; indexer consumes outbox and upserts vector/graph indexes. Reconciliation job fixes drift. Search may lag seconds — acceptable for internal docs.

### Q79. Audit tables?

**A:** `audit_log` append-only: user, action, resource, timestamp, payload hash. Separate from application logs. Retention per compliance policy.

### Q80. Analytics on usage and answer quality?

**A:** `query_events`, `retrieval_metrics`, `feedback` aggregated nightly. Dashboards: queries/team, top unanswered topics, citation click-through, average retrieval latency, low-rated clusters for content improvement.

---

## 9. Architecture Tradeoffs & Principal-Level Judgment

### Q81. Build vs buy for LLM orchestration, vector search, observability?

**A:**

| Component | Recommendation |
|---|---|
| LLM API | Buy (OpenAI/Anthropic/Azure) — don't host weights initially |
| Orchestration | Build thin layer after prototype |
| Vector search | Buy managed or pgvector — don't operate custom ANN early |
| Observability | Buy (Langfuse/Arize/Helicone or OpenTelemetry + custom dashboards) |

Buy commodity, build differentiation (ingestion, graph, SDLC workflows, policy).

### Q82. Phase an MVP in 90 days?

**A:**

- **Days 1–30:** ingest ADRs + Confluence subset; basic RAG Ask with citations; SSO auth.
- **Days 31–60:** Jira + git OpenAPI ingestion; feedback loop; eval golden set.
- **Days 61–90:** one agent workflow (requirement summary OR HLSD draft); typed dependency query v1; internal pilot with 10 architects.

Explicitly defer: full graph automation, write-back to Jira, advanced visualizations.

### Q83. What would you NOT automate in v1?

**A:** Final HLSD approval, production architecture sign-off, creating production infra tickets, cross-team dependency commitments, anything that modifies external systems without human approval.

### Q84. Balance speed vs maintainability?

**A:** Move fast on prompts, retrieval tuning, and UX experiments — version and measure. Move carefully on data models, permission boundaries, and graph truth — hard to unwind. Principal role is knowing which layer you're optimizing.

### Q85. Monolith vs microservices for internal AI dev tool?

**A:** **Modular monolith** first — one deployable with clear internal modules (ingest, retrieve, chat, graph). Split workers when ingestion load isolates. Internal tools rarely need microservices complexity on day one.

### Q86. Choose models (cost, latency, quality, privacy)?

**A:** Tiered routing:

- Simple retrieval QA → smaller/faster model.
- HLSD drafting / complex reasoning → stronger model.
- Sensitive corpora → enterprise/private endpoint if required.

Benchmark on golden tasks; track cost per successful task, not cost per token alone.

### Q87. Strategy if requirements shift from Ask to autonomous SDLC agents?

**A:** Architecture already supports it if orchestration, tools, audit, and human approval are first-class. Add agent plans incrementally; don't rewrite retrieval/graph layers. Modular tool registry makes new agents compositional.

### Q88. Avoid building a tool architects don't trust?

**A:** Citations always visible; show confidence and conflicts; never hide inferred vs verified; fast feedback loop with architecture team; ship small accurate wins before ambitious automation; architects co-own golden eval sets.

### Q89. Acceptable vs dangerous technical debt?

**A:** **Acceptable:** UI polish, manual ingestion for edge doc types, hardcoded prompts with tests.

**Dangerous:** weak ACL model, unversioned prompts, no eval pipeline, treating inferred graph as truth, unaudited agent write actions.

### Q90. Platform success after 6 months?

**A:**

- ≥60% weekly active architects/lead engineers in pilot org.
- Median architecture question answered with useful citation in <30s.
- Measurable reduction in time to draft HLSD/requirements (self-reported + sample timing).
- Low hallucination rate on golden set (<5% unsupported factual errors).
- Clear backlog of v2 features driven by user demand, not speculation.

---

## 10. AI-Assisted Development & Engineering Practices

### Q91. How do you use AI-assisted coding in your workflow?

**A:** I use Cursor/Copilot for boilerplate, test scaffolding, parser stubs, and refactors — but I always review generated code for edge cases, security, and fit with existing patterns. For LLM features, I dogfood the same eval/discipline I expect the team to follow.

### Q92. Standards for teams using Copilot/Cursor on this codebase?

**A:**

- No committed secrets or proprietary prompts in public tools without policy review.
- PR author owns AI-generated code — same review bar as human code.
- Require tests for AI-touched business logic.
- Document architectural decisions in ADRs, not only in chat logs.

### Q93. Review AI-generated code differently?

**A:** Extra scrutiny on: error handling, injection risks, silent failures, missing auth checks, hallucinated APIs/library calls. Run static analysis + unit tests + security scan. Ask "what would break in prod?"

### Q94. Mandatory tests for LLM-integrated features?

**A:** Retrieval unit tests, golden Q&A regression, tool schema validation tests, permission-denial tests, prompt snapshot tests, latency smoke tests in CI.

### Q95. PR review for prompt changes?

**A:** Prompt diffs reviewed like code — require linked eval results showing no regression on golden set. Tag reviewer who owns domain (architecture team representative for SDLC prompts).

### Q96. CI checks for retrieval quality regressions?

**A:** Nightly or PR-triggered eval job: recall@k, citation accuracy, abstention behavior on unanswerable questions. Fail build on significant regression thresholds.

### Q97. Document nondeterministic LLM behavior?

**A:** Document expected variability, eval ranges, fallback behavior, model/version dependencies. Runbooks for "answer quality dropped" — check index freshness, model change, prompt version.

### Q98. Roll out new agent capability?

**A:** Feature flag → internal dogfood → shadow mode → 10% canary → full rollout. Kill switch ready. Monitor error rate and feedback sentiment each stage.

### Q99. Debug a bad answer in production?

**A:** Replay with stored `trace_id`: user query → retrieval candidates + scores → prompt → model output → post-processing. Check index freshness, ACL filtering, prompt version, model change. Fix root cause (content gap vs retrieval vs prompt), add case to golden set.

### Q100. Observability stack?

**A:** OpenTelemetry traces across API → retrieval → LLM → tools; structured logs with session/trace IDs; prompt/response capture in secure store; Langfuse or similar for LLM analytics; dashboards for latency, cost, eval scores, feedback; alerts on error spikes and retrieval empty-rate.

---

## 11. System Design & Live Exercise Prep

### Q101. System design: internal AI assistant for architects (dataflows + HLSD drafts)

**A:** See Q1, Q11, Q41, Q82 for integrated answer. Summary architecture:

```
[Angular UI] → [API/Orchestration (Node or Python)] → [LLM API]
                              ↓
        [Retrieval] ← [Vector index + Postgres metadata]
                              ↓
        [Graph service (Neo4j)] ← [Ingestion workers] ← [Git/Jira/Confluence/OpenAPI]
```

Non-functional: SSO, ACL-aware retrieval, audit, eval pipeline, 99.5% availability target for internal tool, p95 chat latency <5s excluding long agent runs.

### Q102. Coding exercise: small RAG endpoint with citations

**A:** Implementation sketch (Python/FastAPI — my strongest stack):

1. Load/chunk docs offline into pgvector.
2. `POST /ask` — embed query, hybrid search top-k, build prompt with numbered sources, call LLM.
3. Return `{answer, citations: [{id, title, snippet, score}]}`.
4. Refuse if max score < threshold.

I'd implement in interview with clear function boundaries and tests for retrieval.

### Q103. API design exercise: chat, feedback, upload, graph lookup

**A:** RESTful resources as in Q62; OpenAPI spec first; pagination on history; idempotency keys for feedback; async `POST /ingest/jobs` for uploads with job status polling.

### Q104. Data modeling exercise: Postgres + Neo4j

**A:** Postgres: users, sessions, messages, documents, chunks, citations, feedback, ingestion_jobs, ACL grants.

Neo4j: Service, Database, Team, Document nodes; CALLS, READS, OWNS, DOCUMENTED_IN edges with provenance properties.

### Q105. Agent workflow: Jira epic → requirement summary + architecture impacts

**A:** Steps: fetch epic → retrieve similar past epics + ADRs → extract requirements → query graph for affected services → summarize impacts → present draft with citations → user approves → optional Jira comment draft (not auto-post in v1).

### Q106. Debugging scenario: confident but wrong answers

**A:** Diagnose in order: (1) stale/wrong docs retrieved, (2) missing ACL filtering caused wrong context, (3) prompt not enforcing citations, (4) model upgrade regression, (5) conflicting docs not surfaced. Fix retrieval + validation first; re-run golden eval; add failing case permanently.

### Q107. Tradeoff: 8 weeks, 2 engineers — what do you cut?

**A:** Ship: Confluence+ADR RAG Ask, citations, SSO, feedback, golden eval.

Cut: Neo4j automation (manual CSV import OK), HLSD agent (summaries only), Angular polish (basic chat UI), dynamic trace analysis, Jira write-back.

One engineer backend/AI, one full-stack — I would own orchestration/retrieval/ingest and pair on API contract + minimal UI.

---

## Quick Reference — Top 10 to Rehearse Aloud

1. **End-to-end "Ask" architecture** — Q1
2. **RAG vs tools vs fine-tuning** — Q3
3. **Hallucination prevention** — Q5
4. **HLSD agent design** — Q11
5. **Permission-aware retrieval/tools** — Q14
6. **Context engineering definition** — Q21
7. **Vector + graph together** — Q25
8. **Ingestion pipeline** — Q41
9. **90-day MVP phasing** — Q82
10. **Debug bad production answer** — Q99

---

## Top 10 — 30-Second Spoken Versions

Use these as openers in live interviews; expand with details if the interviewer probes.

### 1. End-to-end "Ask" architecture (Q1)

"I'd design it in three layers: ingestion, orchestration, and UI. Ingestion continuously indexes ADRs, HLSDs, OpenAPI specs, Jira, and repo metadata into a vector index plus Postgres metadata, and optionally a graph for dependencies. Orchestration classifies the question, retrieves grounded context, calls tools when needed, and returns cited answers with validation. The UI is chat plus source links and feedback. I built the same pattern at Intapp — natural language to secure API calls for 10,000-plus CRM users — so the shape transfers directly to an internal architecture assistant."

### 2. RAG vs tools vs fine-tuning (Q3)

"I default to RAG for anything that changes frequently — docs, code, tickets — because it keeps answers grounded and updatable. I add tool use when the answer needs live or structured data: dependency graphs, repo search, schema lookup. Fine-tuning is later, only when we need stable output format or classification behavior that prompting can't hold. For an internal architecture tool, I'd ship RAG plus tools first and treat fine-tuning as an optimization, not a prerequisite."

### 3. Hallucination prevention (Q5)

"Hallucination control is architectural, not just prompting. I require source-backed generation with citations, use structured retrieval and graph queries for factual topology, validate generated API or dataflow claims against OpenAPI and approved specs, and abstain when retrieval confidence is low. At Intapp we blocked ungrounded API execution the same way — the model doesn't get to invent system behavior. For HLSD drafts, output is a suggestion with provenance, not auto-published truth."

### 4. HLSD agent design (Q11)

"I'd structure it as intake, context gather, gap analysis, draft, validate, and human review. The agent pulls the Jira epic, retrieves related ADRs and dependency context, flags missing information, drafts HLSD sections with citations, validates against reference architectures, and stops for architect approval before save. Tools are search, graph lookup, and schema validation — not open-ended autonomy. I designed a similar agentic reference architecture at Intapp with audit trails and human-in-the-loop approval, which is the right bar for SDLC tooling."

### 5. Permission-aware retrieval/tools (Q14)

"Every request carries user identity and team scope. Retrieval filters exclude documents the user can't access, the policy engine checks each tool call against resource ACLs, and responses are redacted if they'd leak cross-team data. Audit logs capture who asked, what tools ran, and what was accessed. I implemented this at Intapp for multi-tenant CRM schemas — permission-aware context and API execution — and it's non-negotiable for internal enterprise tools."

### 6. Context engineering definition (Q21)

"Context engineering is turning fragmented enterprise data — documents, schemas, APIs, permissions, workflow state — into a bounded, trustworthy context package the model can reason over safely. It's ingestion, chunking, enrichment, retrieval, ranking, validation, and lifecycle management. Models are interchangeable; context quality is the durable advantage. That was the core of my Intapp work for legal, CRM, and financial workflows, and it's exactly what an internal architecture knowledge platform needs."

### 7. Vector + graph together (Q25)

"Vector search answers 'find docs like this question.' Graph traversal answers 'what depends on Service X?' Real architecture questions usually need both — semantic search to find the right services and docs, then graph walks for blast radius and dataflows. I wouldn't force everything through one store. The orchestration layer merges structured graph results with retrieved documents and makes conflicts visible instead of blending them silently."

### 8. Ingestion pipeline (Q41)

"I'd use source connectors — git, Confluence, Jira — into a raw store, parsers into a normalized document model, enrichment for entity tags, then parallel writes to vector index, Postgres metadata, and Neo4j for relationships. Event-driven on git push and doc webhooks, plus nightly reconciliation. Jobs are idempotent on content hash, failures go to a DLQ. I've built comparable pipelines with Kafka, Spark, and Kubernetes at Genapsys and Velti, so the operational patterns are familiar."

### 9. 90-day MVP phasing (Q82)

"First 30 days: ingest ADRs and a Confluence slice, ship basic RAG Ask with citations and SSO. Days 31 to 60: add Jira and OpenAPI from git, feedback loop, golden eval set. Days 61 to 90: one agent workflow — requirement summary or HLSD draft — plus typed dependency queries and a pilot with ten architects. I'd explicitly defer full graph automation, Jira write-back, and fancy visualizations. The goal is a trusted Ask feature first, then automation on top."

### 10. Debug bad production answer (Q99)

"I replay the trace ID end to end: query, retrieval candidates and scores, prompt version, model output, post-processing. I check index freshness, ACL filtering, prompt or model changes, and conflicting source docs. Usually it's a retrieval or content-gap problem, not the model. I fix root cause, add the case to the golden set, and re-run eval before redeploying. At Intapp we treated LLM regressions like any production incident — reproducible traces and permanent test cases."

---

## Honest Positioning Notes

**Lean in:** enterprise context engineering, NL→API/tool orchestration, RAG, agents with guardrails, ingestion pipelines, Postgres/Kafka/K8s, architecture-level thinking, Intapp production GenAI at scale.

**Address directly if asked:** strongest implementation in Python/FastAPI and JVM services; transferable API design to Node/TypeScript; will ramp on Angular component patterns or pair with front-end specialist; Neo4j — solid conceptual and query design, will deepen operational expertise on the job.

**Intapp proof points to repeat:** 10,000+ customer NL CRM interface; schema grounding; secure tool orchestration; compliant narrative generation; forward-deployed stakeholder work.
