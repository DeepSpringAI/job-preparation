# Interview Experience Stories: Intapp and Other Projects

This document gives five experience-based interview questions and answer scripts. Each answer is designed to support about three minutes of spoken explanation. The goal is not to memorize every sentence, but to keep the structure, technical depth, and business framing consistent.

Use the following pattern when answering:

1. Start with the business context.
2. Explain the technical problem.
3. Describe your specific role and decisions.
4. Explain the architecture, tradeoffs, or implementation.
5. End with measurable or practical impact.

---

## 1. Intapp — Building AI and ML systems for professional-services workflows

### Question
Can you walk me through the kind of AI/ML work you did at Intapp and how it connected to real business workflows?

### Three-minute answer
At Intapp, my work was focused on applying AI and machine learning to professional-services workflows, especially in domains like legal, finance, and relationship intelligence where the cost of mistakes is high and the data is sensitive. The important point is that this was not just generic model development. The systems had to fit into real enterprise workflows, respect permissions, integrate with existing products, and produce outputs that users could trust.

One of the main themes was turning fragmented enterprise data into useful signals. In products such as DealCloud, Intake, Time, and Terms-related workflows, the data was often spread across CRM records, documents, emails, matter information, time entries, firm-specific metadata, and structured business objects. My role was to design and build ML and AI capabilities that could reason over these sources and help users make better decisions or automate repetitive steps.

Technically, this involved a combination of classical ML, NLP, ranking, retrieval, entity extraction, and later LLM-style architecture patterns. For example, when dealing with documents or internal knowledge, the problem was not only retrieving relevant text. We also had to preserve traceability: why did the system produce this recommendation, which record or document supported it, and whether the user had permission to see that evidence. That shaped the architecture. We needed pipelines for ingestion, normalization, metadata extraction, search indexing, and model inference, but also guardrails around authorization, explainability, and auditability.

My specific contribution was usually at the architecture and implementation boundary. I worked on how to represent the data, how to connect models to product workflows, how to evaluate outputs, and how to make the system reliable enough for enterprise use. For example, if a model produced a suggestion inside an intake or CRM workflow, the product could not behave like a toy chatbot. It had to be grounded in approved data, support human review, and degrade gracefully when confidence was low.

The biggest lesson from Intapp was that enterprise AI is mostly about workflow integration and trust. A model can be accurate in isolation but still fail in production if it cannot explain itself, if it violates permissions, if it ignores firm-specific terminology, or if users cannot act on the result. So my approach became very system-oriented: build the model, but also build the data contracts, evaluation process, permission model, feedback loop, and product integration around it.

That experience is very relevant to modern GenAI systems. Whether we call it RAG, agents, copilots, or workflow automation, the core problem is the same: connect AI to real business context in a way that is secure, explainable, measurable, and useful.

### Key points to remember
- Do not describe Intapp as only model training.
- Emphasize enterprise workflows, permissions, traceability, and trust.
- Mention professional-services complexity: legal, finance, CRM, documents, and business objects.
- End with the idea that production AI is a system, not just a model.

---

## 2. Intapp — Designing bounded-context, multi-turn AI systems

### Question
Tell me about a time you had to support a complex multi-step or multi-turn workflow. How did you keep the system accurate and prevent context drift?

### Three-minute answer
A good example comes from my Intapp experience, where many workflows were multi-step by nature. In professional-services environments, users are not usually asking one isolated question. They might be working through a client intake, a deal review, a matter analysis, a document workflow, or a CRM update. The system needs to remember what the user is trying to accomplish, but it also cannot blindly rely on the entire conversation history because that becomes expensive, noisy, and sometimes stale.

The way I think about this problem is to separate conversation memory from workflow state. Conversation memory is what the user said in natural language. Workflow state is the structured representation of what the system has resolved so far: the original goal, entities such as client name or matter ID, retrieved evidence, open questions, assumptions, and pending actions. That structured state is much more reliable than just appending more chat history.

In a production architecture, I would maintain a state object that includes the user goal, resolved entities, evidence references, and open decisions. Every new turn updates that state. But I would also re-retrieve evidence when needed instead of assuming the previous evidence is still valid. This matters in enterprise systems because records, permissions, documents, or workflow status can change. For example, if a user asks a follow-up about a client or opportunity, the system should use the resolved entity from the prior turn, but still validate current data from approved sources before making a claim.

At Intapp, this mindset was important because systems had to stay permission-aware and traceable. If the system summarized something from a document or CRM record, we needed to know which source supported it. If the user was not authorized to access a record, the system should not leak that information through a generated answer. So the architecture had to combine retrieval, authorization, evidence tracking, and response generation.

