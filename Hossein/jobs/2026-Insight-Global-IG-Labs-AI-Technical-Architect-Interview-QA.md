# Insight Global / IG Labs — AI Technical Architect — Technical Interview Q&A

Companion prep doc for `2026-Insight-Global-IG-Labs-AI-Technical-Architect.md`.

Answers are written in first person for interview use. They emphasize client-facing enterprise AI architecture, reusable reference architectures, context engineering from Intapp, and honest positioning on consulting pre-sales and data platforms (strong on Spark/cloud pipelines; partner-level depth on Snowflake/Databricks specifics).

---

## 1. Role Fit & Consulting Mindset

### Q1. Why IG Labs / AI Technical Architect, and why you?

**A:** This role sits exactly where I've been operating — client-facing enterprise AI architecture plus reusable patterns that scale. At Intapp I worked in a forward-deployed model with legal, CRM, and financial-services stakeholders, translating ambiguous workflows into production GenAI systems. IG Labs adds the consulting layer I want: define architectures with CTO/VP/CDO clients, deliver in real environments, then codify what worked into playbooks and reference architectures for the next engagement. That's architecture with leverage, not one-off designs.

### Q2. How is an AI Technical Architect different from a Principal AI Engineer in your view?

**A:** The Principal AI Engineer role is deeper on building a specific internal platform — agents, ingestion, full-stack delivery. The AI Technical Architect role is broader on **client discovery, target-state architecture, reference patterns, and practice-scale IP**. I can go deep on implementation when needed, but the architect role optimizes for stakeholder alignment, reusable frameworks, and cross-client consistency. I've done both architecture ownership and hands-on delivery; this role weights the former more heavily.

### Q3. How do you handle CST/EST hours from California?

**A:** I'm on Pacific time but routinely work with East Coast and European stakeholders. I can block 8–12 PT for client calls and collaboration, use async written architecture docs for deep work, and reserve afternoons for build/review cycles. I would confirm exact core hours with the team upfront.

### Q4. What does "consulting + product thinking" mean to you?

**A:** Every client engagement should produce two outputs: a **client-specific solution** and a **reusable asset** — reference architecture, playbook module, eval template, or connector pattern. Product thinking means versioning those assets, measuring reuse, and designing for configurability rather than rewriting from scratch each time. Consulting without IP capture doesn't scale a practice.

### Q5. How do you stay close to delivery without becoming a full-time implementer?

**A:** I stay hands-on on architecture spikes, reference implementations, and critical path reviews — not every sprint ticket. I join design reviews, unblock integration decisions, pair on eval/governance setup, and validate that what's built matches the reference model. At Intapp I both set architecture direction and shipped production NLP/GenAI features; I know where to dive in vs delegate.

---

## 2. Client Discovery & Stakeholder Architecture

### Q6. Walk through how you run AI architecture discovery with a C-level client.

**A:** I structure discovery in four passes:

1. **Business outcomes** — what decisions or workflows should AI improve; success metrics.
2. **Current state** — data sources, systems, teams, constraints, compliance, existing AI experiments.
3. **Risk & trust** — accuracy requirements, human review, audit, data residency, vendor policy.
4. **Target architecture** — phased roadmap with MVP, platform layers, and explicit non-goals.

I deliver a short readout within the first week: problem framing, options, recommendation, and open questions. I avoid jumping to "let's do RAG" before understanding whether the bottleneck is data, workflow, or governance.

### Q7. How do you translate vague client asks like "we want an AI copilot" into an architecture?

**A:** I decompose into: **users**, **tasks**, **context sources**, **actions**, **risk tier**, and **integration surfaces**. Example: "copilot for sales" becomes — who (AEs), what tasks (CRM lookup, email draft, forecast Q&A), what data (CRM, email, product docs), what actions (read-only vs write-back), what compliance rules apply. That yields a concrete architecture: retrieval layer, schema grounding, tool APIs, approval workflow, eval plan. I did this at Intapp for legal/CRM NL interfaces serving 10,000+ customers.

### Q8. How do you communicate architecture to non-technical executives?

