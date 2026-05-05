# Production Debugging

Debugging in production is constrained: you cannot pause the system to inspect it, you cannot add print statements without a deploy, you cannot reproduce locally. The methodology is the same — observe, hypothesise, test — but the tools are different.

---

## The hard rule of production debugging

**Read-only by default.** Investigation in production must not change production state. No experimental queries that modify data. No restarts to "see what happens." No code changes deployed without going through the normal review process.

The cost of a bad investigation step in production is borne by users. Investigation must be safer than the bug itself.

---

## What you have to work with

**Structured logs (with cardinality).** If observability is well-designed (see the observability skill), logs carry request IDs, user IDs, and other high-cardinality fields. You can find every log entry related to a specific failed request, then walk the request's path through the system.

**Distributed traces.** A trace shows the full path of a request across services, with timing at each hop. The slow span or the errored span is the place to look. Without traces, correlating "user X complained at 14:32" to "the cache layer was slow" requires guessing.

**Metrics dashboards.** Aggregates over time. Useful for: when did this start? how widespread is it? what other metrics changed at the same time?

**Database query logs / slow query logs.** Often the cause of "the system is slow" is a query that started returning more rows than the engineer planned for, or running on data that exceeded the index.

**Profilers (sampling, low-overhead).** Production-safe profilers sample stack traces periodically without significantly impacting performance. They can show "where is the CPU going right now?" without restarting the service.

**Feature flags / debug flags.** Some teams build in the ability to turn on more verbose logging for a specific user or request. Useful for reproducing behaviour for one customer.

---

## What you cannot do

**Pause execution.** No breakpoints in production.

**Add print statements without a deploy.** And a deploy adds delay (and noise — every change is now mixed in with the debugging change).

**Reproduce by trying things.** Production users are not your test fixtures. You cannot trigger the bug to see what happens.

**Trust your local intuition.** Production is different from local. The bug exists because some assumption you made does not hold in production. Verify, do not assume.

---

## The investigation playbook

When investigating a production issue:

**1. Frame the question precisely.**
"The checkout flow is broken" is not a question you can investigate. "Users are getting 500 errors when posting to /api/checkout starting at 14:00 UTC, mostly from Europe" is. Sharpen the question before searching.

**2. Find the entry point.**
What is the first signal that the bug started? An alert? A user report? A change in a metric? When? What happened around that time? (Deploy? Config change? Traffic spike? External event?)

**3. Walk the request path.**
For an API failure, find one failed request via its trace ID. Walk through the spans. Find the span that errored or was anomalously slow. That span's service and operation are where to investigate.

**4. Confirm the scope.**
Is this affecting all users or some? All requests or some? All endpoints or one? The pattern of who-is-affected often points at the cause.

**5. Hypothesise based on what changed.**
What was deployed recently? What configuration changed? What dependency changed? Most production issues are caused by a recent change. Investigate recent changes before investigating ancient code.

**6. If a hypothesis emerges, test safely.**
"I think the cache is returning stale data" — verify by reading the cache directly with a known key, comparing to the source of truth. Do not modify the cache to test.

---

## When to escalate from "investigate" to "mitigate"

Investigation can take time. While investigation is ongoing, **users are still being affected**.

If the bug is causing user impact, the goal is mitigation first (see the incident-response skill), investigation second. Mitigation strategies in approximate order of preference:
1. Roll back the deploy that introduced the bug
2. Disable the feature with a feature flag
3. Route traffic away from the broken path
4. Throttle or rate-limit the trigger condition

Investigation continues, but mitigation has stopped the bleeding. Do not delay mitigation to learn the root cause.

---

## Production debugging that requires writes

Sometimes you genuinely need to change production state to investigate (e.g., reset a stuck record, manually retry a failed job). The discipline:

**Document the action before taking it.** Write down what you are about to do and why, in the incident channel. This creates an audit trail and gives others a chance to object before damage is done.

**Use the smallest possible scope.** "Reset this one record" rather than "reset all stuck records." If the action is wrong, the blast radius is small.

**Verify before and after.** Capture the state before the change. Take the action. Capture the state after. Verify the change had the intended effect and nothing else.

**Use a tool that records the action.** Don't run ad-hoc SQL in a production console with no record. Use a tool that logs the query, the user, and the result.

---

## When the system is too distributed to debug locally

Modern systems often span 10+ services. The bug is in the interaction, not in any single service. Debugging strategies:

**Trust the trace.** A single user-request trace shows the path. If service A calls B calls C and the failure is at C, the cause is somewhere in C or in what A and B sent to C.

**Look at the request payloads.** If C errored, what did B send to C? Was it well-formed? Did it match the contract? Often the cause is "B sent something C didn't expect" — a serialization difference, a schema mismatch, a missing field.

**Use synthetic transactions.** Run a known-good test transaction through the system periodically. When it starts failing, you have a controlled reproduction without waiting for a user complaint.

**Reproduce locally with the request payload.** Capture the failing request. Replay it in a local environment with the same code version. If the bug reproduces, you now have a local reproduction. If not, the cause is environmental — investigate what is different.
