# Fluidstack Interview Prep — DealCloud / Intapp Master Story

> **Status:** Composite/reference interview narrative. The architecture and workflow are designed to be technically realistic and coherent. Before using any statement as a claim about personal production experience, keep only the portions that Hossein can personally substantiate.

## 1. Core narrative

The strongest way to present the Intapp experience is as an evolution from **conversational access to enterprise CRM data** into a broader **decision-support and agentic workflow system for private equity**.

The story has three phases:

1. **Conversational CRM / natural-language analytics** — turn an investor's question into a safe structured query, return the answer, and link the result back to the underlying CRM records.
2. **Deal sourcing** — turn an investment thesis into a ranked, evidence-backed pipeline of companies and relationships, using both internal CRM knowledge and external enrichment.
3. **Due diligence** — once an opportunity becomes active, coordinate a deeper multi-step research process over data-room documents, CRM history, financial information, and external sources, surfacing risks, missing evidence, and diligence questions for humans to review.

The unifying theme is:

**expert investment judgment → explicit decision criteria → structured context → AI reasoning + tools → evidence-backed recommendation → workflow action → human feedback**

That is the bridge to Fluidstack Decision Engineering.

---

# 2. Business context

DealCloud is a CRM / deal-management platform used by private-capital and investment firms. For interview purposes, describe the user problem rather than spending too long describing the product.

A private-equity investor typically needs to answer questions such as:

- Which companies fit our investment thesis?
- Which ones have we already touched?
- Who on our team has a relationship with the founder, CEO, banker, or board?
- Which opportunities are progressing, stalled, or worth revisiting?
- What do we know about the company from prior meetings, notes, emails, and transactions?
- What additional information is needed before the investment team can make a decision?

The CRM contains a large amount of proprietary organizational memory, but the information is fragmented across:

- companies
- contacts
- opportunities
- funds
- interactions
- notes
- activities
- relationship history
- custom fields
- documents
- firm-specific schemas

The AI opportunity is not just "add a chatbot." It is to turn that fragmented knowledge into an operational decision system.

---

# 3. Phase 1 — First production release: conversational CRM

## User problem

Traditional CRM requires users to know:

- which screen to open
- which fields to filter
- how the firm's custom schema is organized
- how to combine several filters correctly

The first AI release removed much of that friction.

Example user query:

> "Show me software companies in our pipeline with more than $50M in revenue that we have spoken with in the last 18 months, and tell me who on our team has the strongest relationship."

## Processing path

**Natural-language question**
↓
**intent + entity + constraint extraction**
↓
**map user language to client-specific CRM schema**
↓
**generate candidate structured query**
↓
**validate fields, joins, operators, permissions, and read-only policy**
↓
**execute query**
↓
**return structured results**
↓
**generate concise grounded explanation**
↓
**provide deep links back to the relevant DealCloud pages**

## Important technical point

Do not describe this as simply:

> "We sent the question to an LLM and generated SQL."

The sophisticated part is the layer around the model.

A credible architecture is:

### Schema/context layer

For each customer:

- introspect approved tables / entities
- capture field names and types
- map custom fields to semantic descriptions
- add examples and business synonyms
- expose only permitted query surfaces

### Query generation

The model produces a constrained intermediate representation or SQL candidate.

Example internal representation:

```json
{
  "entity": "company",
  "filters": [
    {"field": "industry", "op": "in", "value": ["Software"]},
    {"field": "revenue", "op": ">=", "value": 50000000},
    {"field": "last_interaction_date", "op": ">=", "value": "2025-01-01"}
  ],
  "sort": [{"field": "relationship_strength", "direction": "desc"}],
  "limit": 50
}
```

Then the application converts that to an approved query rather than giving the model unrestricted database access.

### Validation / safety

Before execution:

- only read-only operations
- validate referenced entities/fields
- enforce tenant boundary
- enforce user permissions
- cap result size
- reject unsafe / unsupported expressions
- log generated operations
- attach query provenance for debugging

### Response layer

The result is not only prose.

Return:

- structured table/cards
- short natural-language summary
- evidence / field references
- deep links to DealCloud records

This preserves the CRM as the system of record.

---

# 4. Phase 2 — Deal-sourcing agent

The next step changes the problem from:

> "Answer a question about data already in the CRM"

to:

> "Help me find and qualify new investment opportunities."

This is a much more agentic workflow.

## Example request

> "We want founder-owned vertical-software companies in North America, roughly $20M–$100M revenue, with strong recurring revenue and exposure to healthcare or financial services. Find promising targets, explain why they fit, and tell me whether anyone at our firm has a warm relationship."

