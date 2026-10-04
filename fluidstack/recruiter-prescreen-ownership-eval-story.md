# Recruiter Prescreen — End-to-End Ownership Story

## Core story

While building a multi-step deal-sourcing workflow, Hossein noticed that a seven-to-ten-step agent could look good in isolated demos while remaining fragile end to end because errors compound across steps.

No one had explicitly assigned an evaluation-platform project. Hossein initiated it because without a reusable workflow-level benchmark, the team could optimize for one customer or one prompt and still fail to generalize across customers.

## Strong framing

1. **Observation**
   - Multi-step workflows compound errors.
   - A good final answer can hide unstable intermediate behavior.
   - Client-specific success is not evidence of generalization.

2. **Initiative**
   - Proposed and started a dedicated evaluation framework without being assigned the work.
   - Defined the common workflow independent of a specific firm's investment thesis.

3. **Implementation**
   - Broke the workflow into generic stages.
   - Created representative benchmark cases.
   - Evaluated both step-level behavior and end-to-end task success.
   - Captured failure modes by stage so regressions could be diagnosed.
   - Used customer examples / focus-group workflows to make the benchmark realistic.

4. **Outcome**
   - Turned an ad hoc client-specific agent into something that could be tested systematically.
   - Created a reusable benchmark for comparing versions and preventing regressions.
   - Made it possible to decide whether a new model / prompt / tool actually improved the system.

## Recruiter-ready answer

"One example was when we started moving from a simple CRM assistant into a multi-step deal-sourcing agent.

The workflow could require seven to ten steps—understanding the investment thesis, retrieving candidates, enriching them from external sources, evaluating criteria, ranking companies, and then doing deeper research. I realized that even if each individual step looked reasonable, errors could compound across the workflow, so a system that worked well for one customer could still be very fragile.

No one had specifically asked me to build an evaluation platform, but I felt we couldn't responsibly scale the agent without one. So I initiated that work.

I first separated the customer-specific investment criteria from the generic workflow itself. Then I defined benchmark cases for each stage and for the full end-to-end task. That let us measure not only whether the final answer looked good, but where failures occurred—retrieval, tool selection, evidence extraction, ranking, or reasoning—and whether a new model or prompt actually improved the overall system.

We also used representative customer workflows to make those benchmark cases realistic.

That evaluation framework became a reusable benchmark for the CRM-agent work rather than something tied to a single client. The part I'm proud of is that it wasn't a feature somebody assigned to me. I saw that without it the entire agent architecture would remain fragile, so I took ownership of the problem and built the mechanism we needed to scale it."

## Coaching notes

- Avoid: "the chance of having a successful agent is slim."
- Prefer: "multi-step agent workflows compound error, so local quality does not guarantee end-to-end reliability."
- Avoid overusing "harness" with a recruiter; say "evaluation framework" or "workflow-level benchmark."
- Include one outcome sentence. Ownership stories need to end with what changed because of the work.

## Core message

**I identified a systemic reliability gap nobody had assigned, built the missing evaluation infrastructure, and made the agent workflow testable and scalable.**
