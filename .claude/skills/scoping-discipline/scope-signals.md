# Scope Signals

Signals that scope is wrong — either too much for the current stage or too little for the risks being taken.

---

## Over-scoped PoC signals

**The PoC takes longer than a week.** A PoC that takes more than a week is probably an MVP in disguise. Ask: what is the hypothesis being validated? If it requires a full auth system and a production database, the question is not "can this idea work?" — it is "can we build the full system?"

**The PoC has tests.** Tests on a PoC are a signal that someone is planning to keep this code. Acknowledge that and plan accordingly — or explicitly discard the code after the hypothesis is validated.

**The PoC has a proper data model.** If the schema is being designed carefully, the team is building an MVP without calling it one.

**The team argues about which design patterns to use.** A PoC does not need design patterns. It needs enough code to answer a question.

---

## Under-scoped MVP signals

**"We'll add error handling later."** Error handling for external calls is not optional at MVP. A third-party API failure, a database connection error, or a timeout will happen in production. Without handling, it crashes the user's flow.

**No request timeouts.** Every outbound call — HTTP requests, database queries, external API calls — must have a timeout. Without one, a slow dependency hangs the calling thread indefinitely. At MVP scale, this produces cascading failures under light load.

**Secrets hardcoded in source.** Hardcoded secrets are a security incident waiting to happen. At MVP, secrets in environment variables is the minimum. Secret rotation is fine to defer.

**No idempotency on payment or state-changing operations.** A user who clicks "submit" twice creates two records. A mobile client that retries a failed request creates two records. At MVP with real users and real data, this is a data integrity problem.

**"We'll add logging when something breaks."** You will not know something broke until a user tells you, and you will not be able to diagnose it without the logs. Basic structured logging with request IDs is not optional at MVP.

---

## The PoC promotion trap

The most expensive scoping mistake: a PoC that "works" gets promoted to production without an engineering pass because:
- The demo was convincing
- The deadline is tight
- "We'll clean it up later" (later does not come)
- Nobody wants to be the person who says the working code needs to be rewritten

**Signals a PoC is being treated as production:**
- Users are accessing it without the team being able to see failures (no monitoring)
- The team is afraid to change it because they don't know what will break (no tests, no understanding of the failure modes)
- Performance or reliability problems appear under real load that were invisible in demos
- The on-call team does not know how to restart or recover the system

**The honest conversation:** "This is a PoC that got promoted. Before we add more features, we need an engineering pass. Here are the specific things that are missing: [list]. Here is the cost of skipping them: [specific failure scenarios]." That conversation is uncomfortable. The production incident it prevents is more uncomfortable.

---

## Right-sizing for the next stage

When scoping work for the next stage, use the cost-of-failure frame:

1. List the concerns that are currently skipped (from the promotion checklist)
2. For each: if this fails in production, what is the impact? (data loss, user-visible error, security incident, silent data corruption)
3. Scope the ones with unacceptable failure impact for this stage
4. Explicitly document the ones being deferred and why

This produces a written record of the trade-offs, not an implicit assumption that "we'll get to it."