**A:** Lead with outcomes, risks, and phasing — not stack diagrams. Use one-page visuals: user journey, data flow, trust boundaries, timeline. Quantify tradeoffs ("Option A ships in 8 weeks with read-only Q&A; Option B adds write actions in 16 weeks with approval workflow"). Tie recommendations to their KPIs. Provide a technical appendix for engineering leaders.

### Q9. How do you handle clients who want to fine-tune a model before fixing their data?

**A:** I explain respectfully that **enterprise AI is a context problem**. Fine-tuning on messy data encodes mess. I propose a phased plan: inventory and access controls first, retrieval MVP second, fine-tuning only where eval proves need (format, classification, domain phrasing). I share Intapp examples where schema grounding and context layers delivered more value than model changes.

### Q10. How do you run architecture reviews with VP Engineering / CDO stakeholders?

**A:** Pre-read architecture doc with decision log (options considered, chosen pattern, rationale). Review agenda: principles alignment, NFRs (scale, latency, security), integration points, operability, cost model, open risks. End with explicit decisions and owners. Follow up with updated reference diagram and backlog items.

### Q11. How do you contribute to pre-sales without overpromising?

**A:** I join discovery calls to assess feasibility, scope MVP vs phase 2, and flag data/governance blockers early. In proposals I separate **committed capabilities** from **assumptions requiring validation**. I provide reference architectures showing what's proven vs what would be client-specific. Never promise autonomous agents or "100% accuracy" — promise grounded workflows, eval discipline, and phased delivery.

### Q12. Client wants multi-cloud AI on AWS and Azure — how do you approach it?

**A:** Define a **cloud-agnostic application layer** (orchestration, eval, policy) and cloud-specific managed services underneath (embedding, vector store, identity, logging). Avoid duplicating data long-term across clouds; pick primary data plane per workload. Use IaC modules per cloud with shared interface contracts. Document portability boundaries honestly — some components will be cloud-native.

---

## 3. Reference Architectures & Reusable IP

### Q13. What is a reference architecture in the IG Labs context?

**A:** A validated pattern bundle: context diagram, component responsibilities, integration contracts, security model, deployment topology, eval/governance checklist, and implementation notes — parameterized for client context. It's not a slide deck; it's something engineers can implement repeatedly with configuration, not reinvention.

### Q14. How do you extract reusable IP from a client engagement?

**A:** During delivery I tag decisions as **client-specific** vs **generalizable**. After MVP ship, run a retrospective: what modules could be templated (ingestion connector, RAG service, agent approval workflow, eval harness). Generalizable pieces get cleaned, documented, and versioned in the IG Labs IP library with anonymized examples and config hooks.

### Q15. Example reference architecture you've conceptually built before?

**A:** At Intapp I standardized patterns for: **AI-ready context construction**, **schema-grounded NL interfaces**, **secure tool orchestration**, **narrative generation with validation**, and **human-in-the-loop governance**. I also designed a reference architecture for **agentic enterprise workflows** covering retrieval, permission-aware connectors, audit trails, and eval/guardrails — directly reusable as an IG Labs playbook for regulated clients.

### Q16. How do you version and maintain playbooks across clients?

**A:** Semver for playbook modules; changelog with breaking changes; compatibility matrix (cloud, model provider, data platform). Each client engagement pins to a playbook version. Central team owns core patterns; engagement teams contribute PRs back. Deprecation policy for outdated patterns (e.g., naive chunk-only RAG without eval).

### Q17. How do you balance client customization vs standardization?

**A:** **Standardize the platform layers** (ingestion framework, retrieval service, policy engine, eval harness, observability). **Customize domain adapters** (schemas, prompts, tools, UI workflows). If a customization would fork core architecture, escalate — either promote to playbook or keep as client fork with maintenance cost acknowledged.

### Q18. How would you build an IG Labs "Enterprise GenAI Starter" playbook?

**A:** Modules:

1. Discovery questionnaire + readiness scorecard
2. Context layer design guide (documents, APIs, permissions)
3. RAG reference implementation with hybrid search + rerank
4. Agent pattern with approval tiers
5. Eval golden-set template + CI integration
6. Governance checklist (PII, model routing, audit)
7. Deployment templates (K8s/serverless) for AWS/Azure
8. Runbook for production support

