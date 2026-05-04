# The Concurrency Testing Gap

Concurrency bugs are nearly invisible in unit tests. Understanding why is necessary to know when and how to test for them.

---

## Why unit tests miss concurrency bugs

**Sequential execution.** Unit tests run code sequentially — one call completes before the next begins. Race conditions require simultaneous execution by multiple threads or coroutines. A race between two threads reading and writing the same variable will never trigger in a test that runs the code one call at a time.

**Single thread.** Most unit test runners use a single thread. Multi-threaded bugs require multiple threads.

**Deterministic timing.** Race conditions are timing-sensitive. The window for a race may be microseconds wide. A unit test that runs in milliseconds will miss a race that requires two operations to interleave within microseconds.

**Clean state.** Unit tests start with a clean, controlled state. Race conditions often depend on accumulated state, shared caches, or resource contention that only exists under real load.

---

## Signals that concurrency testing is needed

**The code modifies shared state.** If a function modifies a class field, a global variable, or any state accessible from multiple threads, it needs concurrency testing — not just unit testing.

**The code uses async primitives.** Async/await, goroutines, callbacks, event emitters — any of these create concurrent execution paths that unit tests do not exercise.

**Tests pass but production fails non-deterministically.** A bug that only appears sometimes, that disappears when you add logging (the logging changes the timing), or that only appears under load is almost certainly a concurrency bug.

**The code implements locking, synchronisation, or coordination.** If you added a lock or a semaphore, write a test that proves the lock is necessary and effective. A test that does not exercise the concurrent case does not test the lock.

---

## Approaches to concurrency testing

**Stress testing.** Run the concurrent code many times simultaneously and verify correctness of outcomes. A test that spawns 100 goroutines all incrementing a counter and asserts the result is 100 — if the counter is not thread-safe, the test fails non-deterministically. Run it enough times and you will see the failure.

**Controlled interleaving.** Inject sleeps or barriers at specific points to force the timing that would cause a race. This makes the race deterministic and thus testable. Requires knowing the specific race to test for.

**Property-based testing.** Generate random sequences of concurrent operations and verify invariants hold. Libraries like Hypothesis (Python), fast-check (JS), or gopter (Go) can generate these sequences.

**Static analysis.** Tools that detect data races at compile time or via instrumentation: Go's race detector (`-race`), ThreadSanitizer (C/C++/Go/Rust), Java's FindBugs/SpotBugs. These find races that tests miss by analysing access patterns, not executing the code.

---

## The practical minimum

For code that modifies shared state:

1. Run the race detector if your language/runtime provides one (`go test -race` is free and catches real races)
2. Write at least one stress test that exercises the concurrent path — multiple concurrent callers, verified correct outcome
3. Document what synchronisation is in place and why — a comment that says "this is safe because X" is a future reviewer's guide to whether a proposed change breaks the invariant
