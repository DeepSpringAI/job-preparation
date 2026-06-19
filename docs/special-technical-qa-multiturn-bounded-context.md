# Special Technical QA: Multi-Turn Conversations with Bounded Context

## Q6. How would you support multi-turn conversations while keeping context bounded?

### What the interviewer is testing

This question is not only about token limits. It tests whether you understand how to design a production-grade conversational AI system that remains accurate, fresh, permission-aware, traceable, and cost-controlled across many turns.

A strong answer should cover:

- Context management
- Memory architecture
- Token budget control
- Retrieval freshness
- Permission-aware RAG
- Conversation state design
- Hallucination prevention
- Enterprise auditability and traceability

---

## Short interview answer

> I would not rely on raw chat history alone. I would maintain a structured conversation state object that contains the original user goal, resolved entities, retrieved evidence, open questions, and recent action history. Recent turns can stay verbatim, but older turns should be summarized into compact state. For every new turn, I would re-interpret the user intent, use the current state to resolve entities, and re-retrieve fresh evidence from the underlying systems instead of assuming prior retrieved content is still valid. Retrieved chunks should be deduplicated, ranked, permission-filtered, and capped to a fixed token budget. This keeps the conversation coherent while controlling cost, latency, drift, and security risk.

---

## Why bounded context matters

A naive chatbot keeps appending every message to the prompt:

```text
Turn 1
Turn 2
Turn 3
...
Turn 100
```

That approach does not work well in production because:

- Token cost grows continuously.
- Latency increases.
- Important facts get buried in irrelevant history.
- The model may over-focus on stale or misleading earlier context.
- The context window can eventually be exceeded.
- Retrieved documents or system records may have changed since earlier turns.
- Permission-sensitive evidence may no longer be safe to reuse without revalidation.

In enterprise AI, chat history is useful context, but it is not the source of truth.

---

## Production pattern

I separate conversation information into four major buckets.

### 1. Working memory

This contains the most recent turns, usually kept verbatim.

Typical content:

- The last few user and assistant messages
- The immediate task under discussion
- Recent clarifications
- Recent tool or retrieval outputs that are still directly relevant

Example:

```text
User: Why did the deployment fail?
Assistant: The logs suggest a database connection timeout.
User: Can you show me the affected service?
```

These recent turns are useful because they preserve conversational continuity and reduce over-summarization errors.

---

### 2. Structured session memory

Instead of keeping the entire transcript, I extract stable facts into a compact state object.

Example:

```json
{
  "goal": "Investigate production deployment failure",
  "entities": {
    "service": "payments-api",
    "environment": "production",
    "incident_id": "INC-4231"
  },
  "open_questions": [
    "Was there a deployment immediately before the failure?",
    "Did the database connection pool hit its limit?"
  ]
}
```

The principle is:

> Store facts, not the whole chat.

This helps the system stay focused even when the conversation becomes long.

---

### 3. Evidence memory

Evidence memory contains retrieved information from tools, documents, tickets, logs, CRM records, or knowledge bases.

Example:

```json
{
  "evidence": [
    {
      "source": "runbook_42",
      "claim": "payments-api can fail with DB pool exhaustion under high checkout traffic",
      "timestamp": "2026-06-18T10:15:00Z"
    },
    {
      "source": "datadog_metric_db_pool",
      "claim": "Connection pool saturation started at 14:32",
      "timestamp": "2026-06-18T14:35:00Z"
    }
  ]
}
```

I do not keep every retrieved chunk forever. I would:

- Deduplicate repeated chunks.
- Keep source IDs and timestamps.
- Rank evidence by relevance and freshness.
- Keep only the top N chunks in the prompt.
- Preserve citations or references for traceability.
- Revalidate permissions before using evidence in later turns.

The goal is to avoid turning the context window into a pile of old retrieval results.

---

### 4. Long-term memory or user/project memory

Some information may be useful across sessions, but it should not be blindly loaded into every prompt.

Examples:

```json
{
  "project": "CRM migration",
  "customer": "Acme Corp",
  "preferred_dashboard": "Grafana",
  "preferred_response_style": "technical and concise"
}
```

This type of memory should be:

- Stored outside the prompt.
- Loaded only when relevant.
- Permission-aware.
- Auditable.
- Easy to update or delete.

---

## Conversation state object

A practical state object might look like this:

```python
class ConversationState:
    user_goal: str
    resolved_entities: dict
    evidence_refs: list
    open_questions: list
    action_history: list
    constraints: dict
    last_summary: str
```

Example instance:

```json
{
  "user_goal": "Find the root cause of the checkout outage",
  "resolved_entities": {
    "service": "payments-api",
    "environment": "production",
    "cluster": "prod-us-west-2"
  },
  "evidence_refs": [
    "runbook_42",
    "incident_INC-4231",
    "datadog_db_pool_metric"
  ],
  "open_questions": [
    "Was there a release before the outage?",
    "Was traffic unusually high?"
  ],
  "action_history": [
    "Checked deployment status",
    "Retrieved error logs",
    "Compared DB pool metrics"
  ],
  "constraints": {
    "permission_scope": "current_user_acl",
    "freshness_required": true
  },
  "last_summary": "The investigation is focused on payments-api in production. Current evidence suggests DB connection pool exhaustion, but deployment timing has not yet been confirmed."
}
```