Each module: architecture doc + reference code + client config examples.

### Q19. How do you measure playbook reuse and quality?

**A:** Metrics: engagements using playbook vN, time-to-MVP vs bespoke baseline, defect/regression rate, client satisfaction on architecture clarity, engineer onboarding time. Qualitative: engagement leads report what's missing; update backlog quarterly.

### Q20. How do you document architecture decisions for reuse?

**A:** ADR format: context, decision, alternatives, consequences, client applicability tags. Link ADRs to playbook modules and diagrams. Store in searchable repo — future engagements find "how we handled permission-aware retrieval for financial services" quickly.

---

## 4. Enterprise GenAI & RAG Architecture

### Q21. What is context engineering, and why does it matter for clients?

**A:** Context engineering transforms fragmented client data — documents, schemas, APIs, workflow artifacts, permissions — into **bounded, trustworthy context** for LLM systems. It's the highest-leverage work in enterprise GenAI. Clients often underestimate it; I make it explicit in every architecture because model choice matters less than whether the system can access the right information safely.

### Q22. Design a production RAG architecture for a regulated enterprise client.

**A:**

```
Sources → Ingestion → Normalized docs + metadata (Postgres)
                   → Embeddings (vector store / pgvector)
User query → AuthZ filter → Hybrid retrieval → Rerank → Context pack
          → LLM generation → Citation enforcement → Output validation → Response
Parallel: eval pipeline, observability, feedback loop, index freshness monitors
```

Key NFRs: permission-aware retrieval, source citations, abstention on low confidence, audit logging, model/version tracking.

### Q23. RAG vs fine-tuning vs agents — how do you recommend to clients?

**A:**

| Pattern | Recommend when |
|---|---|
| **RAG** | Knowledge changes often; need citations; multiple source types |
| **Fine-tuning** | Stable format/domain phrasing; enough curated examples; retrieval alone insufficient |
| **Agents/tools** | Tasks require live data, multi-step workflows, or system actions |

Default enterprise path: **RAG + policy + eval**, add agents for high-value workflows, fine-tune selectively after metrics justify it.

### Q24. How do you prevent hallucinations in client-facing GenAI systems?

**A:** Source-backed answers with mandatory citations; retrieval confidence thresholds; structured data via tools not generation; output schema validation; human review for high-risk tiers; continuous eval on client golden questions. At Intapp we blocked ungrounded API actions — same principle for client copilots.

### Q25. How do you design schema grounding for customized enterprise data models?

**A:** Maintain machine-readable schema/catalog; retrieve relevant subset per query; constrain NL-to-API translation to valid entities/fields; validate generated queries before execution; log grounding failures for improvement. I shipped this for customized CRM schemas at Intapp for 10,000+ customers.

### Q25b. Hybrid search and reranking — your default pattern?

**A:** BM25/keyword + dense embeddings → reciprocal rank fusion → cross-encoder rerank on top 20 → metadata filters (domain, freshness, ACL). Evaluate with client-provided golden questions; tune weights per domain. Upgrade embedding model only when offline eval proves recall gain.

### Q26. Multi-tenant vs single-tenant RAG for clients?

**A:** **Single-tenant** (or dedicated stack per client) for strict regulatory isolation — common in financial/legal/pharma. **Multi-tenant** with hard logical isolation (separate indexes, encryption keys, ACL namespaces) for cost-sensitive clients with moderate risk. Architecture decision driven by compliance and contract, not convenience.

### Q27. How do you handle conflicting documents in retrieval?

**A:** Canonical source hierarchy defined with client (policy > ADR > wiki draft). Surface conflicts in UI: "Doc A and Doc B disagree." Never silently merge. Store conflict resolutions as metadata overrides for future queries.

### Q28. Chunking strategy for legal/financial/technical corpora?

**A:** Structure-aware chunking on headings/sections; preserve tables/code intact; attach section path metadata; moderate overlap at boundaries; special handling for defined terms and policy sections. Naive fixed-token splits lose semantics quickly in regulated docs.

### Q29. How do you design eval for a client RAG system before launch?

