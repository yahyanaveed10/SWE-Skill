# Capacity Planning and Load Testing

Profiling tells you where time is going on a single request. Capacity planning tells you what happens when many requests arrive at once. They are complementary disciplines — neither replaces the other.

---

## The three kinds of testing under load

The names are sometimes used interchangeably; the goals are not.

**Load testing.** Run the system at expected peak load. Verify it behaves correctly — meets SLOs for latency and error rate. Answers: *can the system handle expected traffic?*

**Stress testing.** Increase load until the system breaks. Identify the failure mode (does latency degrade gracefully? do errors cascade? does it crash? does memory grow unboundedly?) and the breaking point. Answers: *where does the system break, and how?*

**Soak testing.** Run at sustained moderate load for many hours or days. Surfaces problems that only appear over time — memory leaks, file descriptor leaks, log volume issues, slow database growth, cache size growth. Answers: *can the system run continuously without degrading?*

Load tests pass without revealing leaks. Stress tests find break points without revealing realistic load. Soak tests find leaks without finding break points. Most production failures are caught by one of these three; running only one leaves blind spots.

---

## Designing representative load shapes

A load test that does not resemble production traffic gives confidence about a system you are not running. Common mistakes:

**Uniform traffic distribution.** Real traffic has bursts, time-of-day patterns, and hot keys. A uniform load misses behaviour under bursts and against hot data.

**Single endpoint.** Hitting `/health` 10000 times per second is not a load test. Real traffic mixes endpoints with different cost profiles. Test a realistic mix.

**Empty data.** A new test database with 1000 rows behaves differently from production with 100 million rows. Index hits, query plans, and cache behaviour all differ. Run load tests against production-sized data (or close to it).

**No think time.** Real users do not send requests as fast as possible. They wait, read, decide. Including think time in the load model produces realistic concurrency patterns; omitting it produces unrealistic concurrency that exhausts resources differently.

**No failure injection.** Real production has dependencies that fail intermittently. A load test that runs against perfect dependencies tells you nothing about behaviour when a dependency is slow or down. Combine load testing with chaos injection for realism.

---

## Reading load test results

The headline numbers are misleading. Drill into the distribution.

**p50 latency (median):** half of users are faster, half are slower. The "typical" experience.

**p95 latency:** 95th percentile. The experience of the 1-in-20 user. Often 3-10x p50.

**p99 latency:** 99th percentile. The experience of the 1-in-100 user. Often 10-50x p50.

**p99.9 latency (and p99.99):** the slowest 0.1% (and 0.01%). At high traffic, this is many users per minute. Tail latency.

**The trap:** average latency. If 99% of requests are fast and 1% are very slow, the average looks acceptable but the slow 1% is bad enough to drive churn. Use percentiles, not averages.

**Throughput vs. latency.** Often these trade off. A system can handle higher throughput by accepting higher latency (queueing more requests). The right operating point is the highest throughput that still meets the latency SLO. Push past that and latency degrades; pull back and throughput is wasted.

---

## Headroom and scaling triggers

**Headroom** is the gap between current load and the system's maximum sustainable load. A system at 80% capacity has 20% headroom. A system at 95% capacity has 5% headroom — and is one traffic spike from saturation.

**The signal of dangerous headroom:** latency is currently fine, but a small load increase causes large latency increases. The system is on the edge of a non-linear region. Capacity planning means understanding where that edge is.

**Scaling triggers** are the metrics that cause auto-scaling to add capacity. Common choices:
- CPU utilisation (e.g., scale up when CPU > 70%)
- Memory utilisation
- Request rate (RPS)
- Queue depth
- Latency

**The trap with CPU-based autoscaling:** for I/O-bound services, CPU may be low while threads/connections are exhausted. Scaling on CPU does nothing because adding instances doesn't help if each instance is waiting on I/O.

**The trap with reactive scaling:** scaling triggered by a metric exceeding a threshold has lag — the new instances take time to start up. By the time they are ready, the spike may be over (and you have over-provisioned) or the system may have already failed (because the spike outpaced the scaling). Predictive or schedule-based scaling can complement reactive scaling for known patterns.

---

## From load test results to infrastructure decisions

A load test tells you "this configuration handles X RPS at Y latency." To make sizing decisions, you need to project from this to expected load.

**Demand forecasting:** what is current production load, what is expected growth, what spikes occur, what is the largest spike to plan for? Without forecasts, capacity is sized for current load and immediately falls behind.

**Provisioning lead time:** how long does it take to add capacity? For cloud autoscaling, minutes. For specialised hardware or contracted capacity, weeks. The lead time determines how much headroom you need to keep — short lead time means less headroom is acceptable.

**Cost vs. headroom trade-off:** more headroom is more cost. Less headroom is more risk. Pick the trade-off explicitly based on cost of capacity vs. cost of brief overload.

---

## When to invest in load testing infrastructure

**Always:** for any service that is user-facing, has SLOs, or is on the critical path. Even a simple smoke test in CI ("can this endpoint handle 100 concurrent requests?") catches obvious regressions.

**Heavyweight investment (dedicated load testing environment, weekly load runs):** for high-stakes services where a brief outage costs significant revenue or trust, or for systems that frequently change behaviour under load.

**Skip:** for internal tools used by a few engineers, batch jobs that run on a schedule with no concurrency, or services where load is bounded and well-understood (e.g., one request per minute per user with 100 users).

The investment should be proportional to the cost of being wrong about capacity.