This state object becomes the compact memory of the task.

---

## Summarization strategy

Every few turns, or when the token budget crosses a threshold, older messages should be summarized into structured state.

Example pattern:

```python
if token_count(conversation_history) > threshold:
    state.last_summary = summarize_old_turns(conversation_history)
    conversation_history = keep_recent_turns(conversation_history, n=6)
```

Before summarization:

```text
20 turns of conversation
```

After summarization:

```text
Compact summary of stable facts
+
Structured state object
+
Last 4-6 turns verbatim
+
Top ranked evidence chunks
```

This keeps the model grounded while avoiding unlimited context growth.

---

## Why re-retrieve every turn

This is one of the most important points.

A weak answer says:

> I keep the retrieved documents in conversation memory.

A stronger answer says:

> I use conversation state to guide retrieval, but I re-retrieve from source systems on each new turn when freshness matters.

Example:

```text
Turn 1: Deployment status is SUCCESS.
Turn 8: User asks, "What is deployment status now?"
```

If the assistant only relies on old context, it may incorrectly answer:

```text
Deployment status is SUCCESS.
```

But the real system may have changed. The correct production pattern is:

```text
New user turn
  -> interpret intent
  -> resolve entities from state
  -> retrieve fresh source-of-truth data
  -> permission-filter results
  -> rank and deduplicate evidence
  -> answer with citations or traceable evidence
```

This avoids stale answers.

---

## Permission-aware context

Permission awareness is critical in enterprise systems.

Suppose User A can access:

```text
Contract A
Contract B
```

User B can access:

```text
Contract A
```

If evidence from Contract B remains in shared memory or is reused without revalidation, the system may leak unauthorized information.

Therefore, retrieved evidence should be:

- Associated with source permissions.
- Revalidated against the current user.
- Filtered at retrieval time and answer time.
- Avoided in summaries when it may cross permission boundaries.
- Logged for auditability when used in an answer.

A safe design treats ACLs as part of the retrieval and context-building pipeline, not as a UI-only concern.

---

## Token budget policy

A useful prompt budget might be divided like this:

```text
System instructions:        fixed budget
Developer/task policy:      fixed budget
Recent conversation:        last 4-8 turns
Structured state:           compact JSON or YAML
Evidence:                   top ranked N chunks
Tool results:               only current relevant outputs
Answer space:               reserved generation budget
```

The important design point is that every category has a cap. The system should never allow old conversation history or old retrieval results to consume the entire context window.

---

## Traceability and auditability

For enterprise workflows, especially legal, finance, healthcare, and CRM, the assistant should be able to explain where an answer came from.

That means preserving:

- Source document IDs
- Ticket IDs
- CRM object IDs
- Timestamps
- Retrieval query metadata
- User permission scope
- Tool calls or action history

This matters because users may ask:

```text
Why did you say this?
Where did this answer come from?
Was this based on the latest contract?
Did you use the right customer record?
```

A bounded-context system should still be explainable.

---

## Failure modes to mention

A strong interview answer can also mention common failure modes:

| Failure mode | Mitigation |
| --- | --- |
| Context drift | Maintain original goal and open questions in state |
| Stale evidence | Re-retrieve from source systems when freshness matters |
| Token explosion | Summarize older turns and cap evidence chunks |
| Lost details | Keep recent turns verbatim and extract stable facts |
| Duplicate evidence | Deduplicate by source ID, chunk ID, and semantic similarity |
| Permission leakage | Revalidate ACLs every retrieval and avoid unsafe summaries |
| Hallucination | Answer only from current evidence when source-grounded output is required |
| Unclear references | Maintain resolved entities and ask clarifying questions when needed |

---

## Architecture sketch

```text
User Turn
   |
   v
Intent + Entity Extraction
   |
   v
Conversation State Update
   |
   v
Fresh Retrieval / Tool Calls
   |
   v
Permission Filtering
   |
   v
Evidence Deduplication + Ranking
   |
   v
Context Builder
   |-- recent turns
   |-- compact state
   |-- top evidence
   |-- open questions
   v
LLM Response
   |
   v
State + Evidence Reference Update
```

---

## Strong final answer for interview

> I separate context into working memory, structured session state, retrieved evidence, and optional long-term memory. Recent turns stay verbatim, but older turns are periodically summarized into a compact state object that captures the user's goal, resolved entities, constraints, open questions, and action history. I do not rely only on chat history because enterprise data changes; each new turn should use the current state to re-resolve intent and re-retrieve fresh evidence from the source systems. Retrieved evidence is deduplicated, ranked, permission-filtered, and capped to a fixed token budget. This keeps the system coherent across turns while controlling cost and latency. It also improves accuracy, traceability, and security because answers are grounded in fresh, permission-aware evidence rather than stale conversation text.

---

## Intapp-style framing

> In legal and CRM workflows, context has to remain bounded, permission-aware, and traceable. For example, in a multi-step matter, deal, or customer workflow, I would keep the original user goal, resolved entities such as matter ID or account ID, prior evidence references, and open questions as structured state. But I would re-retrieve documents, CRM records, or workflow objects on each relevant turn because permissions and underlying records may change. That approach gives the user a continuous conversation while keeping the system safe, fresh, and auditable.