**A:** Co-create golden Q&A set with client SMEs (50–200 questions). Metrics: retrieval recall@k, citation accuracy, unsupported claim rate, abstention correctness, latency. Run eval in CI on prompt/retrieval changes. Pilot with friendly users; capture thumbs feedback. No go-live without signed-off eval baseline.

### Q30. Model routing strategy across clients?

**A:** Tier by task complexity and sensitivity: fast/cheap model for retrieval QA; stronger model for complex reasoning/drafting; private/enterprise endpoint for restricted data. Config-driven router with per-client policy. Track cost per successful task and quality score — optimize routing monthly.

---

## 5. Agentic Architectures & Orchestration

### Q31. When do you recommend agents vs simple RAG Q&A?

**A:** Agents when the task is **multi-step**, requires **live system access**, or combines **retrieve → reason → act** with approvals. Simple RAG when users need read-only answers over documents. I push clients toward read-only MVP first; agents on high-value workflows second — reduces trust and security risk early.

### Q32. Design an agentic workflow architecture for enterprise clients.

**A:** Components:

- **Orchestrator** — plan/execute loop with step limits
- **Tool registry** — typed, permission-scoped APIs
- **Policy engine** — action risk tiers, approval gates
- **State store** — session, plan, evidence, audit trail
- **Human-in-the-loop UI** — previews, approve/reject
- **Eval harness** — scenario tests, red-team prompts

LangChain/LlamaIndex acceptable for prototype; thin custom orchestration for production governance.

### Q33. LangChain vs LlamaIndex vs custom — client recommendation?

**A:** **LlamaIndex** — strong for document-centric RAG prototypes. **LangChain** — broader agent/tool chaining, faster experimentation. **Custom thin layer** — when client needs strict audit, policy hooks, long-term maintainability, or multi-team ownership. I often prototype with LlamaIndex/LangChain, harden critical paths custom — and document that path in the playbook.

### Q34. How do you implement human-in-the-loop at scale?

**A:** Classify actions by risk tier. Read-only auto. Drafts require preview confirm. External writes require explicit approval with diff. SLAs for approvers; escalation paths. Log approvals for eval and compliance. Matches patterns I used for compliant narrative generation at Intapp.

### Q35. How do you secure agent tool access?

**A:** OAuth/service principals with least privilege; per-user delegation where possible; tool wrappers validate inputs against allowlists; no raw SQL/Cypher from model; rate limits; full audit log. Permission checks at retrieval and tool layers — defense in depth.

### Q36. Common agent failure modes in client deployments?

**A:** Infinite loops, wrong tool selection, overconfident synthesis, stale retrieved context, permission leaks, untested write actions. Mitigations: step caps, schema validation, citation requirements, freshness checks, policy engine, shadow mode rollout.

### Q37. How would you productize an "approval-aware agent" module for IG Labs?

**A:** Configurable risk classifier, approval UI component, audit API, webhook integrations (Slack/Teams/email), playbook docs, and reference deployment. Clients configure which tools require approval without rewriting orchestration core.

---

## 6. Cloud Architecture (AWS, Azure, GCP)

### Q38. Cloud-native GenAI platform — high-level architecture on AWS?

**A:** Typical stack:

- **Identity:** Cognito/Okta federation
- **Ingestion:** Lambda/EventBridge or EKS workers; S3 raw; Glue/Step Functions
- **Search:** OpenSearch vector or pgvector on RDS/Aurora
- **Orchestration:** EKS microservices or Lambda + API Gateway
- **LLM:** Bedrock or external API via PrivateLink
- **Observability:** CloudWatch + OpenTelemetry + secure prompt store
- **IaC:** Terraform modules parameterized in playbook

I've operated extensively on AWS (Genapsys, Velti, AutoGrid, Intapp patterns).

### Q39. How do you design for Azure or GCP when client is committed?

**A:** Map equivalent services (Azure: Entra ID, AI Search, Foundry/OpenAI, AKS, Monitor; GCP: IAM, Vertex AI, GKE, BigQuery). Keep application contracts stable; swap infrastructure adapters. Document cloud-specific limits (quota, networking, private endpoints) in engagement ADRs.

