# Partial Failure in Distributed Calls

The failure mode that does not exist in single-process systems: a distributed call can fail after the remote system has processed the request but before the response is received. The operation happened; the caller does not know.

---

## The partial failure problem

In a single process: if a function call fails, nothing happened or something happened — you can tell from the exception. In a distributed system: if a network call fails, the remote may have processed the request and you received a timeout. The operation may have happened or may not have. You cannot tell from the error alone.

This is why idempotency is not optional for distributed write operations — see data-and-api-design for idempotency keys.

---

## Multi-step distributed operations

When an operation requires multiple calls across services (create an order, charge the payment, send the confirmation email), failure in any step leaves the system in a partial state. The question is not whether this can happen — it will. The question is: what is the recovery path?

**Ask for every multi-step operation:**
- What state is the system in if step 2 fails after step 1 succeeds?
- Can step 1 be rolled back? (Reversal / compensating transaction)
- Can step 2 be retried safely? (Idempotency)
- Can the partial state be detected and recovered by a background process? (Eventually consistent)

---

## The Saga pattern

A saga is a sequence of local transactions where each step publishes an event or message that triggers the next step. If a step fails, compensating transactions undo the work of preceding steps.

**When to use:** Long-running business processes across services that require coordination, where distributed transactions (2PC) are too expensive or not supported by all participants.

**When not to use:** Simple operations where the partial failure risk is low and a retry is sufficient. Sagas add significant complexity — event ordering, compensating transaction design, failure recovery. Do not introduce them speculatively.

**The compensating transaction constraint:** not all operations are reversible. A payment capture can often be voided. A confirmation email cannot be unsent. Design compensating transactions for the operations that matter; for irreversible operations, accept that they may complete even when the saga rolls back, and handle this in business logic.

---

## Exactly-once vs at-least-once delivery

Message queues and event systems typically guarantee at-least-once delivery — a message may be delivered more than once if an acknowledgement is lost. They do not guarantee exactly-once delivery.

**Implication:** consumers of queues and event streams must handle duplicate messages. The consumer must be idempotent — processing the same message twice must produce the same result as processing it once.

**Common duplicate handling approaches:**
- Check whether the operation has already been performed before processing (deduplication key lookup)
- Design the operation itself to be naturally idempotent (set a field to a specific value rather than incrementing it)
- Use the message ID as an idempotency key and record processed message IDs

**The wrong assumption:** assuming a queue delivers each message exactly once. This assumption will fail under network partitions, consumer restarts, and normal operational conditions.

---

## Timeouts and consistency

A timeout does not mean the operation failed. It means the caller stopped waiting. The operation may have succeeded, failed, or still be in progress on the remote system.

After a timeout on a write operation, the caller is in an unknown state. Options:
1. Retry with idempotency — if the operation is idempotent, retry safely
2. Query to determine the outcome — if the operation leaves detectable state, query before deciding
3. Accept ambiguity — for low-stakes operations, accept that the outcome is unknown and let a reconciliation process detect inconsistency later

Never silently proceed as if the operation failed — it may have succeeded, and ignoring a successful write creates inconsistency.
