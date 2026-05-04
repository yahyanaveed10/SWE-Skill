# Shared State Signals

Signals for identifying concurrency hazards in code that shares state across threads, goroutines, or async contexts.

---

## Race condition signals

**A read-modify-write sequence without synchronisation.** The most common race condition pattern. Two threads both read a value, both compute a new value, both write — one write overwrites the other.

```
# Unsafe: read-modify-write without atomicity
count = get_count()      # thread A and B both read 5
count += 1               # both compute 6
set_count(count)         # both write 6 — one increment is lost
```

**Ask:** Is this a read-modify-write sequence? Is it possible for another thread to read between the read and the write? If yes, the operation is not atomic and needs a synchronisation mechanism.

**A "check-then-act" sequence.** Check a condition, then act on the assumption it is still true. Another thread may change the condition between the check and the act.

```
# Unsafe: check-then-act
if not lock_acquired(resource):
    acquire_lock(resource)  # another thread may have acquired it between check and acquire
```

**Global or shared mutable state accessed from multiple call paths.** A singleton with mutable state, a global cache, a shared counter — any of these accessed from concurrent code requires synchronisation.

**Signal:** The state is stored outside the function (class field, module-level variable, injected shared object) and the function can be called concurrently.

---

## Atomicity and memory visibility

**Atomicity:** An operation is atomic if it completes without interruption. Incrementing a counter in most languages is not atomic — it is three operations (read, add, write) that can be interrupted between any two steps.

Use atomic primitives (AtomicInteger in Java, sync/atomic in Go, Interlocked in .NET) for simple counters and flags. Use locks for compound operations that must be atomic as a group.

**Memory visibility:** In multi-threaded programs, one thread's writes may not be visible to another thread immediately — CPUs and compilers reorder operations. A flag set by thread A may not be seen by thread B without a memory barrier.

Languages provide primitives for this: `volatile` in Java, `atomic` in C++, synchronisation in Go. The general rule: any variable written by one thread and read by another must have explicit synchronisation, not just the write operation.

---

## Deadlock signals

**Two threads, each waiting for a lock the other holds.** Thread A holds lock 1, waits for lock 2. Thread B holds lock 2, waits for lock 1. Neither can proceed.

**Signal patterns:**
- Multiple locks acquired in different orders in different parts of the code
- A lock is held while making an external call that may also acquire a lock
- Recursive locking without a reentrant lock

**Prevention:** Always acquire locks in the same order. Never hold a lock while making an external call. Prefer lock-free designs when possible.

**Starvation:** A thread that never gets scheduled because other threads always get priority. Signals: a thread exists and is supposed to be doing work, but its work never completes. Common with priority-based schedulers and long-running holders of shared resources.

---

## When synchronisation is not the answer

Synchronisation (locks, mutexes, semaphores) solves race conditions but introduces its own failure modes: deadlock, starvation, and contention that eliminates the benefit of concurrency.

Before adding synchronisation, ask:
- Can this state be made immutable?
- Can this state be confined to a single thread/actor?
- Can this operation be designed to be naturally idempotent, so concurrent execution produces the same result as sequential?

If none of these apply, synchronisation is correct. But synchronisation is the last resort, not the first tool.