### Q40. Network security for enterprise LLM workloads?

**A:** Private endpoints for LLM APIs where available; no training data in prompts logged to vendor without contract; VPC isolation for ingestion workers; secrets in managed vault; egress controls; encryption at rest and in transit; SIEM integration for audit events.

### Q41. Serverless vs Kubernetes for client AI platforms?

**A:** **Serverless** — lower ops burden, good for MVPs and spiky ingestion. **Kubernetes** — better for long-running agents, GPU workloads, complex service mesh, multi-team platform teams. I default K8s for enterprise platform engagements with ongoing iteration; serverless for proof-of-concept or low-scale copilots. At AutoGrid and Genapsys I ran production ML on Kubernetes.

### Q42. How do you estimate cloud cost for a GenAI engagement?

**A:** Model tokens (by query volume/complexity), embedding/index storage, retrieval compute, ingestion frequency, observability storage, redundant environments. Provide low/medium/high scenarios. Include cost controls: caching, model routing, index pruning, batch embedding. Review monthly with client.

### Q43. Disaster recovery and HA for client AI services?

**A:** Define RTO/RPO with client. Multi-AZ deployment minimum; backup indexes and metadata; replay ingestion from raw store; model provider failover plan; runbooks for index corruption and model outage (degraded read-only mode).

---

## 7. Data Platforms & Pipelines

### Q44. Where do Snowflake and Databricks fit in an enterprise AI architecture?

**A:** **Snowflake** — analytical data plane, feature/metadata store, structured enterprise data for retrieval enrichment, governance with role-based access. **Databricks** — large-scale data engineering, feature pipelines, model training/fine-tuning at scale, lakehouse integration. The GenAI serving layer often sits adjacent — retrieval orchestration, vector index, agents — fed by curated data products from Snowflake/Databricks.

### Q45. Your hands-on depth with Snowflake/Databricks — how do you position it?

**A:** Strong on **modern data pipeline architecture** — Spark, Kafka, Airflow, cloud object storage, batch/stream patterns, data quality gates — from Velti (50B records/month), Genapsys, and AutoGrid. I've architected systems where AI platforms consume curated datasets from warehouse/lakehouse layers. For Snowflake/Databricks specifically I design integrations and governance patterns; I partner with data platform specialists for deep platform-specific optimization rather than overclaiming hands-on admin depth.

### Q46. Design a data pipeline feeding a client RAG system.

**A:**

```
Operational systems → CDC/batch extract → Raw zone (S3/ADLS)
                     → Transform (Spark/Databricks) → Curated documents + metadata tables
                     → Document publisher → Chunk/embed → Vector index
                     → Lineage + quality checks + ACL sync
```

Schedule + event triggers; idempotent jobs; monitoring on freshness and row/doc counts.

### Q47. How do you handle PII in data pipelines for AI?

**A:** Classification at ingest; redact/mask before embedding where required; separate indexes for restricted corpora; tokenization or field-level encryption; access policies enforced at retrieval; DLP scans; client legal review for cross-border transfer.

### Q48. Feature store vs document index — division of responsibility?

**A:** **Feature store** — structured ML features for traditional models and ranking signals. **Document/knowledge index** — unstructured/semi-structured text for RAG. Some overlap via metadata filters (client ID, product line). Architecture doc clarifies which queries hit which store.

### Q49. Real-time vs batch ingestion for enterprise copilots?

**A:** **Batch/near-line** default for docs, policies, tickets (minutes–hours latency OK). **Real-time** for high-volatility data (inventory, case status) via event stream + short TTL cache. Client UX sets SLA; architecture matches SLA without overengineering day one.

### Q50. Data quality gates before indexing?

**A:** Schema validation, duplicate detection, language detection, empty doc rejection, broken PDF alerts, ACL completeness check (no doc indexed without owner/permission mapping). Quarantine bad records; dashboard for content ops.

---

## 8. MLOps, Governance, Evals & Deployment

### Q51. What does MLOps mean for LLM applications vs traditional ML?