To keep context bounded, I use a few practical techniques. I summarize older conversation turns into structured state, keep only the most recent interaction window, deduplicate evidence, and maintain citations or references to source records. I also separate facts from assumptions. If something is inferred, it should be marked as inferred. If something is missing, the system should either ask a targeted question or abstain from making a claim.

The broader lesson is that multi-turn AI should behave like a workflow engine, not just a chat transcript. The conversation is the interface, but the durable state, retrieval layer, permissions, and evaluation logic are what make it reliable. That is the approach I would bring to any enterprise GenAI or agentic system.

### Key points to remember
- Use “conversation memory vs workflow state.”
- Mention state object: goal, entities, evidence, open questions, assumptions.
- Mention re-retrieval because enterprise data may change.
- Tie it back to Intapp: permission-aware, traceable, workflow-heavy systems.

---

## 3. Intapp — Preventing hallucinations in enterprise AI systems

### Question
How do you prevent hallucinations when building AI systems that make claims about APIs, dataflows, documents, or business records?

### Three-minute answer
My approach is that hallucination prevention is not only a prompt-engineering problem. In enterprise systems, especially the kind of systems I worked on at Intapp, hallucination prevention has to be designed into the architecture. The model should not be treated as the source of truth. It should be treated as a reasoning and language layer on top of approved systems, schemas, documents, and workflows.

At Intapp, we dealt with business-critical workflows where an incorrect claim could create legal, financial, or operational risk. For example, if a system is explaining a client intake workflow, a CRM record, a contract term, or a dataflow between systems, the answer has to be grounded in approved evidence. It is not acceptable for the model to invent an API, assume a field exists, or describe a workflow step that is not actually part of the system.

The first control is retrieval grounding. The system should retrieve from approved sources: OpenAPI specs, database schemas, product documentation, workflow definitions, ADRs, architecture docs, or source records. The generated answer should be constrained by that evidence. If the evidence does not support a claim, the system should either avoid the claim or label it as an assumption.

The second control is validation. For API-related answers, generated claims can be checked against OpenAPI or approved service contracts. For dataflow claims, the system can validate against architecture diagrams, event schemas, lineage metadata, or HLSD-level documentation. For database claims, it can check the actual schema. This turns hallucination prevention from a language problem into a verification problem.

The third control is traceability. Every important answer should carry references to the evidence used. Internally, that means keeping source IDs, document chunks, schema names, timestamps, and permission checks. Externally, that may appear as citations, links, or explanation cards. The user should be able to inspect why the system gave an answer.

The fourth control is abstention. If the system does not have enough evidence, it should say so. In enterprise AI, a well-calibrated “I do not have enough approved evidence to answer that” is much safer than a confident but unsupported answer.

My practical design is: retrieve approved evidence, generate a draft, validate factual claims against source-of-truth systems, attach evidence, and only then return the answer. Over time, failed or low-confidence queries should be reviewed with architecture leads or product owners. That review process becomes one of the best evaluation signals because it reveals where documentation, schemas, or retrieval coverage are weak.

The key lesson from Intapp is that hallucination prevention is a full lifecycle discipline: data governance, retrieval, validation, permissions, citations, abstention, and review loops. Prompting helps, but production reliability comes from system design.

### Key points to remember
- Say “the model is not the source of truth.”
- Mention OpenAPI, schemas, ADRs, HLSDs, dataflows, and approved docs.
- Mention validation and abstention.
- Tie it to business risk and enterprise trust.

---

## 4. Medstream / Vitachain — Building healthcare intake and workflow automation

### Question
Can you describe another project outside Intapp where you built a production-oriented AI or workflow system?

### Three-minute answer
Outside Intapp, one important area of my work was healthcare workflow automation, including Medstream / Vitachain-style intake and operational systems. The business context was very different from legal or professional services, but the core technical challenge was similar: there are many fragmented steps, sensitive data, and human decisions that need to be supported rather than blindly automated.

In healthcare intake, the workflow usually starts with messy inputs: referrals, forms, patient information, insurance details, eligibility data, scheduling constraints, clinical notes, and communications from multiple parties. The goal is to reduce manual work, but also preserve safety, auditability, and human oversight. You cannot simply build a chatbot and let it make final decisions without controls.

My approach was to think in terms of an orchestrated workflow. Each step should be explicit: ingest the document or request, extract structured information, validate required fields, classify urgency or routing category, check for missing information, communicate with the right party, and escalate high-risk or ambiguous cases to a human. This maps very naturally to agentic or workflow-based architectures, but with strict guardrails.