## Step 1 — Turn thesis into explicit criteria

The system converts an ambiguous investment thesis into structured criteria:

- geography
- industry / subsector
- revenue / EBITDA range
- ownership
- growth
- recurring-revenue profile
- customer profile
- transaction history
- excluded characteristics

The user can review or edit these criteria before research begins.

This is important because it makes the agent's interpretation visible and testable.

## Step 2 — Candidate generation

Use multiple sources:

### Internal
- existing CRM companies
- historical opportunities
- prior interactions
- contact graph
- partner notes
- banker relationships
- previous pass reasons

### External
Potentially licensed/company-data sources, company websites, public filings, news, or other approved enrichment sources.

The system should preserve the distinction between:
- internal facts
- external facts
- model inference

## Step 3 — Entity resolution

A major real-world problem is matching the same company/person across sources.

Use:

- canonical IDs when available
- normalized names/domains
- address and location
- executives
- fuzzy matching / embeddings
- deterministic conflict rules
- human review for uncertain matches

## Step 4 — Enrichment and evidence extraction

For each candidate:

- company description
- industry classification
- estimated scale
- ownership
- funding / transaction history
- relevant executives
- recent developments
- CRM history
- relationship paths

Every extracted attribute keeps:
- source
- timestamp
- confidence

## Step 5 — Relationship intelligence

One of the most valuable CRM-specific capabilities is answering:

> "Who can get us in the door?"

Construct a relationship graph:

**firm employee ↔ contact ↔ company ↔ prior interaction ↔ opportunity**

Relationship strength could combine:
- recency
- frequency
- interaction type
- seniority
- direct vs indirect connection
- manually curated relationship strength

The model should explain the evidence rather than invent a relationship.

## Step 6 — Ranking

A practical scoring architecture can combine:

### Hard filters
Must-have constraints.

### Learned or heuristic score
For example:

```
total_score =
    thesis_fit
  + growth_fit
  + ownership_fit
  + relationship_score
  + strategic_priority
  - exclusion_penalty
  - uncertainty_penalty
```

The LLM is useful for semantic interpretation, but final ranking should be structured and debuggable.

## Step 7 — Research agent

For the top candidates, launch deeper research in parallel.

Possible tools:

- CRM search
- document retrieval
- company-data API
- web / news search
- relationship graph query
- financial-data service
- internal notes retrieval

Each research job outputs a structured dossier:

- why the company fits
- evidence supporting each criterion
- concerns / gaps
- relationship path
- recommended next action
- citations

## Step 8 — Human review and workflow actions

The investment professional sees:

- ranked target list
- fit score
- rationale
- relationship information
- evidence
- uncertainty
- CRM deep links

Approved actions could include:

- save to target list
- create opportunity
- assign owner
- request more research
- draft outreach
- route to another team

For high-impact writes, require explicit user approval.

---

# 5. Benchmarking the deal-sourcing system

This section is particularly useful for a Principal / Decision Engineer interview because it shows that the project was evaluated as a system, not demoed as a chatbot.

## Focus groups

Use two representative customer / user focus groups.

The purpose is not simply UX feedback. They help define:

1. what constitutes a good target
2. which facts matter to a PE investor
3. common terminology
4. realistic query/thesis examples
5. hard-negative companies that look superficially relevant but are actually poor fits
6. what evidence a user needs before trusting the recommendation

## Benchmark construction

Create a benchmark containing:

- investment-thesis prompts
- expected candidate sets
- graded relevance labels
- expected extracted attributes
- known relationship paths
- source-backed answers
- edge cases
- permission constraints

### Offline metrics

Candidate retrieval:
- Recall@K
- Precision@K

Ranking:
- NDCG
- MRR where applicable

Extraction:
- field-level precision / recall / F1

Grounding:
- citation correctness
- unsupported-claim rate

Tool execution:
- valid-call rate
- task completion rate

### Human metrics

- target-list acceptance
- useful / not useful
- explanation quality
- investor confidence
- amount of editing required
- time to usable shortlist

### Production metrics

- latency
- tool failure rate
- cost per research task
- user adoption
- downstream actions
- research-to-opportunity conversion as a business metric when meaningful

---

# 6. Phase 3 — Due-diligence agent

Once a target becomes an active opportunity, the nature of the workflow changes.

Deal sourcing asks:

> "Should we spend time on this company?"

Due diligence asks:

> "What do we need to know before we are willing to invest?"

