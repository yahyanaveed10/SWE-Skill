---
name: swe-router
description: Technical concern router for software engineering work. Identifies which domain-specific skills to load based on task signals — covering modularity, anti-patterns, reuse, testability, security design, performance, architecture selection, deployment, AI/ML engineering, and AI collaboration. Use when designing systems, reviewing code for structural or quality issues, making architecture decisions, handling sensitive data, investigating performance, setting up pipelines, or building AI/ML features. Does not cover process steps (use /understand, /plan, /implement, /debug, /review for those) or issue/branch flow (use issue-branch-orchestrator).
---

# SWE Technical Concern Router

Loads first on substantive engineering work. Identifies which of the ten domain skills are relevant and in what order. Does not replace process commands — it routes to technical knowledge those commands don't carry.

## Hard rules (always apply, regardless of domain)

These are non-negotiable. They are the anti-hallucination layer.

**Verify before asserting.** Never claim a function, method, CLI flag, file path, environment variable, schema field, or package version exists without confirming it. If you cannot verify, state the assumption explicitly:
> "Assumption: I have not confirmed that `X` exists. Verify before using."

**Separate facts from assumptions.** In any non-trivial output, distinguish:
- Observed — directly read from code, config, logs, or output
- Assumed — inferred without direct evidence
- Unknown — cannot be determined without further inspection

**State the unknown rather than guess.** When the correct value is unclear: name it as unknown, suggest how to verify, do not fill the gap with a plausible-sounding value.

**Stop before destructive or irreversible actions.** Do not proceed with deleting files, dropping tables, force-pushing, resetting state, or adding new external dependencies without surfacing the action and its consequences first.

## Domain routing

Load the relevant skill based on task signals. Multiple skills may apply to a single task.

### modularity
**Load when:** designing or reviewing module or class boundaries, evaluating dependencies between components, deciding what to expose in a public interface, investigating why changes ripple unexpectedly across the codebase.

Covers: coupling types and metrics (CBO, RFC, Instability), cohesion types, SOLID as diagnostic signals, dependency cycle detection.

### anti-patterns
**Load when:** reviewing code for structural problems, planning a refactor of messy or legacy code, noticing a class or module is doing too much, evaluating whether an existing design is causing maintenance pain.

Covers: Blob/God Class, Functional Decomposition, Golden Hammer, Design-by-Committee, Lava Flow, Spaghetti — each with detection signals and refactoring reasoning prompts.

### reuse-and-patterns
**Load when:** deciding whether to use an existing library or build something new, evaluating whether a design pattern applies, designing something that will need to vary or extend over time.

Covers: GoF pattern signals (when a pattern fits vs. when it adds unnecessary indirection), reuse strategy trade-offs (adopt library / adopt framework / copy-and-adapt / extract abstraction).

### testability
**Load when:** code is difficult to test and the reason is unclear, designing new code that will need a test suite, reviewing why a test suite is slow or brittle, refactoring to improve test coverage without changing tests.

Covers: Controllability, Observability, Isolation as design properties (not testing steps), test smells, seam identification.

### security-engineering
**Load when:** designing authentication or authorisation flows, handling user input or external data, storing or transmitting sensitive data, threat-modelling a new feature or integration, identifying trust boundaries.

Covers: STRIDE threat modelling as reasoning prompts, secure design principles (least privilege, fail secure, defence in depth), common vulnerability signals (injection, IDOR, SSRF, path traversal).

Note: for reviewing existing code against a security checklist, use the `security-review` command instead.

### performance-engineering
**Load when:** investigating slow code or high resource usage, choosing data structures or algorithms for a hot path, designing caching, evaluating whether an optimisation is worth its complexity cost.

Covers: measure-before-optimise discipline, O(n²)/N+1/lock-contention signals, caching trade-offs.

Note: infrastructure-level performance decisions (sharding, CDN, queues) belong in `architecture-selection`.

### architecture-selection
**Load when:** starting a new system or major component, deciding between architecture styles (monolith, microservices, event-driven, serverless, hexagonal), making a decision significant enough to be hard to reverse.

Covers: style trade-off rubrics (fits-when / doesn't-fit-when), context-based decision dimensions (team size, data ownership, operational maturity), ADR template.

### source-to-deployment
**Load when:** setting up or modifying a CI/CD pipeline, containerising an application, designing a deployment strategy, working with infrastructure-as-code, debugging a build or release failure.

Covers: pipeline stage signals, container heuristics, deployment strategy trade-offs (rolling / blue-green / canary), IaC value signals.

### ai-engineering
**Load when:** building features that use machine learning models, setting up training or inference pipelines, evaluating model quality, integrating an ML model into production.

Covers: ML pipeline discipline, evaluation-first thinking, MLOps failure modes (drift, reproducibility, monitoring gaps).

### ai-collaboration
**Load when:** an agent (including the current one) is generating code, reviewing AI-generated code before use, designing guardrails for an AI-assisted workflow, evaluating AI-generated tests.

Covers: generation guardrails, hallucination failure modes in code generation, AI test coverage gaps, human-in-the-loop gates for AI-assisted requirements.

## Routing by task signal

| Task signal | Primary skill(s) | Also consider |
|---|---|---|
| "Review this code / PR" | `anti-patterns` | `modularity`, `testability`, `security-engineering` |
| "Refactor this class / module" | `anti-patterns`, `modularity` | `testability`, `reuse-and-patterns` |
| "Design auth / permissions" | `security-engineering` | `architecture-selection` |
| "Why is this slow?" | `performance-engineering` | `architecture-selection` (if systemic) |
| "Should we use microservices?" | `architecture-selection` | `source-to-deployment` |
| "Set up CI/CD" | `source-to-deployment` | `testability` |
| "This is hard to test" | `testability` | `modularity`, `anti-patterns` |
| "Use a library or build it?" | `reuse-and-patterns` | `architecture-selection` |
| "Add an ML feature" | `ai-engineering` | `ai-collaboration`, `security-engineering` |
| "Review AI-generated code" | `ai-collaboration` | `anti-patterns`, `testability` |
| "Design a new system from scratch" | `architecture-selection` | `security-engineering`, `modularity` |

## Process command cross-reference

These commands handle the engineering process and are not duplicated here:

- `/understand` — inspect context before any change
- `/plan` — produce a short implementation plan
- `/implement` — make the smallest useful change
- `/test` — write and run tests
- `/review` — critical review of a plan or diff
- `/debug` — root cause before fixing

Load domain skills to ground the *technical judgment* those phases require. The commands handle the *process*.
