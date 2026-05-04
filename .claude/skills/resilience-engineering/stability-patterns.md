# Stability Patterns

Patterns for keeping a system stable when dependencies fail. From "Release It!" (Nygard) — the canonical reference.

---

## Timeout

**What it is:** A maximum time to wait for a response from a dependency. If the response does not arrive within the timeout, the call fails immediately rather than blocking indefinitely.

**Why it is not optional:** Without a timeout, one slow dependency can exhaust all available threads/connections in the calling service. The calling service becomes unavailable — not because of its own failure, but because it is waiting for someone else.

**Setting the timeout:** The timeout should be shorter than the caller's own timeout. If your API must respond within 5 seconds, your call to a downstream service cannot timeout after 10 seconds. Work backwards from the user-facing latency requirement.

**Separate connect timeout from read timeout.** A connect timeout (how long to wait for the connection to be established) and a read timeout (how long to wait for data after the connection is open) are different failure modes. Both should be set.

---

## Retry with exponential backoff and jitter

**What it is:** Retrying a failed request after a delay, with the delay increasing exponentially on each attempt.

**Why exponential:** Retrying immediately after a failure amplifies load on an already-struggling dependency. If 100 clients all retry at the same time with zero delay, the dependency receives 100x its normal load exactly when it is least able to handle it. Exponential backoff spreads the retry load over time.

**Why jitter:** Without jitter, all clients that started at the same time will retry at the same time, even with exponential backoff. Jitter (random variation in the delay) desynchronises retries across clients.

**Typical backoff:** base_delay * (2 ^ attempt) + random_jitter. E.g.: 100ms, 200ms, 400ms, 800ms — each with ±50ms jitter.

**What to retry:** Only idempotent operations — operations where retrying produces the same result as a single call. A GET is generally safe to retry. A POST that creates a resource is not, unless the API provides idempotency keys.

**Maximum retry count:** Set a maximum. Retrying indefinitely under sustained failure keeps load on a dependency that needs time to recover.

---

## Circuit Breaker

**What it is:** A component that monitors failures to a dependency and stops sending requests when the failure rate exceeds a threshold — giving the dependency time to recover. After a cooldown period, it allows a small number of test requests through. If those succeed, the circuit closes and normal operation resumes.

**Three states:**
- **Closed:** Normal operation. Requests flow through. Failures are counted.
- **Open:** Failure threshold exceeded. Requests fail immediately without hitting the dependency. The dependency gets time to recover.
- **Half-open:** Cooldown elapsed. A small number of test requests are allowed through. If they succeed, circuit closes. If they fail, circuit reopens.

**When to use:** When a dependency has a known failure mode that benefits from a recovery period, and when failing fast (failing immediately rather than waiting for a timeout) provides better user experience than waiting.

**When not to use:** Circuit breakers add complexity. For dependencies that must always be called (a database for a write operation), a circuit breaker that opens leaves no fallback. Add a circuit breaker only when there is a meaningful fallback or when fast-fail is genuinely better than a timeout.

---

## Bulkhead

**What it is:** Isolating resources (thread pools, connection pools) by dependency so that exhaustion caused by one dependency does not affect others.

**The problem it solves:** Without isolation, a slow dependency that exhausts the shared thread pool takes down all other functionality in the service. With bulkheads, each dependency gets its own resource pool. A failing dependency exhausts its own pool, not the shared one.

**When to use:** When a service calls multiple dependencies with different reliability characteristics, and allowing one dependency's failure to exhaust shared resources would cause unacceptable collateral damage.

**Cost:** Each pool requires sizing. Under-sized pools cause unnecessary rejections; over-sized pools waste resources. Requires monitoring pool utilisation per dependency.
