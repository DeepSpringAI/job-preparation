# Special Technical QA: Hallucination Prevention for API and Dataflow Claims

## Question

How would you prevent hallucinations when an AI assistant generates API behavior, integration logic, or dataflow explanations?

## Short interview answer

> I would not allow the model to invent API contracts, fields, endpoints, events, or dataflows from memory. For any generated API or dataflow claim, I would validate it against source-of-truth artifacts such as OpenAPI specs, approved architecture documents, ADRs, HLSDs, schema registries, protobuf definitions, database schemas, and runbooks. The assistant can draft an answer, but before presenting it as factual, the system should check whether the named endpoints, methods, request fields, response fields, event names, transformations, and downstream dependencies actually exist in the approved specs. Unsupported claims should be removed, marked as assumptions, or routed to clarification. This gives the user a useful conversational experience while keeping the output grounded, traceable, and safe for enterprise use.

---

## What this really means

Hallucination prevention is not only about saying "use RAG." In enterprise systems, the model often needs to talk about concrete technical objects:

- API endpoints
- HTTP methods
- request and response schemas
- required fields
- status codes
- authentication requirements
- event names
- Kafka topics
- database tables
- data transformations
- upstream and downstream systems
- ownership and operational runbooks

A model may produce something that sounds technically reasonable but is wrong.

Example hallucinated claim:

```text
The assistant says the CRM API supports PATCH /accounts/{id}/contacts.
```

But the OpenAPI spec may only define:

```text
PUT /accounts/{accountId}/relationships/contact
```

That difference matters. If the answer is used by an engineer, integration agent, or automation workflow, the hallucination can lead to broken code, failed workflows, or incorrect architecture decisions.

---

## Core principle

The model should not be treated as the source of truth for system contracts.

For API and dataflow questions, the source of truth should be:

- OpenAPI / Swagger specs
- AsyncAPI specs
- protobuf or Avro schemas
- approved HLSDs and LLDs
- ADRs
- database schemas
- schema registry
- source-code-owned interface definitions
- workflow definitions
- runbooks
- production metadata catalogs

The assistant can reason over these artifacts, summarize them, and explain them, but factual claims should be grounded in them.

---

## Validation pattern

A strong production pattern is:

```text
User question
   |
   v
Retrieve relevant specs and approved docs
   |
   v
Generate draft answer or plan
   |
   v
Extract factual claims from the draft
   |
   v
Validate claims against source-of-truth specs
   |
   v
Remove, correct, or flag unsupported claims
   |
   v
Return grounded answer with references
```

This can be implemented as a multi-step agent or as a deterministic validation layer around the LLM.

---

## Example: API validation

Suppose the user asks:

```text
How do I create a new invoice through the billing API?
```

The assistant should not answer from generic API knowledge. It should retrieve the actual OpenAPI spec and validate:

- Does the endpoint exist?
- What is the HTTP method?
- What path parameters are required?
- What request body fields are required?
- What response schema is returned?
- What authentication scope is needed?
- What error codes are documented?
- Are there deprecation warnings?
- Is this endpoint available for the current tenant, product, or API version?

A grounded answer might say:

```text
According to the approved OpenAPI spec, invoice creation uses POST /v2/invoices.
Required fields are customerId, currency, and lineItems. The endpoint returns an Invoice object with id, status, createdAt, and totalAmount. I do not see a documented PATCH endpoint for creating invoices, so I would not generate code against one.
```

That is much safer than:

```text
You can probably call /invoice/create with the customer and amount.
```

---

## Example: dataflow validation

Suppose the user asks:

```text
Explain the dataflow from intake form submission to CRM account creation.
```

A model may invent a plausible pipeline:

```text
Form -> API Gateway -> Kafka -> Enrichment Service -> CRM Service -> Salesforce
```

But the approved architecture may actually be:

```text
Form -> Intake API -> Temporal Workflow -> Validation Worker -> CRM Adapter -> DealCloud
```

To prevent hallucination, validate the generated dataflow against approved artifacts:

- HLSD
- ADRs
- workflow definitions
- service catalog
- OpenAPI specs
- event schema registry
- source code references
- runbooks

The answer should distinguish between confirmed facts and assumptions.

Example grounded phrasing:

```text
The approved HLSD shows the intake form calling the Intake API, which starts a Temporal workflow. The workflow calls the Validation Worker and then the CRM Adapter. I do not see Kafka listed in the approved dataflow, so I would not include Kafka unless we verify it from another spec or runtime trace.
```

---

## Claim extraction

One useful technique is to extract claims from the model's own draft before returning it.

Example draft:

```text
The assistant calls POST /v1/accounts, passes ownerId and companyName, then publishes AccountCreated to crm.account.events.
```

Extracted claims:

