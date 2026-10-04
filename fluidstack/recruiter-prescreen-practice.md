# Fluidstack Recruiter Prescreen Practice — Decision Engineer, Business Operations

## Context
Recruiter / Talent Partner screen. Goal is to communicate seniority, fit, motivation, and a clear connection between Intapp experience and Fluidstack Decision Engineering without overloading the conversation with deep technical detail.

## Question: Why Fluidstack and why this role?

### Candidate's natural answer
Hossein connected Fluidstack to the growing importance of compute availability for AI, including power, data centers, cooling, infrastructure, and workload design. He described the Decision Engineer role as forward-deployed and attractive because it translates expert problems into AI-enabled software systems.

### What worked
- Strong macro motivation: compute infrastructure is a critical bottleneck for AI.
- Demonstrates genuine interest in the infrastructure layer, not just LLM applications.
- Correct instinct that Decision Engineering is about working closely with domain experts and translating workflows into software.
- Connects to the future of AI engineering and high-leverage operational automation.

### Corrections / improvements
- For the Business Operations role, do not say "works directly with the client." The current posting says the engineer embeds with Fluidstack's finance, treasury, accounting, legal, and people teams.
- Avoid over-indexing on wiring/cooling/data-center design in the recruiter screen. That is relevant to Fluidstack generally, but the Business Operations role is specifically about internal operational systems such as hiring, cash forecasting, capital flows, contracts, and continuous close.
- Avoid saying the role is mainly about "harnesses, logic, and prompts for coding agents." The posting emphasizes production software, LLM APIs, MCP servers, agentic frameworks, and AI coding tools, but the core outcome is automating real business workflows.
- Add the direct bridge to Intapp: understanding expert workflows, translating them into structured data and tools, building AI systems around them, and driving adoption with non-engineering stakeholders.
- Avoid superlatives such as "probably the most promising job in tech" in a recruiter conversation. Genuine enthusiasm is good; a grounded reason is stronger.

### Recommended structure
1. Why Fluidstack: AI progress increasingly depends on compute infrastructure and the physical/software systems that deliver it.
2. Why Decision Engineering: turning expert judgment and manual workflows into software/AI systems.
3. Why Business Operations: finance/legal/recruiting workflows are complex, high-value, and rich in structured + unstructured information.
4. Why me: Intapp experience translated domain workflows and customized enterprise data into production AI systems while working with business stakeholders.
5. Close with excitement about combining hands-on engineering with domain-facing ownership.

### Recruiter-ready answer

"I’m interested in Fluidstack for two related reasons. First, I think compute infrastructure is becoming one of the fundamental constraints on how quickly AI can progress. It’s not just about having better models; you need power, data centers, networking, cooling, hardware, and the software systems that let all of that operate at scale. Fluidstack is working directly at that layer, which I find very compelling.

The second reason is the Decision Engineer role itself. What stood out to me is that it’s not a traditional backend or ML role where requirements are handed to you. The engineer embeds with domain experts, understands how the work is actually done, and then turns that judgment and workflow into software—often using LLMs, agents, and structured systems.

That maps closely to what I enjoyed most at Intapp. I worked with business users and product teams to understand complex private-capital workflows, translate customized enterprise data into something AI could reason over, and then ship production systems that users could actually incorporate into their workflow.

For Business Operations specifically, I like that the problems are concrete and consequential—things like hiring, cash forecasting, contracts, and capital workflows. So for me it combines three areas I care about: hands-on AI engineering, solving ambiguous business problems, and working on infrastructure that is important to the next phase of AI."

## Likely recruiter follow-up
"Can you give me an example of a time you worked closely with non-engineering stakeholders to understand a business process and turn it into a technical solution?"


## Question: Example of working with non-engineering stakeholders

### Candidate's natural answer
Hossein described working with financial analysts in private-equity customer focus groups after releasing the first conversational CRM interface. Through observing and discussing their workflow, he realized the larger pain point was not simply retrieving CRM records; analysts spent substantial time enriching candidate companies from sources such as PitchBook and FactSet, interpreting less-structured attributes such as founder-led ownership, and comparing the evidence against the firm's investment strategy. That observation motivated an agentic deal-sourcing workflow that automated more of the research and qualification process.

### What worked
- Strong discovery story: customer observation changed the product direction.
- Shows interaction with domain experts rather than receiving static requirements.
- Clearly distinguishes information retrieval from the higher-value research/decision workflow.
- Good examples of external data sources and implicit attributes.
- Strong match to Fluidstack's current Business Operations expectation that Decision Engineers embed with experts and turn real workflows into structured software.

