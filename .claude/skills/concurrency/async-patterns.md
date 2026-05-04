# Async Patterns

Correctness signals for async/await code and event-driven patterns. Async is not the same as concurrent — but async code has its own class of correctness problems that sequential unit tests do not catch.

---

## What async/await actually does

`async/await` does not create new threads. It allows the current thread to do other work while waiting for I/O. Multiple async operations can be in flight simultaneously on the same thread (in single-threaded environments like JavaScript) or across a thread pool (in Go, Python asyncio, .NET).

**The implication:** in JavaScript, you cannot have race conditions on JavaScript objects between two async functions — only one runs at a time. But the state of the system can change between `await` points, because other code runs during the await.

In Go or Python with multiple threads, async code can have full race conditions because multiple threads may execute coroutines simultaneously.

---

## Common async correctness mistakes

**Fire-and-forget without error handling.**
```javascript
// Wrong: errors are silently swallowed
doSomethingAsync();  // no await, no .catch()

// Right: either await or explicitly handle the promise
doSomethingAsync().catch(err => logger.error('failed', err));
```
An unhandled promise rejection or an unhandled goroutine panic terminates silently in many runtimes (or logs a warning that gets ignored). The operation failed; nothing knows.

**Sequential awaits that could be parallel.**
```javascript
// Wrong: 2 seconds total when 1 second is possible
const user = await fetchUser(id);        // 1 second
const prefs = await fetchPrefs(id);      // 1 second

// Right: 1 second total
const [user, prefs] = await Promise.all([fetchUser(id), fetchPrefs(id)]);
```
Awaiting independent operations sequentially is a latency bug. Use `Promise.all`, `gather`, or equivalent to run independent async operations concurrently.

**Awaiting in a loop.**
```javascript
// Wrong: processes items sequentially, one at a time
for (const item of items) {
    await processItem(item);  // each waits for the previous
}

// Right: process in parallel (with concurrency control for large lists)
await Promise.all(items.map(item => processItem(item)));
```
Awaiting inside a loop serialises what could be concurrent. For large lists, add concurrency limiting to avoid overwhelming downstream services.

**State change between await points (JavaScript-specific).**
```javascript
async function transferFunds(from, to, amount) {
    const balance = await getBalance(from);  // 100
    // other code runs here during the await
    if (balance >= amount) {
        await debit(from, amount);  // balance may have changed since check
    }
}
```
In single-threaded JS, only one async function runs at a time — but another async function can run between your `await` points. A check made before an `await` may be invalid after it. This is the async analogue of check-then-act.

**Blocking the event loop (JavaScript/Node.js).**
CPU-intensive synchronous code in a Node.js application blocks the event loop and prevents any other async operation from running. Symptoms: all requests slow down simultaneously, not just requests using the slow code path. Fix: move CPU-intensive work to a worker thread.

---

## Async in constructors

Constructors cannot be async. A class that requires async initialisation (connecting to a database, loading a config file) is a design problem — the object appears constructed but is not ready to use.

**The factory pattern fix:** expose a static async factory method that does the async work and returns a fully initialised instance. Callers cannot create instances directly.

```javascript
class DatabaseClient {
    private constructor(private connection: Connection) {}

    static async create(config: Config): Promise<DatabaseClient> {
        const connection = await connect(config);
        return new DatabaseClient(connection);
    }
}
```

---

## Context cancellation

Long-running async operations should respect cancellation signals — a user navigating away, a request timeout, a service shutdown. An async operation that cannot be cancelled holds resources indefinitely.

In Go: pass `context.Context` as the first parameter to any function that does I/O. Check `ctx.Done()` in loops.
In .NET: pass `CancellationToken` to async methods.
In JavaScript: use `AbortController` for fetch; implement cancellation manually for custom async operations.

An async operation that ignores its context continues running after the caller has given up — consuming resources for work whose result will never be used.
