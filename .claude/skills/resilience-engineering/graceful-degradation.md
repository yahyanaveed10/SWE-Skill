# Graceful Degradation

A system that can only succeed or completely fail provides no partial value when dependencies are unavailable. Graceful degradation is the design of what the system does — and what it tells the user — when it cannot do everything it normally does.

---

## The core design question

For every feature that depends on an external service or non-critical resource, ask: **what should the system serve if this dependency is unavailable?**

The answer determines the degradation strategy. There is always an answer — the question is whether it was designed or discovered during an incident.

---

## Degradation strategies

**Serve stale data.** Cache the last successful response and serve it when the dependency is down. The user gets data that may be slightly out of date rather than an error. Appropriate when freshness requirements are not strict (a product catalogue, a user's profile, a public dashboard). Requires a cache with TTL and clear UI labeling when serving stale data.

**Return a reduced feature set.** Disable the features that depend on the unavailable dependency; serve everything else. A payment service being down should not disable the product browsing experience. A recommendation engine being down should not prevent checkout.

**Return a static fallback.** Return a hardcoded or pre-generated response when the live response is unavailable. Appropriate for low-stakes features (a "featured items" widget that returns the same items for all users when the personalisation service is down).

**Fail fast with a clear message.** Sometimes there is no meaningful fallback — a feature requires the dependency to be useful. In that case, fail immediately (do not wait for timeout) with a user-readable message that explains what is unavailable and when to try again. This is better than a hanging spinner or a generic 500 error.

---

## Load shedding

When a service is overloaded — more traffic than it can handle — serving all requests poorly is worse than declining some requests explicitly. Load shedding intentionally rejects low-priority requests to preserve capacity for high-priority ones.

**Signals that load shedding is needed:**
- Latency increases linearly with load (the system is saturated)
- A traffic spike causes full outage rather than graceful slowdown
- Queue depth grows unboundedly under load

**Implementation:** Return `503 Service Unavailable` with a `Retry-After` header when the service is saturated. Prioritise requests by type if possible (authenticated over anonymous, paid tier over free tier). Always shed load before the system becomes completely unavailable.

---

## What the user should see during degradation

**Never a blank page or a silent hang.** The worst degraded experience is one where the user does not know whether to wait or leave. Every degraded state must have a visible indication.

**Distinguish between "this specific feature is unavailable" and "the system is unavailable."** Users handle partial unavailability better than total unavailability. A message like "recommendations are temporarily unavailable — you can still browse and check out" preserves more user trust than a generic error.

**If you are serving stale data, say so.** "Showing prices from 5 minutes ago" is more trustworthy than showing prices that may be wrong without notice.