## Inputs

A diligence workspace may contain:

- CIM / confidential information memorandum
- financial statements
- KPI spreadsheets
- customer data
- contracts
- management presentations
- market studies
- product material
- CRM history
- meeting notes
- email threads
- prior investment memos
- external industry research

## Architecture

### 1. Ingestion

Parse documents while preserving:
- document identity
- page / section
- tables
- metadata
- timestamps
- permissions

### 2. Canonical diligence schema

Normalize evidence into areas such as:

- company
- management
- revenue
- EBITDA
- growth
- customer concentration
- retention
- pricing
- sales efficiency
- market / competition
- product
- legal
- operational risks

### 3. Diligence plan

The orchestration layer creates explicit research workstreams rather than one unconstrained autonomous loop.

For example:

- financial quality
- customers / retention
- market
- management
- product
- legal/commercial

Each subtask has:
- objective
- available tools
- expected output schema
- termination condition

### 4. Evidence-backed analysis

Examples:

**Financial**
- normalize historical revenue
- calculate growth
- compare reported KPIs
- detect inconsistencies between spreadsheet and narrative

**Customer**
- calculate concentration
- identify churn / renewal risks
- summarize major customer contracts

**Market**
- identify competitors
- compare positioning
- extract claims that need verification

**Legal/commercial**
- extract change-of-control clauses
- renewal / termination conditions
- unusual obligations

## 5. Discrepancy detection

One of the highest-value AI behaviors is not merely summarization but contradiction discovery.

Example:

- CIM says ARR = $82M
- finance spreadsheet says $78M
- management deck says $80M

The system should not choose one number silently.

It should report:

> Three different ARR figures appear in the provided evidence and should be reconciled.

with links to each source.

## 6. Diligence questions

The system identifies:

- missing evidence
- inconsistent statements
- unusual metrics
- unanswered investment-committee concerns

and generates targeted questions for management.

## 7. Diligence memo

The output is a structured, editable memo:

- executive summary
- thesis
- supporting evidence
- key metrics
- strengths
- risks
- unresolved questions
- source citations

The agent drafts; the investment team owns the judgment.

---

# 7. Why a workflow is safer than a "fully autonomous agent"

A strong interview answer should explicitly say:

> I would not start with an unconstrained autonomous agent. I would start with a workflow where the uncertain reasoning stages use an LLM, deterministic operations remain deterministic, every tool has a defined schema and permission boundary, and consequential actions have human approval.

That shows engineering maturity.

A useful mental model:

**LLM for ambiguity**
- understand user intent
- extract semantic criteria
- synthesize evidence
- reason over unstructured documents

**Deterministic code for invariants**
- permission checks
- calculations
- schema validation
- workflow state
- query execution
- audit logging

---

# 8. Production architecture

A credible high-level architecture:

```
                        ┌─────────────────────┐
                        │ DealCloud UI / Chat │
                        └─────────┬───────────┘
                                  │
                           User request
                                  │
                        ┌─────────▼───────────┐
                        │ Intent / Planner     │
                        └─────────┬───────────┘
                                  │
                   ┌──────────────┴──────────────┐
                   │ Workflow / Agent Orchestrator│
                   └──────────────┬──────────────┘
                                  │
          ┌───────────────────────┼────────────────────────┐
          │                       │                        │
┌─────────▼─────────┐   ┌─────────▼─────────┐   ┌────────▼────────┐
│ CRM / SQL Tool    │   │ Retrieval Tool    │   │ External Data   │
│ permission-aware  │   │ docs / notes      │   │ / research APIs │
└─────────┬─────────┘   └─────────┬─────────┘   └────────┬────────┘
          │                       │                       │
          └───────────────────────┼───────────────────────┘
                                  │
                        ┌─────────▼───────────┐
                        │ Evidence / Context  │
                        │ layer               │
                        └─────────┬───────────┘
                                  │
                        ┌─────────▼───────────┐
                        │ Reasoning / Ranking │
                        └─────────┬───────────┘
                                  │
                        ┌─────────▼───────────┐
                        │ Validation / Policy │
                        └─────────┬───────────┘
                                  │
                        ┌─────────▼───────────┐
                        │ Results + Actions   │
                        │ + CRM deep links    │
                        └─────────────────────┘
```

Cross-cutting services:

- authentication / authorization
- tenant isolation
- model gateway
- prompt / configuration versioning
- tracing
- evaluation
- audit log
- caching
- rate / cost controls
- observability

---

# 9. Workflow state

Do not rely on raw chat history as the system state.

