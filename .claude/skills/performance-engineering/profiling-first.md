# Profiling First

The discipline of measuring before optimising.

## Why profiling, not intuition

Human intuition about performance bottlenecks is wrong more often than it is right. The bottleneck is frequently not in the code that feels slow — it is in a small hot path that runs thousands of times, or in I/O that is invisible in code but dominant in wall-clock time.

A profiler shows where time is actually spent. Without it, you are optimising based on a guess.

## What to measure

**CPU profiling** — which functions consume the most CPU time. Useful for compute-bound work. Tools: `cProfile` (Python), `perf` (Linux), Xcode Instruments, Chrome DevTools profiler.

**Memory profiling** — allocation patterns, peak usage, garbage collection pressure. High allocation rates in hot paths cause GC pauses. Tools: `memory_profiler` (Python), heap profilers in JVM/Go/Node.

**I/O profiling** — time spent waiting for disk, network, or database. Often the dominant cost in web services. Measure at the database level (slow query log, EXPLAIN ANALYZE) and at the service level (distributed tracing).

**Wall-clock vs. CPU time** — wall-clock time includes I/O waits; CPU time does not. If wall-clock >> CPU time, the bottleneck is I/O, not computation.

## Reading profiling output

**Hot paths** — functions that appear frequently in the call stack or consume a disproportionate percentage of total time. These are the candidates for optimisation.

**Ask for each hot path:**
- Is this function actually in the critical path for the user-facing operation?
- How much of the total time does it represent?
- What is calling it, and is the call frequency expected or surprising?
- Would optimising it move the needle on the user-facing metric, or is the bottleneck elsewhere?

**Flat profile vs. call graph** — a flat profile shows total time per function; a call graph shows how functions relate and where time accumulates. Use both. A function that takes 2% of CPU time but is called by a function that takes 60% may be the real problem.

## The 80/20 rule in performance

Typically 80% of execution time is spent in 20% of the code. Find that 20%. Optimising anything else has diminishing returns.

## When to stop

Performance work has diminishing returns. Once the bottleneck is addressed, the next bottleneck is usually harder to fix and provides less improvement. Stop when:
- The measured metric meets the actual requirement (not an arbitrary goal)
- The marginal improvement no longer justifies the complexity cost
- The bottleneck has shifted to something outside your control (third-party API, client-side rendering)
