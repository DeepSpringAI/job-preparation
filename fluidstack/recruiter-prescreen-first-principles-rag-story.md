# Recruiter Prescreen — First-Principles / Challenged Assumption Story

## Core story

When the DealCloud conversational AI project started, an early idea was to create a separate RAG index over each customer's CRM records so the assistant could map user questions to relevant data.

Hossein challenged that approach because it optimized for a quick single-customer prototype but did not scale operationally or security-wise across 10,000+ customers with isolated data and custom schemas.

## Assumption challenged

**Assumption:** The fastest way to add AI to each client is to replicate/index that client's CRM data into a dedicated RAG system.

## First-principles question

What does the AI actually need in order to answer the user's question?

It does not necessarily need a copied semantic index of all customer records.

It needs:
- understanding of the customer's schema / metadata
- the ability to map user language to that schema
- a permission-aware tool/API for querying the system of record
- enough returned evidence to answer the request

## Proposed architecture

Instead of indexing customer records into separate RAG stores:

1. Leave record-level customer data inside the existing DealCloud security / ACL boundary.
2. Build semantic retrieval over schema metadata, such as:
   - entities
   - fields
   - descriptions
   - relationships
   - business synonyms
3. Use that metadata to map the user's natural-language request to the customer's schema.
4. Generate a constrained query/API request.
5. Execute that request through the existing CRM permission model.
6. Return only the data the authenticated user is allowed to access.

## Why this is stronger

### Scalability
Avoid maintaining a separate record-level vector store and synchronization pipeline for each customer.

### Security
Customer data remains behind the existing tenant / permission boundary.

### Freshness
Queries use the system of record directly rather than a secondary copied index that can become stale.

### Governance
Existing ACLs and audit controls remain authoritative.

### Generalization
The same AI/query architecture can adapt dynamically to each customer's schema.

## Recruiter-ready answer

"When we started the conversational AI work for DealCloud, one of the early approaches was to create a separate RAG system over each customer's CRM records.

That was an attractive prototype because it could get one customer working quickly, but I disagreed with the architecture. We had more than 10,000 customers, each with isolated data, custom schemas, and its own permissions. Replicating and maintaining a separate semantic index of customer records would create a major synchronization, security, and operational problem.

So I went back to the underlying requirement: what does the AI actually need? It doesn't need its own copy of the customer's data. It needs to understand that customer's schema and have a safe way to query the existing system of record.

I proposed that we keep the actual records behind DealCloud's existing security and ACL layer and build the semantic retrieval layer primarily around schema metadata—entities, fields, relationships, and business terminology. The model could use that metadata to understand the user's question, map it to the customer's schema, and generate a constrained query or API call. The CRM itself would then execute that request under the user's existing permissions.

That gave us several advantages at once: we didn't need to maintain a separate copy of every customer's records, the data stayed fresh because we queried the source system directly, and we reused the security model that had already been built and validated.

The important part for me was challenging the assumption that because RAG worked for one customer, we should scale that same architecture to every customer. I tried to reason from what information the AI actually needed and keep the system of record responsible for the things it already did well—data ownership, permissions, and query execution."

## Coaching notes

- Say "private-capital customers/firms," not "10,000 private equities."
- Avoid saying "one agent can query across all clients" because that can sound like cross-tenant access. Better: "the same architecture can operate across customers while each request remains tenant- and permission-scoped."
- Do not say all record-level RAG is inherently wrong. The point is that record replication was the wrong default architecture for this structured, permission-sensitive use case.
- If asked technically, distinguish:
  - schema/metadata retrieval
  - structured query/API execution
  - document/note RAG for unstructured content

## Core message

**I rejected a prototype architecture that duplicated customer data and redesigned the system around metadata grounding plus permission-aware querying of the existing system of record.**