**A:** Traditional MLOps emphasizes training pipelines and model registry. **LLM application MLOps** emphasizes **prompt/version control, retrieval index lifecycle, eval regression, observability of prompts/responses, guardrails, and rollout controls** — the model weights often change via vendor API, not your training job. Both need monitoring; LLM systems add non-determinism and content-dependent failures.

### Q52. Model governance framework for enterprise clients?

**A:** Model allowlist per data classification; approval for new models; documented eval baseline per model; routing policies; retention rules for prompts/responses; periodic re-certification; incident process for model behavior drift. Align with client AI steering committee structure — similar to AbbVie-style enterprise AI governance patterns.

### Q53. Deployment patterns for GenAI features?

**A:** Feature flags → internal pilot → canary % → full rollout. Shadow mode for agents. Blue/green for orchestration service; index migrations with dual-read validation. Rollback = revert prompt/retrieval version + feature flag — must be tested.

### Q54. What goes in a client eval harness?

**A:** Golden Q&A sets by domain; retrieval metrics; citation/support checks; toxicity/PII leakage tests; permission bypass tests; latency SLO checks; regression on prompt changes in CI; monthly SME review of failed cases.

### Q55. Guardrails beyond prompt instructions?

**A:** Output schema validation; blocklists/allowlists; PII detectors; max action counts for agents; retrieval confidence gates; mandatory citations; human approval tiers; rate limits; semantic cache poisoning protections for retrieved content (sanitize embedded instructions).

### Q56. Observability for client LLM systems?

**A:** OpenTelemetry traces; structured logs with trace/session IDs; secure prompt/response capture; retrieval candidate logging; cost dashboards; quality metrics from feedback; alerts on error rate, empty retrieval rate, latency spikes. Tools: Langfuse, Arize, Helicone, or custom on CloudWatch/Azure Monitor.

### Q57. How do you handle model vendor upgrades?

**A:** Pin versions in config; eval golden set before switch; canary traffic; compare quality/cost/latency; document rollback. Treat vendor model deprecation like dependency upgrades — planned, measured, reversible.

### Q58. AI lifecycle from POC to production — your staged model?

**A:**

1. **POC** — prove workflow value with manual/offline eval
2. **Pilot** — production infra, ACL, logging, limited users
3. **Production** — SLOs, on-call runbooks, eval CI, governance sign-off
4. **Scale** — cost optimization, playbook hardening, multi-team rollout

Explicit exit criteria between stages; no "POC in prod" without logging and kill switch.

---

## 9. Domain & Regulated Enterprise Patterns

### Q59. Experience architecting AI for regulated industries?

**A:** At Intapp I architected GenAI for **legal, CRM, and private-capital / financial-services** workflows — permission-aware retrieval, compliant narrative generation, validation controls, auditability, human review, and schema grounding in customized enterprise data models. These patterns transfer to healthcare, pharma, and insurance clients with similar trust and compliance requirements.

### Q60. Architecture differences for legal/financial vs generic enterprise copilots?

**A:** Higher bar on **source traceability, permission enforcement, abstention, audit logs, and human approval**; customized schemas vs generic document search; workflow integration not standalone chat; eval sets co-owned with compliance SMEs.

### Q61. How do you architect for "AI-ready" client data platforms?

**A:** Layer on top of existing MDM/warehouse: document registry with metadata, schema catalog, lineage, ACL sync, chunk/embedding pipeline, quality scores, freshness SLAs. Don't require rip-and-replace — incremental maturity model in playbook.

### Q62. Permission-aware retrieval architecture?

**A:** Index-time: tag docs with ACL metadata. Query-time: filter by user/group/service account scopes before search and rerank. Tool calls re-check authorization. Integration with IdP groups and client data platform RBAC. Test with negative cases (user must NOT see restricted doc).

---

## 10. Principal Tradeoffs & Engagement Delivery

### Q63. 12-week client engagement — how do you phase delivery?

**A:**

- **Weeks 1–2:** Discovery, readiness assessment, target architecture, eval plan
- **Weeks 3–6:** MVP platform (ingestion subset, RAG Q&A, auth, logging)
- **Weeks 7–9:** Pilot with users, eval tuning, governance hardening
- **Weeks 10–12:** Production cutover or handoff, playbook extraction, training