```json
[
  {
    "type": "api_endpoint",
    "claim": "POST /v1/accounts exists"
  },
  {
    "type": "request_field",
    "claim": "ownerId is a valid request field for POST /v1/accounts"
  },
  {
    "type": "request_field",
    "claim": "companyName is a valid request field for POST /v1/accounts"
  },
  {
    "type": "event",
    "claim": "AccountCreated is published to crm.account.events"
  }
]
```

Each claim can then be validated against the right source:

| Claim type | Validate against |
| --- | --- |
| REST endpoint | OpenAPI spec |
| Request field | OpenAPI request schema |
| Response field | OpenAPI response schema |
| Event name | AsyncAPI, schema registry, code definitions |
| Kafka topic | platform topic registry or runbook |
| Database table | DB schema or migration files |
| Service dependency | HLSD, service catalog, runtime traces |
| Data transformation | LLD, code, tests, workflow definitions |

---

## Confidence policy

The assistant should use different language depending on validation result.

| Validation result | Response behavior |
| --- | --- |
| Confirmed in approved spec | State as fact and cite/reference the source |
| Partially supported | State only the supported part and flag the gap |
| Not found | Do not present as fact |
| Conflicting sources | Explain the conflict and prefer the approved/current spec |
| Spec unavailable | Say the answer is an assumption or ask for the spec |

Good wording:

```text
I can confirm from the OpenAPI spec that POST /v2/invoices exists and requires customerId and lineItems. I do not see taxCode in the request schema, so I would not include it unless another approved spec confirms it.
```

Bad wording:

```text
The API probably supports taxCode because most billing APIs do.
```

---

## Runtime validation

For agents that generate executable code or perform actions, validation should go beyond text.

Examples:

- Validate generated API calls against OpenAPI before execution.
- Use JSON Schema validation for request bodies.
- Reject unknown fields unless the API explicitly allows them.
- Validate enum values.
- Check authentication scopes before attempting an action.
- Use dry-run or sandbox mode before production writes.
- Use contract tests for generated clients.
- Compare generated dataflow with runtime traces when available.

This turns hallucination prevention into a system property, not just prompt engineering.

---

## Where this fits in agent architecture

A practical architecture could be:

```text
User Question
   |
   v
Retriever
   |-- OpenAPI specs
   |-- ADRs / HLSDs / LLDs
   |-- Schema registry
   |-- Service catalog
   v
LLM Draft Generator
   |
   v
Claim Extractor
   |
   v
Spec Validator
   |
   v
Unsupported Claim Filter
   |
   v
Grounded Final Answer
```

For action-taking agents:

```text
Generated Tool/API Call
   |
   v
OpenAPI / JSON Schema Validation
   |
   v
Permission + Scope Check
   |
   v
Dry Run / Simulation when needed
   |
   v
Execution
   |
   v
Audit Log
```

---

## How this connects to bounded context

This also supports bounded context. Instead of carrying every old explanation in the conversation history, the assistant keeps references to source-of-truth artifacts and revalidates when needed.

For example, the conversation state might store:

```json
{
  "goal": "Explain invoice creation flow",
  "resolved_entities": {
    "api": "billing-api",
    "version": "v2",
    "operation": "createInvoice"
  },
  "evidence_refs": [
    "openapi:billing-api:v2:createInvoice",
    "hlsd:billing-dataflow:2026-05"
  ],
  "unsupported_claims": [
    "PATCH /v2/invoices was not found in OpenAPI"
  ]
}
```

Then on a later turn, the assistant can re-fetch the current spec instead of trusting the old conversation.

---

## Interview-ready framing

> For hallucination prevention, especially around APIs and dataflows, I would validate generated claims against source-of-truth artifacts. If the assistant says an endpoint exists, I would check the OpenAPI spec. If it says a field is required, I would check the request schema. If it describes an event or workflow, I would check AsyncAPI, schema registry, approved HLSD/LLD documents, or workflow definitions. I would also extract claims from the draft answer and run them through a validation layer before responding. Unsupported claims should be removed, corrected, or explicitly marked as assumptions. In production, this is stronger than prompt-only grounding because the system mechanically prevents the model from inventing endpoints, fields, dependencies, or dataflows.

---

## Stronger senior-level answer

> I treat the LLM as a reasoning and explanation layer, not as the authority on system contracts. For API and dataflow answers, the authority is the approved specification: OpenAPI for REST contracts, AsyncAPI or schema registry for events, database schemas for persistence, and ADR/HLSD/LLD documents for architecture decisions. The assistant can generate a draft, but then I would extract concrete claims like endpoint names, methods, fields, event topics, transformations, and service dependencies, and validate them against those specs. Anything not supported should either be removed, corrected, or labeled as an assumption. For action-taking agents, I would additionally validate the generated request body with JSON Schema, check auth scopes, and use dry-run or sandbox execution before writes. That reduces hallucinations and makes answers traceable, auditable, and safe enough for enterprise workflows.
