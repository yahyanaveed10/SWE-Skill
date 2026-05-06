---
name: performance-engineering
description: Performance investigation, optimisation discipline, and capacity planning. Use when investigating slow code or high resource usage, choosing data structures or algorithms for a hot path, designing caching strategies, evaluating whether an optimisation is worth its complexity cost, or designing load tests and sizing infrastructure for expected traffic. Covers measure-before-optimise as the hard rule, reading profiling output, O(n²)/N+1/lock-contention signals, caching trade-offs, and load/stress/soak testing for capacity planning. Infrastructure-level performance decisions (sharding, CDN, queues) belong in architecture-selection.
---

# Performance Engineering

**The hard rule: measure before optimising.**

Optimising without measurement is guessing. Guessing produces complex code that does not measurably improve performance. The cost of a profiling pass is always lower than the cost of a wrong optimisation.

For profiling discipline and reading output see [profiling-first.md](profiling-first.md).
For common complexity and bottleneck signals see [complexity-signals.md](complexity-signals.md).
For caching trade-offs see [caching-heuristics.md](caching-heuristics.md).
For capacity planning and load/stress/soak testing see [capacity-and-load-testing.md](capacity-and-load-testing.md).

## Performance dimensions

**Response time** — time between a request arriving and a response being sent. What the user experiences directly.

**Throughput** — requests served per unit time. A system can have good response time at low load and poor throughput under concurrency if resources are contended.

**Resource utilisation** — CPU, memory, I/O, network. High utilisation is not inherently a problem; unused capacity is wasted cost. The question is whether utilisation causes response time or throughput degradation.

These three often trade off. Optimising for throughput (batching, buffering) can increase individual response time. Optimising for response time (eager loading, caching) can increase memory utilisation.

Know which dimension is the actual problem before optimising.

## The premature optimisation trap

Premature optimisation adds complexity before there is evidence that the complexity is needed. It is the source of most performance-related technical debt: code that is harder to read, harder to change, and often no faster than the simpler version would have been.

Ask before optimising:
- Is there a measured performance problem, or is this anticipated?
- If there is a problem, is this code path the bottleneck, or is the bottleneck elsewhere?
- If this is the bottleneck, how much improvement is needed to meet the actual requirement?
- Is the optimisation complexity worth the measured gain?

An optimisation that adds significant complexity for a 2% improvement on a non-bottleneck path is a poor trade.
