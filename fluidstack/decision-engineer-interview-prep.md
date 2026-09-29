# Fluidstack Decision Engineer Interview Preparation

## Goal

Prepare for the Fluidstack Decision Engineer interview loop by building fluency across three connected areas:

1. **Personal experience narrative** — especially Intapp, private equity CRM, deal sourcing, due diligence, and agentic AI.
2. **AI data-center and compute infrastructure** — enough technical and operational depth to speak credibly with Fluidstack engineers.
3. **Situational Awareness / AI scaling thesis** — understand the compute, power, industrial, and strategic context behind Fluidstack's mission.

The preparation is designed around the likely interview loop:
- Hiring manager
- Technical / system-design
- Behavioral / culture

---

## Track A — Intapp / Private Equity / Decision Engineering Story

### A1. Private Equity domain foundation
Be able to explain clearly:
- What private equity firms do
- Fundraising, LPs, GPs, funds, portfolio companies
- Deal sourcing
- Qualification and screening
- Due diligence
- Investment committee workflow
- Relationship intelligence and CRM
- Portfolio monitoring
- Exit process

### A2. Intapp CRM context
Build a concise story for:
- What the CRM product was
- Who used it
- Why private-equity workflows are difficult
- What data existed in the system
- Why conventional CRM/search/workflow automation was insufficient
- Where AI added value

### A3. Deal-sourcing AI use case
Practice a complete product/system story:

1. Define target users and workflow
2. Define target company / investment thesis
3. Gather internal + external information
4. Entity resolution
5. Enrichment
6. Ranking / qualification
7. Explainability
8. Human review
9. Feedback loop
10. Evaluation and production monitoring

Technical topics:
- embeddings and semantic retrieval
- structured + unstructured data
- ranking models
- LLM extraction
- RAG
- tool calling
- knowledge graphs
- data quality
- permissions / enterprise security
- evaluation metrics

### A4. Due-diligence agent
Design an agent that can:
- ingest CIMs, financial statements, contracts, CRM notes, emails, and external sources
- identify relevant entities and claims
- generate diligence questions
- call tools / retrieve supporting evidence
- identify inconsistencies and missing information
- produce a structured diligence memo
- cite evidence
- escalate uncertainty to humans

Key design distinction:
**workflow first, autonomy second.**

Discuss:
- deterministic stages vs agentic stages
- state machine / orchestration
- tool permissions
- memory
- human-in-the-loop
- hallucination control
- traceability
- evaluation
- latency / cost
- security

### A5. Decision-engineering framing
Practice reframing the Intapp story as:

domain expert judgment
→ structured representation
→ AI reasoning
→ decision support
→ action/workflow automation
→ feedback

This should map directly to Fluidstack's Decision team.

### A6. Interview-ready versions
Prepare the same Intapp story at:
- 30 seconds
- 90 seconds
- 3 minutes
- 8–10 minute deep dive

---

## Track B — AI Data Centers and Compute Infrastructure

### B1. Data-center architecture: mental model

Learn the stack from utility power to model training:

Grid / generation
→ substation
→ transformers / switchgear
→ UPS / backup
→ power distribution
→ racks
→ GPU servers
→ networking
→ storage
→ cluster scheduler
→ training / inference workload

Be fluent in:
- MW vs GW
- power density
- PUE
- redundancy (N, N+1, 2N)
- availability
- commissioning

### B2. GPU server basics
Understand:
- GPU / accelerator node
- CPU host
- HBM
- NVLink / NVSwitch
- PCIe
- NIC / DPU
- local NVMe
- rack architecture

Know why large-model training is different from ordinary cloud workloads.

### B3. Cluster networking
Learn:
- scale-up vs scale-out networks
- InfiniBand / Ethernet
- RDMA / RoCE
- leaf-spine topology
- bandwidth, latency, congestion
- collective communication
- all-reduce
- topology-aware scheduling

### B4. Training cluster software
Understand:
- bare metal
- containers
- Kubernetes
- Slurm
- NVIDIA GPU Operator
- scheduling
- job queues
- gang scheduling
- topology awareness
- health checks
- observability

Fluidstack specifically exposes managed Kubernetes and Slurm for frontier training workloads.

### B5. Reliability / operations
Be able to reason about:
- failed GPU
- failed NIC
- thermal issue
- node degradation
- network partition
- straggler
- storage bottleneck
- job failure

Operational systems:
- telemetry
- Prometheus / Grafana
- DCIM
- BMS / EPMS
- Redfish / IPMI / BMC
- incident response
- burn-in / qualification

### B6. Cooling
Learn:
- air cooling
- direct-to-chip liquid cooling
- CDU
- cooling loops
- chilled water
- closed-loop systems
- heat rejection
- thermal constraints

### B7. Data-center delivery lifecycle
Understand the sequence:

site selection
→ power acquisition
→ permits
→ design
→ procurement
→ construction
→ equipment delivery
→ installation
→ commissioning
→ cluster integration
→ burn-in
→ customer acceptance
→ live compute

Important constraints:
- transformers
- switchgear
- generators
- cooling equipment
- GPUs
- networking
- construction sequencing
- utility interconnection

### B8. Decision-engineering use cases inside a data center
Practice designing AI/software for:
- procurement-delay prediction
- construction schedule risk
- change-order analysis
- commissioning tracking
- equipment failure triage
- incident summarization
- root-cause support
- GPU fleet qualification
- capacity planning
- maintenance workflows