Adjust if data access delays — never skip governance for speed.

### Q64. Client wants autonomous agents in week 4 — your response?

**A:** Acknowledge goal; propose risk-tiered roadmap. Week 4 deliver read-only copilot + architecture for agents; agents on one approved workflow after eval and approval UX exist. Explain trust-building sequence — executives usually accept when framed as risk management.

### Q65. Build vs buy for client engagements?

**A:** Buy commodity (LLM APIs, managed vector, observability SaaS). Build differentiation (context layer, domain tools, policy, eval, integration with client systems). Playbook accelerates build by starting from reference modules — client pays for configuration and domain adapters, not greenfield platform.

### Q66. How do you define success for an IG Labs architecture engagement?

**A:** Client KPI movement (time saved, deflection, quality metric); production system with governance; client team can operate it; reusable playbook components extracted; eval baseline maintained post-handoff.

### Q67. Handoff to client engineering team?

**A:** Architecture docs, ADRs, runbooks, IaC repos, eval suites, onboarding workshop, 2–4 week hypercare. Explicit ownership matrix (client vs IG Labs). Playbook version pinned in handoff package.

### Q68. When do you escalate vs adapt architecture mid-engagement?

**A:** Escalate when security/compliance boundary violated, data access fundamentally blocked, or scope creep threatens MVP. Adapt when learning shows different integration path still meets outcomes — document ADR for change.

---

## 11. System Design & Live Exercise Prep

### Q69. System design: enterprise GenAI platform for a financial services client.

**A:** See Q22 + Q32 + Q38. Emphasize: single-tenant or hard isolation, permission-aware RAG, audit, eval harness, model routing to enterprise endpoint, phased agent rollout, integration with CRM/document management, CST-friendly support model.

### Q70. Whiteboard: reference architecture for reusable IG Labs RAG module.

**A:** Draw ingestion adapter interface → normalized document model → embed/index service → retrieval API (hybrid + rerank + ACL) → orchestration API → client UI adapter. Sidecar: eval service, observability, config/version management. Parameterize: source connectors, schema mappers, policy rules.

### Q71. Client has Snowflake + Salesforce + SharePoint — design context layer.

**A:** Connectors for each; unified metadata catalog in Postgres; Salesforce structured data enriches retrieval filters; SharePoint docs chunked to vector index; Snowflake curated tables for factual lookups via tools; SSO unified; eval questions spanning all three sources.

### Q72. Debug: client says copilot answers are wrong after go-live.

**A:** Replay traces; check index freshness and ACL filters; compare pre/post model or prompt change; inspect retrieval candidates; verify conflicting docs; run golden eval diff; check if users query data not yet ingested. Fix retrieval/governance before swapping models.

### Q73. Estimate team size for MVP enterprise copilot?

**A:** Minimum effective: architect (me) + 2 engineers (backend/data + integration) + client SME access — 8–12 weeks. Add frontend specialist for polished UX; add data platform engineer for heavy Snowflake/Databricks integration.

---

## Quick Reference — Top 10 to Rehearse Aloud

1. **Why IG Labs / this role** — Q1
2. **Client discovery approach** — Q6
3. **Consulting + product thinking / reusable IP** — Q4, Q14
4. **Context engineering definition** — Q21
5. **Production RAG architecture** — Q22
6. **RAG vs fine-tuning vs agents** — Q23
7. **Reference architecture / playbook** — Q13, Q18
8. **Agentic architecture with governance** — Q32
9. **Regulated enterprise patterns** — Q59
10. **12-week engagement phasing** — Q63

---

## Top 10 — 30-Second Spoken Versions

### 1. Why IG Labs / this role (Q1)

"This role is where I've been heading — client-facing enterprise AI architecture plus reusable patterns that scale. At Intapp I worked forward-deployed with legal and financial stakeholders, shipping production GenGen systems. IG Labs adds consulting leverage: solve real problems with CTO-level clients, then codify what worked into playbooks for the next engagement. That's architecture with multiplier effect, not one-off slide decks."

### 2. Client discovery approach (Q6)

