# Architecture Decision Records

An Architecture Decision Record (ADR) captures a significant architectural decision: the context, the options considered, the choice made, and the consequences.

## When to write an ADR

Not every technical decision needs an ADR. Write one when:
- The decision is hard to reverse (switching architectures, choosing a data store, defining service boundaries)
- The decision will affect multiple teams or components
- The rationale will not be obvious to future contributors who see only the outcome
- There are legitimate competing options and the choice is a trade-off, not a clear winner

Do not write an ADR for: choice of variable names, code formatting decisions, which library to use for a utility function, or anything that can be changed in a day.

## ADR template

```
# ADR-[number]: [Short title]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-N]

## Context
What is the situation or problem that necessitates a decision?
What constraints exist (technical, organisational, time)?
What makes this decision non-trivial?

## Options considered
### Option A: [Name]
[Brief description]
Pros: ...
Cons: ...

### Option B: [Name]
[Brief description]
Pros: ...
Cons: ...

## Decision
We chose [Option X] because [reasoning grounded in the context and constraints above].

## Consequences
### Positive
- [What becomes easier or better]

### Negative
- [What becomes harder or worse — be honest]

### Risks
- [What could go wrong and how we will monitor for it]
```

## Where to store ADRs

`docs/decisions/` or `docs/adr/` in the repository. Name files `NNN-short-title.md` (e.g., `001-use-postgresql.md`). Store them in version control alongside the code — they are part of the system's history.

## Keeping ADRs useful

- Mark superseded ADRs as `Superseded by ADR-N` rather than deleting them — the historical record of what was decided and why matters
- New decisions that contradict an old one get a new ADR that references the old one
- The "Consequences" section must be honest about trade-offs — an ADR that only lists positives was written to justify a decision already made, not to document it