### Improvements
- Lead with the outcome: "We initially thought retrieval was the core problem; customer observation showed us research and qualification were the real bottleneck."
- Use "private-equity investment professionals / analysts" rather than "financial analyst of a focus group of private equities."
- Clarify the discovery method: observe workflow, ask why each step exists, capture decision criteria, then prototype and review with users.
- Replace vague phrases such as "agentic interface" with a concrete pipeline: thesis criteria → retrieve candidates → external enrichment → evidence extraction → ranking → deep research → user review.
- End with what changed: reduced manual research / produced an evidence-backed shortlist / created a repeatable benchmark, depending on the claim Hossein chooses to substantiate.
- Mention tradeoffs only briefly in recruiter screen: focus deep research on highest-value candidates to manage latency/cost.

### Recruiter-ready version
"A good example was after we launched the first conversational interface for DealCloud. We initially thought the biggest problem was helping users retrieve information from a highly customized CRM, and the chatbot did make that much easier.

But I worked directly with investment professionals in customer focus groups and spent time understanding what they did after they got the initial list of companies. That showed us the larger bottleneck. They would take those companies, go into sources such as PitchBook and FactSet, research each one, look for attributes that were not always represented cleanly as structured fields—things like whether a business was founder-led—and then compare all of that against the firm's investment strategy. That process required a lot of manual research and judgment.

So instead of just asking users whether they liked the chatbot, we mapped their actual workflow step by step and identified where AI could create the most leverage. That led us toward a deal-sourcing workflow: translate the investment thesis into explicit criteria, retrieve a candidate set, enrich each company from approved external sources, extract the relevant evidence, rank the candidates, and then perform deeper research on the strongest opportunities.

We kept reviewing that workflow with the customer groups, both to improve the product and to create realistic benchmark cases for evaluation. The important lesson for me was that the best AI use case was not the first one we imagined. It came from working closely with the domain experts, understanding where their time and judgment were really being spent, and then redesigning the system around that workflow."

### Likely follow-up
"How did you decide what parts of that workflow should be handled by AI versus deterministic software or human judgment?"


## Question: How do you decide AI vs deterministic software vs human-in-the-loop?

### Candidate's natural answer
Hossein described decomposing the firm's investment strategy into explicit scoring criteria, evaluating candidate companies against those criteria, using AI/tool calls to gather evidence from the CRM, web, and external data providers, and tracking confidence for each criterion. When confidence fell below a threshold, the workflow paused for human clarification or review before continuing.

### What worked
- Strong use of explicit investment criteria rather than opaque end-to-end scoring.
- Good instinct to track confidence at the criterion/evidence level.
- Clear human-in-the-loop escalation path instead of forcing an answer.
- Good description of asynchronous/long-running research behavior.
- Matches the general direction of production agentic systems with guardrails, authorization, and evals.

### Improvements
- Answer the three-part question explicitly: AI for ambiguity, deterministic code for invariants, humans for judgment/low-confidence decisions.
- Avoid implying the confidence number is always a calibrated model probability. Call it a confidence/evidence sufficiency score unless there is a validated probabilistic model.
- Add examples of deterministic logic: permission checks, exact calculations, schema validation, scoring rules, workflow state, rate/cost limits.
- Clarify that the human is not only a fallback for model uncertainty; humans should also approve consequential actions and define ambiguous business preferences.
- Mention that the system should preserve evidence/provenance so the user can see why a criterion was scored.

### Recruiter-ready version
"We tried to separate the workflow into three categories. I use AI where the problem is ambiguous or unstructured, deterministic software where the rules need to be exact, and human judgment where either the evidence is insufficient or the decision is consequential.

For deal sourcing, we first translated the firm's investment strategy into explicit criteria. Then the AI could gather and interpret evidence from sources such as our CRM, PitchBook, and other research sources. For each criterion, the system kept the evidence it found and an indication of how confident or complete that evidence was.

But things like permissions, numerical calculations, schema validation, workflow state, and the scoring rules themselves were handled deterministically. I don't want an LLM deciding whether a user is authorized to see data or doing an important financial calculation from free-form text.

Then we had human-in-the-loop checkpoints. If the investment thesis was ambiguous, if the evidence for an important criterion was weak, or if the system was about to take a consequential action, we would pause and ask the user to clarify or approve the next step. Once that information came back, the workflow could continue.

So the principle was controlled autonomy: automate as much of the research as possible, but make uncertainty visible and keep humans involved where their judgment actually matters."

### Likely recruiter follow-up
"Tell me about a time this approach failed or where the AI produced a result that looked reasonable but was wrong. How did you catch it and improve the system?"
