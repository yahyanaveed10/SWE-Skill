# Complexity Signals

Common performance problems that appear in code. Each is a signal — confirm it by measuring, then address it.

---

## O(n²) patterns in disguise

**Signal:** A loop contains another loop (or a call that performs a loop) over the same or a related collection. As the collection grows, performance degrades quadratically.

**Common forms:**
- Nested loops over the same list
- A loop that calls a function that itself queries a database (see N+1 below)
- String concatenation in a loop (each concatenation copies the entire string)
- Sorting inside a loop that runs on every element

**Ask:** What is the size of the input this runs on? What happens when that doubles? If the answer is "it takes four times as long", it is quadratic.

**Direction:** Often solvable by changing the data structure (set/map for O(1) lookup instead of list search), sorting once outside the loop, or restructuring to a single pass.

---

## N+1 queries

**Signal:** One database query retrieves N records, and then N additional queries are issued — one per record — to load associated data.

**Common form:** Load a list of orders, then for each order load the customer. N orders = N+1 database round trips.

**How to spot it:** Enable database query logging and look for many identical or parameterised queries with different IDs in a single request. Distributed tracing shows many sequential spans to the database.

**Direction:** Eager loading (JOIN or `include`/`preload` in ORM), batch loading (single query for all associated records), or application-level caching of associations. The right choice depends on the access pattern.

---

## Lock contention

**Signal:** High CPU but low throughput under concurrency. Threads or goroutines are spending time waiting for each other rather than doing work. Profile shows time in synchronisation primitives.

**Common forms:**
- A global lock held for longer than necessary
- A single connection to an external resource shared by many concurrent workers
- Writes on a heavily-read data structure that could be copy-on-write

**Ask:** How long is the lock held? What work is done inside the critical section? Can the critical section be shortened, or can the lock be replaced with a finer-grained or lock-free approach?

---

## Unnecessary serialisation and deserialisation

**Signal:** Objects are repeatedly serialised to JSON/XML/binary and deserialised within the same process, or large objects are serialised when only a subset is needed.

**Ask:** Is this serialisation crossing a process boundary, or is it within the same process? If within the same process, can the object be passed directly? Is the full object needed or only a few fields?

---

## Chatty interfaces

**Signal:** An operation requires many small calls to a remote service rather than one call that returns everything needed. Common in poorly designed RPC or REST APIs.

**Example:** Fetching a user profile requires 5 separate API calls to assemble all the data.

**Ask:** What is the minimum information needed to complete this operation? Can the interface be redesigned to return it in one call? If you cannot change the interface, can you cache partial results?

---

## Unbounded growth

**Signal:** A collection, cache, or queue grows without a bound, leading to memory pressure or increased processing time as the system runs longer.

**Ask:** What is the lifecycle of items in this collection? When are they removed? Is there a maximum size, and what happens when it is reached?