"I run discovery in four passes: business outcomes, current state, risk and trust requirements, then target architecture with explicit non-goals. Within the first week I deliver a readout — problem framing, options, recommendation, open questions. I don't jump to RAG until I understand whether the real blocker is data access, workflow integration, or governance. That saves clients from expensive prototypes that don't ship."

### 3. Consulting + reusable IP (Q4, Q14)

"Every engagement should produce two outputs: the client solution and a reusable asset — a reference architecture, eval template, or connector module. During delivery I tag decisions as client-specific versus generalizable. After MVP we extract templated pieces into the IG Labs library with config hooks and anonymized examples. Consulting without IP capture doesn't scale a practice."

### 4. Context engineering (Q21)

"Context engineering is converting fragmented enterprise data — documents, schemas, APIs, permissions — into bounded, trustworthy context for LLM systems. It's the highest-leverage work in enterprise GenAI. Clients often overfocus on model selection; I make context layer design explicit in every architecture because that's what determines whether answers are grounded, safe, and useful."

### 5. Production RAG architecture (Q22)

"I design RAG as ingestion into normalized docs and metadata, permission-filtered hybrid retrieval with rerank, context packing, generation with citation enforcement, and output validation — plus eval and observability parallel to serving. Permission-aware retrieval and abstention on low confidence are non-negotiable for regulated clients. I've shipped this pattern at Intapp for legal and CRM workflows at scale."

### 6. RAG vs fine-tuning vs agents (Q23)

"Default is RAG for changing knowledge with citations. Add tools and agents when tasks need live data or multi-step actions with approvals. Fine-tune only when eval proves prompting can't stabilize format or classification. I recommend clients start read-only, prove trust and eval discipline, then add agents on high-value workflows — reduces security and reputational risk."

### 7. Reference architecture / playbook (Q13, Q18)

"A reference architecture is a validated pattern bundle — diagrams, component contracts, security model, deployment topology, eval checklist — parameterized for client config. An IG Labs Enterprise GenAI Starter would include discovery scorecard, context layer guide, RAG reference impl, agent approval pattern, eval harness, governance checklist, and cloud deployment templates. Engineers implement repeatedly; clients pay for adapters, not reinvention."

### 8. Agentic architecture with governance (Q32)

"Agents need orchestration with step limits, a typed permission-scoped tool registry, policy engine for risk tiers, audit trail, and human-in-the-loop approval UI. I prototype with LlamaIndex or LangChain but harden production paths in a thin custom layer for audit and policy. Same pattern I used for compliant narrative workflows at Intapp — automation with review, not autonomous writes on day one."

### 9. Regulated enterprise patterns (Q59)

"At Intapp I architected GenAI for legal and financial workflows — permission-aware retrieval, schema grounding, validation controls, auditability, and human review. Financial and legal clients need source traceability and abstention, not clever chat. Those patterns transfer directly to healthcare and pharma clients with similar trust requirements, and they're central to how I'd design IG Labs playbooks."

### 10. 12-week engagement phasing (Q63)

"Weeks one to two: discovery, readiness, target architecture, eval plan. Weeks three to six: MVP — ingestion subset, RAG Q&A, auth, logging. Weeks seven to nine: pilot, eval tuning, governance hardening. Weeks ten to twelve: production handoff and playbook extraction. I won't skip governance for speed, and I won't promise autonomous agents before approval workflows and eval baselines exist."

---

## Honest Positioning Notes

**Lean in:** enterprise context engineering, client-facing architecture, reference architectures, RAG/agent patterns with governance, Intapp regulated-workflow experience, AWS/K8s/Spark/Kafka production background, forward-deployed stakeholder delivery, eval and guardrails discipline.

**Address directly if asked:** pre-sales — frame as architecture discovery and scoping, not sales closing; Snowflake/Databricks — strong pipeline/lakehouse architecture and integration design, partner for deep platform admin; CST hours — comfortable with early PT blocks for client collaboration.

**Intapp proof points to repeat:** NL CRM interface at 10,000+ customers; schema grounding; permission-aware context; compliant narrative generation; reusable production patterns for LLM integration and governance.

**Resume to lead with:** `Hossein Akhlaghpour Resume 2026 - AI Architect Enterprise GenAI.md`
