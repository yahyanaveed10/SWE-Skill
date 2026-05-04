---
name: swe-router
description: Technical concern router for software engineering work. Identifies which domain-specific skills to load based on task signals — covering modularity, anti-patterns, reuse, testability, security, performance, architecture, deployment, observability, data/API design, scoping, resilience, concurrency, and AI collaboration. Use when designing systems, reviewing code, making architecture decisions, handling data or APIs, debugging production issues, or building AI features. Does not cover process steps (use /understand, /plan, /implement, /debug, /review for those) or issue/branch flow (use issue-branch-orchestrator).
---

# SWE Technical Concern Router

Loads first on substantive engineering work. Identifies which domain skills are relevant and in what order. Does not replace process commands — it routes to technical knowledge those commands don't carry.

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

**The 70% problem.** Agents complete the happy path. What is consistently missing: error handling, timeouts, idempotency, observability, edge cases, and failure modes. Before declaring any implementation complete, ask: what happens when this fails? Is there a timeout? Is the error observable? Is a retry safe?

## Domain routing

Load the relevant skill based on task signals. Multiple skills may apply.

### modularity
**Load when:** designing or reviewing module/class boundaries, evaluating dependencies, deciding what to expose in a public interface, investigating why changes ripple unexpectedly.

Covers: coupling types (Content through Data), cohesion types, structural dependency signals (load-bearing modules, instability mismatches, cycles), SOLID as diagnostics.

### anti-patterns
**Load when:** reviewing code for structural problems, planning a refactor, noticing a class or module is doing too much, evaluating whether an existing design is causing maintenance pain.

Covers: Blob/God Class, Functional Decomposition, Golden Hammer, Design-by-Committee, Lava Flow, Spaghetti.

### reuse-and-patterns
**Load when:** deciding whether to use an existing library or build something new, evaluating whether a design pattern applies, designing for extension or variation.

Covers: GoF pattern signals (signal + overkill-when), reuse strategy trade-offs, patterns most often misapplied.

### testability
**Load when:** code is difficult to test, designing new code that needs a test suite, reviewing why tests are slow or brittle.

Covers: Controllability, Observability, Isolation as design properties, test smells, seam identification, Humble Object.

### security-engineering
**Load when:** designing authentication or authorisation, handling user input or external data, storing or transmitting sensitive data, threat-modelling a new feature, identifying trust boundaries.

Covers: STRIDE as reasoning prompts, secure design principles, common vulnerability signals (injection, IDOR, SSRF, path traversal).

Note: for reviewing existing code against a security checklist, use the `security-review` command.

### performance-engineering
**Load when:** investigating slow code or high resource usage, choosing data structures for a hot path, designing caching, evaluating whether an optimisation is worth its complexity cost.

Covers: measure-before-optimise, O(n²)/N+1/lock-contention signals, caching trade-offs.

### architecture-selection
**Load when:** starting a new system or major component, deciding between architecture styles, making a decision significant enough to be hard to reverse.

Covers: style trade-offs (monolith, microservices, event-driven, serverless, hexagonal), context-based decision rubric, ADR template.

### source-to-deployment
**Load when:** setting up or modifying CI/CD, containerising an application, designing a deployment strategy, working with IaC, debugging a build or release failure.

Covers: pipeline stage signals, container heuristics, deployment strategy trade-offs (rolling/blue-green/canary/feature flags), IaC signals.

### observability
**Load when:** writing code that will run in production, adding error handling, designing a service, diagnosing why a production failure is hard to debug, or when generated code has no logging.

Covers: structured logging, logs vs. metrics vs. traces, cardinality, four golden signals, distributed tracing, SLO-based alerting.

### data-and-api-design
**Load when:** designing a new schema, writing a migration, designing an API endpoint, adding a field to an existing API, versioning an API, or evaluating backward compatibility.

Covers: expand/contract migration pattern, expensive schema mistakes, backward compatibility as hard constraint, idempotency, API versioning, cursor pagination, null vs. absent.

### scoping-discipline
**Load when:** asked to "just make it work," estimating what a first version needs, deciding what to cut from scope, evaluating whether a prototype is ready to promote to production.

Covers: PoC vs MVP vs production requirements, what is safe to skip at each stage, the promotion checklist, cost-of-failure heuristic, over/under-scope signals.

### resilience-engineering
**Load when:** writing code that calls external services, databases, queues, or any dependency that can fail; designing retry logic; debugging cascading failures.

Covers: timeout (every outbound call), retry with exponential backoff and jitter, circuit breaker, bulkhead, graceful degradation, load shedding, partial failure in distributed calls.

### concurrency
**Load when:** writing multi-threaded or async code, reviewing code for thread safety, debugging non-deterministic failures, or when a system behaves differently under load than in sequential tests.

Covers: race condition signals (read-modify-write, check-then-act), async/await correctness, fire-and-forget errors, the concurrency testing gap.

### ai-collaboration
**Load when:** generating code with AI assistance, reviewing AI-generated code, building features where an ML model is the core product, evaluating AI-generated tests.

Covers: generation hallucination modes, AI test coverage gaps, ML system failure modes (training/serving skew, model rot, feedback loops).

## Routing by task signal

| Task signal | Primary skill(s) | Also consider |
|---|---|---|
| "Review this code / PR" | `anti-patterns` | `modularity`, `testability`, `security-engineering` |
| "Refactor this class / module" | `anti-patterns`, `modularity` | `testability`, `reuse-and-patterns` |
| "Design auth / permissions" | `security-engineering` | `architecture-selection` |
| "Why is this slow?" | `performance-engineering` | `architecture-selection` (if systemic) |
| "Should we use microservices?" | `architecture-selection` | `source-to-deployment` |
| "Set up CI/CD" | `source-to-deployment` | `testability`, `observability` |
| "This is hard to test" | `testability` | `modularity`, `anti-patterns` |
| "Use a library or build it?" | `reuse-and-patterns` | `architecture-selection` |
| "Add an ML feature" | `ai-collaboration` | `security-engineering`, `data-and-api-design` |
| "Review AI-generated code" | `ai-collaboration` | `anti-patterns`, `testability` |
| "Design a new system from scratch" | `architecture-selection` | `security-engineering`, `modularity`, `observability` |
| "Write a migration" | `data-and-api-design` | `source-to-deployment` |
| "Add a new API endpoint" | `data-and-api-design` | `security-engineering`, `observability` |
| "Is this PoC ready for production?" | `scoping-discipline` | `observability`, `resilience-engineering` |
| "Nothing in the logs / can't debug this" | `observability` | — |
| "Service goes down when dependency is slow" | `resilience-engineering` | `observability` |
| "Tests pass but prod fails sometimes" | `concurrency` | `testability` |

## Process command cross-reference

- `/understand` — inspect context before any change
- `/plan` — produce a short implementation plan
- `/implement` — make the smallest useful change
- `/test` — write and run tests
- `/review` — critical review of a plan or diff
- `/debug` — root cause before fixing

Load domain skills to ground the *technical judgment* those phases require. The commands handle the *process*.
