---
name: concurrency
description: Concurrency correctness — race conditions, shared state safety, and async patterns. Use when writing multi-threaded code, async/await code, background workers, or any code where multiple operations may execute simultaneously; when reviewing code for thread safety; when a system behaves differently under load than in single-user tests; or when debugging non-deterministic failures. Covers shared state signals, atomic operations, async correctness, and why concurrency bugs don't show in sequential unit tests.
---

# Concurrency

Concurrency bugs are the hardest class of bugs to find and reproduce. They do not appear in sequential unit tests, they are non-deterministic (a race condition may trigger once in a million runs), and they often manifest as data corruption rather than crashes.

Agents write async code and multi-threaded code without reasoning about what runs simultaneously. The result passes all tests and fails in production under real concurrent load.

For shared state and race condition signals see [shared-state-signals.md](shared-state-signals.md).
For async/await correctness signals see [async-patterns.md](async-patterns.md).
For why concurrency bugs are invisible in unit tests see [concurrency-testing-gap.md](concurrency-testing-gap.md).

## The core question

For any piece of code that will run in a concurrent context: **what happens if two instances of this code run simultaneously with the same or overlapping data?**

If the answer is "they might interfere" — you have a concurrency problem. If the answer is "they cannot interfere because X" — X is a design decision that must be deliberate, not accidental.

## The safest defaults

**Immutability first.** Data that cannot be modified cannot have race conditions. Prefer immutable data structures. Copy data before modifying it. Return new values rather than mutating in place.

**Share nothing where possible.** Code that does not share state cannot have races on that state. Prefer passing data through function arguments over accessing shared variables.

**Confine mutation to a single actor.** If mutable state is unavoidable, confine writes to a single goroutine, thread, or actor. Other components send requests to that actor rather than accessing the state directly. The actor processes requests serially — no locking needed.

When these defaults are not possible, explicit synchronisation is required.
