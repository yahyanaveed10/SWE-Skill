# Metrics and Traces

---

## The four golden signals (Google SRE)

If you instrument nothing else, instrument these four for every service:

| Signal | What it measures | Why it matters |
|---|---|---|
| **Latency** | Time to handle a request (successful and failed separately) | Degraded latency is often the first sign of a problem |
| **Traffic** | Request rate (requests per second, events per minute) | Baseline for understanding whether changes in other signals are load-driven |
| **Errors** | Rate of failed requests (explicit 5xx and implicit — wrong results, timeouts) | The signal users feel directly |
| **Saturation** | How full the service is (CPU %, memory %, queue depth, connection pool usage) | Predicts failure before it happens |

**Separate latency for successful and failed requests.** A spike in error rate will pull down average latency because errors are often fast. If you combine them, error spikes look like performance improvements.

---

## Metrics: what to measure and what to avoid

**What to measure:**
- Business-level metrics: conversions, checkouts, signups, jobs processed — what the system exists to do
- SLO-relevant metrics: the metrics that feed your availability and latency commitments
- Resource saturation: connection pool utilisation, queue depth, cache hit rate
- Error rates by type and endpoint — not just total error count

**What not to measure:**
- Everything. High-cardinality dimensions (user ID, request ID) as metric labels will explode your time-series storage. Use structured logs for per-request attribution; use metrics for aggregates.
- Metrics that only confirm what you already know. CPU usage on a batch job is expected to be high — it is not a useful alert signal.

**Histograms, not averages.** Average latency hides tail latency. p99 latency — the latency experienced by the slowest 1% of requests — is often 10x the average and is what users with problems experience. Report p50, p95, p99.

---

## Distributed tracing

A trace is the record of a single request's path through a distributed system, with timing at each hop. Without traces, correlating a slow API response to a specific database query or downstream service call requires log archaeology across multiple services.

**What a span captures:** service name, operation name, start time, duration, parent span ID (to link to the calling service), and arbitrary key-value attributes (user ID, database table, HTTP status).

**Where to create spans:**
- Every inbound request to a service (the root span)
- Every outbound call: HTTP requests, database queries, cache reads, queue publishes
- Every significant internal operation: file reads, external API calls, heavy computations

**Propagate context across service boundaries.** The trace ID must be passed in HTTP headers (W3C `traceparent` is the standard), message metadata, or async job payloads. Without propagation, each service creates isolated traces instead of a connected trace of the full request path.

**Sampling:** tracing every request at high traffic is expensive. Use head-based sampling (decide at the root span entry) or tail-based sampling (decide after the trace completes, keeping slow or errored traces). Never drop errored traces.

---

## Connecting logs, metrics, and traces

The three signal types are most valuable when correlated:
- Log entries carry the trace ID → you can jump from a log entry to the full request trace
- Metrics alert you that something is wrong → you drill into traces to find which request type or user segment is affected → you jump to a specific trace → you jump from the trace span to the log entries in that span

This flow requires that the trace ID is propagated and attached consistently. It is an infrastructure investment — but without it, debugging distributed systems requires guessing.
