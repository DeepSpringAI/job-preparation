# Recruiter Prescreen — AI Failure Example

Two useful failure patterns from the DealCloud sourcing workflow:

1. **Weak or stale external evidence** — public information about private companies could be incomplete or out of date. The production lesson was to keep provenance and timestamps, define source-authority rules, and surface important conflicts rather than letting the model silently choose.

2. **Missing domain invariant** — the workflow could recommend a company that should have been excluded by an existing-portfolio rule. The correct fix was a deterministic exclusion check before ranking or deep research, not another prompt.

## Interview takeaway

Reliable AI is not only a model-quality problem. Strong production systems surround the model with explicit domain rules, source-quality policies, freshness checks, provenance, deterministic validation, and human escalation.

## Recruiter-ready framing

"Two failures taught us a lot. One was source quality: private-company information on the open web could be incomplete or stale, so a result could sound convincing while relying on outdated evidence. We improved that by tracking provenance and timestamps, establishing source authority by data type, and surfacing conflicts instead of letting the model silently resolve them.

The second was a missing business rule. The system could recommend a company that should already have been excluded from the candidate universe. We fixed that with a deterministic eligibility check before ranking and deep research.

The lesson was that production AI reliability often comes from the system around the model: domain rules, source policies, validation, and escalation—not just a better prompt."