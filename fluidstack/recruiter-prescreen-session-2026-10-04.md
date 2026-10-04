# Fluidstack Recruiter Prescreen Practice — October 4, 2026

## AI failure-mode question

Two useful examples came up in practice.

### 1. Source quality and freshness

When researching private companies, public information can be incomplete or stale. A result can sound credible while relying on outdated facts.

Better production controls:
- keep source provenance and timestamps;
- use a source hierarchy based on the type of fact;
- prefer authoritative structured or licensed data where appropriate;
- surface meaningful conflicts instead of silently resolving them;
- ask for human review when important evidence remains unresolved.

### 2. Missing domain rule

A recommendation system can produce a seemingly reasonable candidate that should have been excluded by a business rule, such as a company already belonging to the firm's existing portfolio.

The correct fix is deterministic: apply portfolio and eligibility exclusions before ranking and expensive research.

## Interview takeaway

The strongest lesson is that reliable AI is not only about improving prompts or models. Production quality often depends on domain rules, evidence provenance, freshness, deterministic validation, and human escalation around the model.

## Recruiter-ready answer

Two failure modes taught us a lot.

The first was source quality. Because we were researching private companies, important information about ownership, financing, and financial performance could be incomplete or stale on the open web. The system could find something that looked credible and present it confidently even though the underlying information was no longer current.

We addressed that by making provenance and source authority part of the system. We retained the source and timestamp behind important claims, used stronger sources for the facts they were best suited to establish, and surfaced material conflicts rather than letting the model silently choose one.

The second failure was simpler. The system could recommend a company that should already have been excluded by an existing-portfolio rule. The research looked reasonable, but the domain constraint had not been encoded explicitly.

The fix there was not a better prompt. We added a deterministic exclusion step before ranking or deep research.

The broader lesson was that many serious AI failures are systems problems rather than model problems. Reliable applications need explicit business rules, source-quality policies, provenance, validation, and human escalation around the model.