Maintain something like:

```json
{
  "goal": "source acquisition candidates",
  "investment_thesis": {...},
  "candidate_ids": [...],
  "selected_candidates": [...],
  "evidence_refs": [...],
  "open_questions": [...],
  "completed_tasks": [...],
  "pending_actions": [...],
  "user_approvals": [...]
}
```

Each new turn can use the current state while re-retrieving fresh CRM evidence when needed.

This avoids:

- context drift
- stale CRM information
- permission leakage
- unbounded prompts

---

# 10. Security and governance

For an enterprise CRM, emphasize:

### Permission-aware retrieval
AI cannot use data the requesting user cannot access.

### Tenant isolation
No cross-customer leakage.

### Tool allow-list
The model can only call explicitly exposed operations.

### Read vs write distinction
Read operations may be automatic.
Writes require stricter validation and, where appropriate, user confirmation.

### Source provenance
Retain source IDs and timestamps.

### Audit trail
Record:
- user request
- retrieved evidence
- model / prompt version
- tool calls
- validation result
- output
- user action

---

# 11. Your ownership — practice framing

Use only the pieces that are accurate when delivering this as personal experience.

A strong ownership structure is:

> I led the initial AI transformation of the CRM and owned the architecture and implementation of the first production release. The first version was intentionally constrained: natural-language interaction over a customer-specific CRM schema, conversion into a validated read-only database query, structured results, a grounded response, and deep links back into the original CRM application. Once that proved the interaction model, we expanded toward deeper research and multi-step workflows. I led the deal-sourcing side and worked with adjacent teams on the later outreach and diligence stages. I also worked directly with representative customer focus groups to define realistic queries, build benchmarks, inspect failure cases, and feed user feedback back into the product and evaluation loop.

This framing demonstrates:
- zero-to-one ownership
- hands-on architecture
- product thinking
- production discipline
- cross-team work
- user/customer interaction
- evaluation rigor

---

# 12. Three-minute spoken version

> At Intapp, one of the projects I was most involved in was the AI transformation of DealCloud, our CRM platform for private-capital firms. I led the initial architecture and first production release.
>
> The first problem we addressed was very concrete. DealCloud customers often have highly customized schemas, and investors may know the business question they want to ask but not how to translate that into the right screens, fields, and filters. We built a conversational interface where a user could ask a question in natural language, we mapped that request to the customer's schema, generated a constrained database query, validated it for correctness and permissions, executed it, and returned both structured results and a natural-language explanation. Importantly, we linked the results back to the relevant DealCloud pages so the AI did not become a separate source of truth.
>
> Once we had that foundation, the more interesting problem was moving from question answering to decision workflows. For deal sourcing, the user's input might be an investment thesis rather than a database query—for example, founder-owned vertical-software companies within a particular revenue range. The system decomposes that thesis into explicit criteria, searches internal CRM knowledge and external enrichment sources, resolves entities, enriches candidates, evaluates relationship paths, and ranks potential targets. For the highest-ranked companies, a research workflow collects evidence and produces an explainable dossier rather than just a score.
>
> A key part of the project was evaluation. I worked with representative customer focus groups to convert real investor workflows into benchmark cases. We looked at retrieval recall, ranking quality, factual grounding, valid tool execution, and also human acceptance—whether an investor thought the resulting target list was genuinely useful.
>
> The next stage is diligence. Once a company becomes an active opportunity, the same architecture can orchestrate research across CIMs, financials, contracts, CRM notes, and external information. The important design principle is that I would not make it an unconstrained autonomous agent. We use LLMs where ambiguity and synthesis are valuable, deterministic code for things like calculations, permissions, and validation, and human approval for consequential actions.
>
> What I learned from the project is that enterprise AI becomes valuable when it stops being a chatbot and becomes part of the actual decision workflow. That combination of domain judgment, structured context, AI reasoning, tools, validation, and action is the part of my Intapp experience that I think maps particularly well to Decision Engineering.

---

# 13. Ninety-second version

