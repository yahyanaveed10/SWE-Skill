---
name: resilience-engineering
description: Designing software systems that handle failure gracefully. Use when writing code that calls external services, databases, queues, or any dependency that can fail; when a service needs to remain partially functional when a dependency is down; when designing retry logic; or when a system is experiencing cascading failures. Covers timeouts, retry with backoff, circuit breakers, bulkheads, graceful degradation, and partial failure in distributed calls. Does not cover infrastructure-level resilience (multi-region, CDN failover) — see architecture-selection.
---

# Resilience Engineering

Every dependency can fail. The question is not whether a downstream service, database, or queue will fail — it is whether your system handles that failure gracefully or propagates it into a user-visible outage.

Agents consistently generate the happy path. Resilience is what happens after the happy path fails.

For stability patterns (timeout, retry, circuit breaker, bulkhead) see [stability-patterns.md](stability-patterns.md).
For graceful degradation strategies see [graceful-degradation.md](graceful-degradation.md).
For partial failure in distributed calls see [partial-failure.md](partial-failure.md).

## The default failure mode

Without explicit resilience design, the failure mode of most systems is:

1. Dependency becomes slow or unavailable
2. Calls to the dependency block (no timeout)
3. Thread pool / connection pool exhausts
4. The entire service becomes unavailable — not just the feature that uses the dependency
5. Users see a complete outage for a problem that should have been limited to one feature

This is not a hypothetical. It is the most common cause of cascading failures in production systems. Every external call without a timeout is a ticking clock.

## The minimum resilience floor

For any code that calls a dependency (HTTP, database, queue, cache, external API), the minimum before deploying to production:

- **Timeout on every call** — never let a call block indefinitely
- **Retry with backoff** — retrying immediately amplifies load on an already-struggling dependency
- **Error handling** — the failure must be caught and handled; it must not propagate as an unhandled exception to the caller

Everything above this floor (circuit breakers, bulkheads, graceful degradation) adds resilience at a cost. The floor is not optional.
