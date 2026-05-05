---
name: observability
description: Observability as a software design concern — not an ops add-on. Use when writing code that will run in production, designing service boundaries, adding error handling, or diagnosing why a production failure is hard to debug. Covers structured logging, the difference between logs/metrics/traces, cardinality, instrumenting at service boundaries, and alerting discipline. Does not cover specific tooling (Datadog, Grafana, etc.) — signals are tool-agnostic.
---

# Observability

A system is observable if you can ask arbitrary questions about its runtime behaviour without deploying new code. That is a design property, not a monitoring property. Observability is built into code at write time — bolting it on after a production incident is expensive and incomplete.

For structured logging signals see [logging-signals.md](logging-signals.md).
For metrics and distributed tracing see [metrics-and-traces.md](metrics-and-traces.md).
For alerting discipline see [alerting-signals.md](alerting-signals.md).
For SLO design and error budget management see [slo-design.md](slo-design.md).

## The signal that a system is un-observable

- You need a developer to interpret the logs — the data is there but not structured enough to query
- You learn about failures from user reports or support tickets, not from your own instrumentation
- You cannot correlate a user complaint ("my request failed at 14:32") to a specific trace or log entry
- A new type of failure requires a code deployment to add visibility into it
- Debugging means adding print statements and redeploying

If any of these are true, observability is a design problem in the existing code, not a missing dashboard.

## Logs, metrics, and traces are not interchangeable

Each signal type answers a different question. Using only one leaves blind spots.

**Logs** answer: what happened at a specific moment? Discrete events. Best for debugging known failure modes once you know where to look. Expensive to store and query at scale.

**Metrics** answer: how is the system behaving over time in aggregate? Numerical state over time — error rates, latency percentiles, queue depth. Cheap at scale, good for alerting on known failure patterns. Cannot tell you why.

**Traces** answer: what was the path of this specific request across services? The signal that makes distributed systems debuggable. A trace captures timing at each service hop for a single request. Without traces, correlating a slow user experience to a specific downstream service call requires guesswork.

The practical rule: every service boundary and every external call needs a trace span. Logs should carry the trace ID so a log entry can be correlated to the trace it belongs to.

## Cardinality is the critical missing concept

High-cardinality fields (user ID, request ID, tenant ID, session ID, feature flag state) must be attached to every event. Without them, you can see that error rate increased — you cannot see which users, which tenants, or which code path.

Pre-aggregated metrics destroy cardinality. A metric that records `error_count` tells you errors increased. A structured log event with `user_id`, `tenant_id`, `endpoint`, `error_type` tells you which users on which tenants hit which error on which endpoint. The latter is what debugging requires.

**Design decision:** attach context fields at the request boundary (middleware, interceptor) and propagate them through the call stack. Not every function needs to explicitly pass them — use request-scoped context propagation.
