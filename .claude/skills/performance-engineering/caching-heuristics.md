# Caching Heuristics

Caching adds complexity. It is worth the complexity only when the trade-off is understood.

## When caching adds value

**The data is expensive to compute and cheap to store.** CPU-intensive computations, complex aggregations, results of slow external calls — these are candidates. A simple database lookup is usually not.

**The data is read far more than it is written.** High read-to-write ratio means the cached value is used many times before it becomes stale. A cache that is invalidated on every write and read equally often adds complexity with no benefit.

**Stale data is acceptable for the use case.** A product catalogue that updates hourly can tolerate a 5-minute cache. A bank balance cannot.

**The same data is requested by multiple callers.** Caching is most effective when it serves many requests with a single computation.

## Cache invalidation trade-offs

Cache invalidation is where most caching bugs originate. The two approaches:

**Time-based expiration (TTL)** — the cache entry expires after a fixed time and is recomputed on the next request.
- Simple to implement
- Stale data window is bounded and predictable
- Does not require coordinating the cache with writes
- Correct choice when freshness requirements allow a bounded staleness window

**Event-based invalidation** — the cache entry is invalidated when the underlying data changes.
- Provides near-real-time freshness
- Requires the write path to know about and invalidate the cache
- Coupling between write path and cache layer — a missed invalidation produces stale data silently
- Correct choice when staleness is not acceptable but write frequency is low

**Ask:** What is the acceptable staleness window? How frequently does the underlying data change? Who needs to know when the cache is invalidated?

## What not to cache

**Fast data that changes frequently.** If the data changes faster than the TTL, the cache provides no benefit and adds the risk of serving stale data.

**Data that is cheap to compute.** If the computation takes less than the cache lookup, caching adds overhead without benefit.

**Data with high cardinality and low reuse.** If every request has a unique cache key, the cache has a 0% hit rate and just adds memory pressure.

**Security-sensitive data without careful scoping.** A cache that leaks data across users (serving user A's data to user B because they share a cache key) is a security vulnerability.

## Cache warming

A cold cache (empty at startup) causes a thundering herd — many requests all miss the cache simultaneously and all trigger the underlying computation at once. This can overwhelm the underlying system.

**Signals that warming is needed:** Long startup time before the system reaches steady-state performance, spikes in database or compute load at startup or after a cache flush.

**Options:** Pre-populate the cache on startup before accepting traffic, use a background job to keep high-value keys warm, implement request coalescing (only one request triggers the computation; others wait for the result).