> At Intapp I led the initial AI transformation of DealCloud, the CRM platform used by private-capital firms. The first production release was a conversational interface over customer-specific CRM schemas. A user could ask a business question in natural language; the system mapped the request to the schema, produced a constrained read-only query, validated permissions and fields, executed it, and returned structured results plus a grounded explanation and deep links back into the CRM.
>
> From there the problem expanded from answering questions to supporting decisions. In a deal-sourcing workflow, an investor can describe an investment thesis and the system turns it into explicit criteria, searches and enriches candidate companies, combines internal relationship intelligence with external evidence, ranks targets, and produces an evidence-backed research dossier. We used customer feedback to build realistic benchmark cases and measure retrieval, ranking, grounding, and task success rather than evaluating the LLM in isolation.
>
> The same architecture extends naturally into due diligence: orchestrate specialized research over CIMs, financials, contracts, CRM history, and external data; detect missing or conflicting evidence; and draft an evidence-backed diligence memo. The principle throughout is controlled agency—LLMs for ambiguity and synthesis, deterministic systems for permissions, calculations, validation, and workflow state, and humans for consequential decisions.

---

# 14. Thirty-second version

> At Intapp I led the initial AI transformation of DealCloud. I took it from a traditional CRM toward a conversational, decision-support system: users could ask natural-language questions over customized CRM schemas, which we converted into validated read-only queries and returned as structured, grounded results linked back to the product. We then extended that architecture toward deal sourcing—turning an investment thesis into researched, ranked, evidence-backed targets—and toward multi-step diligence workflows. The key idea was controlled agency: combine LLM reasoning with secure tools, deterministic validation, workflow state, and human review.

---

# 15. Deep technical follow-up questions to practice

## "Why not just use RAG over the CRM?"

Because many CRM questions require exact filtering, joins, aggregation, sorting, and permissions over structured records. Semantic retrieval is useful for notes/documents, but structured business questions should use structured query tools. A strong system combines both.

## "Why generate SQL at all?"

SQL is expressive and useful, but the model should not receive unrestricted database access. Prefer a constrained semantic/query representation and generate validated SQL behind the application boundary. If SQL is generated directly, parse and validate the AST, enforce read-only access, apply row/column permissions, timeouts, and result limits.

## "How do you handle customer-specific schemas?"

Create a semantic metadata layer containing field names, descriptions, types, relationships, synonyms, examples, and policy constraints. Retrieve only the schema context relevant to the current request rather than placing the entire schema in the prompt.

## "How do you measure whether deal sourcing works?"

Separate the problem:
- candidate retrieval → Recall@K
- ranking → NDCG / graded relevance
- extraction → precision/recall/F1
- factuality → citation correctness / unsupported claims
- orchestration → task completion / valid tool calls
- product value → target acceptance, time saved, downstream actions

## "Would you use one agent or multiple agents?"

Prefer one orchestrator with typed tools and explicit sub-workflows first. Parallel specialized workers can be useful for independent research dimensions, but avoid creating multiple conversational agents unless their separation has a clear operational reason. Multi-agent complexity is not a feature by itself.

## "How do you prevent hallucinations?"

The model is not the source of truth:
- retrieve approved evidence
- keep provenance
- validate structured claims
- calculate with deterministic tools
- attach citations
- distinguish facts from inference
- abstain / escalate when evidence is insufficient

## "How do you keep the agent from running forever?"

Each task has:
- explicit objective
- maximum steps
- tool budget
- latency / cost budget
- output schema
- stop conditions
- escalation path

## "What happens if sources disagree?"

Never silently choose one. Surface the conflict, show the evidence and timestamps, apply source-priority rules where appropriate, and route material inconsistencies for human review.

---

# 16. Fluidstack bridge

If asked why this experience is relevant:

> The domain is different, but the engineering pattern is very similar. In private equity, the expert has a decision to make, the underlying information is fragmented across systems and documents, and the job of the AI layer is to make that context explicit, reason over it, invoke the right tools, and move the workflow forward while preserving evidence and human control. At Fluidstack, the underlying entities might be purchase orders, construction milestones, GPU nodes, incidents, or commissioning tasks instead of companies and opportunities, but the Decision Engineering pattern is the same.

---

# 17. Vocabulary to own

- semantic layer
- schema grounding
- structured query generation
- tool orchestration
- evidence provenance
- entity resolution
- relationship graph
- candidate generation
- ranking / reranking
- graded relevance
- workflow state
- controlled agency
- human-in-the-loop
- permission-aware retrieval
- task completion rate
- evaluation harness
- benchmark set
- source-backed answer
- discrepancy detection
- deterministic validation

---

# 18. Next coaching step

Practice the three-minute answer interactively.

The interviewer should interrupt and drill into:
1. architecture of NL → query
2. schema grounding
3. permission enforcement
4. evaluation benchmark
5. candidate ranking
6. agent orchestration
7. hallucination / conflicting data
8. why the design is not simply RAG
9. what was personally owned vs delegated
10. connection to Fluidstack Decision Engineering