Technically, this kind of system needs OCR or document parsing, entity extraction, normalization, rules, confidence scoring, queue management, audit logging, and integration with scheduling or EHR-style systems. I would not rely on one large model to do everything. Instead, I would decompose the workflow into smaller components: an extraction component, a validation component, a triage component, a communication component, and a human-review component.

The healthcare domain also forces strong non-functional requirements. Privacy, auditability, reliability, and traceability are not optional. Every automated decision or recommendation should be explainable. If the system extracts insurance information or patient demographics, the source should be visible. If the system decides that a referral is incomplete, it should explain which required fields are missing. If the confidence is low, it should not pretend to be certain.

My specific contribution in these projects was the architecture thinking: how to move from a manual workflow to a semi-automated operational system without losing control. That means designing data models, state transitions, human-in-the-loop checkpoints, exception handling, and integration points.

The main impact is operational leverage. These systems can reduce repetitive administrative work, speed up intake, improve consistency, and give staff better visibility into where each case stands. But the important design principle is that AI should assist the workflow, not hide it. In high-stakes environments like healthcare, the best system is one that automates the routine parts while making exceptions, risks, and missing information more visible to humans.

### Key points to remember
- Use healthcare as contrast to Intapp: different domain, same production-AI principles.
- Emphasize intake, document parsing, triage, scheduling/EHR integration.
- Mention privacy, audit, human approval, and exception handling.
- Avoid claiming fully autonomous medical decisions.

---

## 5. DeepSpring / Enterprise Agent Projects — Building company-specific AI operating layers

### Question
What have you worked on more recently with agentic AI, RAG, or enterprise copilots?

### Three-minute answer
More recently, my work has focused on building company-specific AI operating layers through DeepSpring-style enterprise agent projects. The idea is that most companies do not need a generic chatbot. They need an AI system that understands their workflows, documents, systems, terminology, and operating constraints. That requires a different architecture from a normal chat interface.

The starting point is usually a discovery and context-building phase. Before building agents, I try to understand the company’s actual workflow: who does what, which systems are involved, what documents matter, where approvals happen, where delays occur, and which decisions are repeated. For example, in an oilfield services or regulated business environment, the workflow may go from customer request to commercial review, operations planning, HSE checks, field execution, work tickets, invoicing, and revenue recognition. AI only becomes useful when it is mapped to that operational reality.

Technically, the architecture has several layers. First is the context layer: connectors to documents, databases, APIs, CRM, ERP, emails, or other business systems. Second is the retrieval and indexing layer, where information is chunked, normalized, embedded, indexed, and filtered by permissions. Third is the reasoning layer, where LLMs or agents use tools to answer questions, draft outputs, inspect records, or trigger workflow steps. Fourth is the control layer: evaluation, human approval, audit logs, data boundaries, and monitoring.

A key design principle is that agents should not be allowed to freely improvise over the business. They should operate through tools and approved data. For example, if an agent answers a question about a client, invoice, field operation, or regulatory package, it should retrieve the relevant records, cite the source, and respect permission boundaries. If the system needs to take action, there should be approval gates and clear audit trails.

My role in these projects is to define the architecture and implementation approach: what the first workflow should be, what data is needed, how to build the retrieval layer, how to design the agent tools, and how to measure success. I also focus heavily on selecting a narrow wedge for the first pilot. Instead of trying to automate the entire company immediately, I prefer choosing one painful workflow with clear ROI, such as request-to-revenue visibility, regulatory-package preparation, intake automation, or document-to-decision workflows.

The main lesson is that enterprise agents are not just LLM prompts. They are workflow systems with context, tools, permissions, evaluation, and human control. My experience at Intapp and in healthcare made me very sensitive to those requirements, and DeepSpring is a way to apply that same production-oriented thinking to broader enterprise AI transformation.

### Key points to remember
- Position this as recent agentic AI and enterprise RAG work.
- Use “company-specific AI operating layer,” not generic chatbot.
- Mention discovery, connectors, indexing, retrieval, tools, permissions, approvals, audit.
- Emphasize pilot wedge and ROI.

---

# 15-minute interview flow

If you need to speak for about 15 minutes, use this order:

1. Start with Intapp as the foundation: production AI inside enterprise workflows.
2. Explain multi-turn and bounded-context design as your system-design depth.
3. Explain hallucination prevention as your GenAI reliability depth.
4. Use Medstream / Vitachain to show domain transfer into healthcare operations.
5. Use DeepSpring to show current agentic AI direction and leadership thinking.

A strong closing statement:

> Across all of these projects, the common thread is that I do not treat AI as a standalone model. I design it as part of a production system: grounded in real data, connected to workflow, permission-aware, measurable, and safe enough for enterprise users. That is the experience I would bring to this role.