---

## Track C — Situational Awareness and the AI Compute Scaling Thesis

### C1. Core argument
Read and understand Leopold Aschenbrenner's *Situational Awareness: The Decade Ahead*.

Focus on:
- scaling laws
- compute scaling
- algorithmic progress
- unhobbling
- growing training clusters
- capital requirements
- power requirements
- industrial mobilization

Do not memorize claims as facts; understand the argument and its assumptions.

### C2. Compute progression
Develop intuition for:
- increasingly expensive training runs
- 10B → 100B → potentially trillion-dollar compute programs
- why power becomes a first-class constraint
- why GPU supply alone is insufficient
- grid, transformers, construction, networking, cooling, and operations as bottlenecks

### C3. Power and industrial infrastructure
Study:
- generation capacity
- transmission
- substations
- interconnection queues
- grid constraints
- gas / nuclear / renewable generation
- behind-the-meter power
- supply-chain constraints

### C4. Critical analysis
Be able to discuss:
- what the essay predicts
- what assumptions drive those predictions
- where uncertainty is high
- what has happened since 2024
- what would falsify or weaken the thesis

The target is thoughtful understanding, not ideological agreement.

### C5. Connection to Fluidstack
Explain why Fluidstack exists if the scaling thesis is roughly correct:

AI capability growth
→ demand for larger clusters
→ compute becomes scarce
→ power + construction become bottlenecks
→ infrastructure speed becomes strategic
→ software must coordinate physical deployment at unprecedented scale

---

## Track D — Technical Interview Preparation

### D1. Agent/system-design questions
Practice:
- diligence agent
- procurement agent
- construction-risk agent
- GPU incident agent
- contract-analysis agent

Answer framework:

1. clarify objective
2. define users
3. define decision / output
4. identify data
5. define system boundaries
6. design data model
7. select deterministic vs AI components
8. design agent/tools
9. handle uncertainty
10. evaluate
11. secure
12. monitor
13. iterate

### D2. AI evaluation
Study:
- benchmark design
- golden datasets
- precision / recall
- extraction accuracy
- retrieval recall
- ranking metrics
- LLM-as-judge limitations
- human evaluation
- online metrics
- failure taxonomy

### D3. Production AI
Cover:
- model routing
- structured outputs
- retries
- caching
- latency
- cost
- rate limits
- observability
- prompt/version management
- fallbacks
- data privacy

### D4. Coding
Practice practical Python:
- parsing structured/unstructured data
- transformations
- API interaction
- graph/tree traversal
- queues
- ranking
- simple algorithms

Priority is readable production reasoning rather than puzzle tricks.

---

## Track E — Hiring Manager / Leadership Interview

Prepare strong answers to:

- Tell me about yourself.
- Why Fluidstack?
- Why Decision Engineering?
- Why leave / why now?
- What did you own at Intapp?
- What was technically difficult?
- What business outcome changed?
- Tell me about ambiguity.
- Tell me about moving fast.
- Tell me about influencing domain experts.
- Tell me about a disagreement.
- Tell me about a failed approach.
- What would you build first if you joined Fluidstack?

Use Intapp + startup experience as anchor stories.

---

## Track F — Behavioral / Culture

Map stories to Fluidstack's stated operating principles:

### Be a barrel
Ownership, autonomy, scope expansion.

### Insane urgency
Fast execution under deadlines.

### Reason from first principles
Challenge assumptions and redesign systems.

### Love of the game
Demonstrate sustained interest in AI and infrastructure without performative enthusiasm.

### Build something that actually matters
Connect work to measurable outcomes and users.

For each principle, prepare:
- one primary story
- one backup story
- failure / learning variant

---

## Track G — Mock Interview Sequence

### Mock 1: Narrative baseline
- 30-minute hiring-manager simulation
- identify weak / vague parts of Intapp story

### Mock 2: Deal-sourcing system
- end-to-end architecture
- aggressive follow-up questions

### Mock 3: Due-diligence agent
- agent architecture
- evaluation
- enterprise constraints

### Mock 4: Data-center fundamentals
- rapid-fire terminology
- explain concepts without jargon

### Mock 5: Data-center decision system
Example:
"We are building five AI data centers simultaneously. Equipment and schedule data are spread across emails, spreadsheets, ERP, BIM, and contractors. Design a system that predicts schedule risks and drives action."

### Mock 6: Situational Awareness
Discussion rather than trivia:
- explain thesis
- challenge assumptions
- connect to infrastructure

### Mock 7: Culture
30 minutes of behavioral probing.

### Mock 8: Full loop
Three consecutive 30-minute rounds:
1. hiring manager
2. technical
3. behavioral

---

## First Preparation Order

Recommended order:

1. Build the **Intapp master narrative**
2. Deep dive on **deal sourcing**
3. Deep dive on **due diligence agent**
4. Learn **data-center fundamentals**
5. Learn **GPU cluster architecture**
6. Learn **data-center build / commissioning lifecycle**
7. Read and discuss **Situational Awareness**
8. Practice **Fluidstack-specific decision systems**
9. Build behavioral story bank
10. Run full mock loop

---

## Working Notes

This file should evolve during preparation.

After every coaching/mock session, add:
- Questions asked
- Candidate answer summary
- Strong points
- Missing concepts
- Better answer
- Follow-up drills
- Vocabulary / facts to review

