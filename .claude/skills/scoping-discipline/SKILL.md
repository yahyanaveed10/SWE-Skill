---
name: scoping-discipline
description: PoC vs MVP vs production scoping decisions. Use when asked to 'just make it work', when estimating what is needed for a first version, when deciding what to cut from scope, when evaluating whether a prototype is ready to promote, or when a working demo is being treated as production-ready. Covers what each stage actually requires, what is safe to defer, what creates hard-to-reverse technical debt, and when to rewrite vs. promote. Does not cover product prioritisation or roadmap — engineering scope only.
---

# Scoping Discipline

The most common cause of both over-engineered PoCs and under-engineered MVPs is the same: treating all stages as equivalent. A prototype and a production system solve different problems. Applying production standards to a prototype wastes time; applying prototype standards to production creates incidents.

For what each stage requires see [poc-vs-mvp-vs-production.md](poc-vs-mvp-vs-production.md).
For signals that scope is wrong see [scope-signals.md](scope-signals.md).

## The core heuristic

For each component or concern, ask: **if this fails, can I fix it before a user notices?**

- If yes → it can be deferred to the next stage
- If no → it must be done now

This question forces honesty about failure modes and blast radius. It is not "is this important?" (everything is important) — it is "what happens if I skip it and the system runs?"

## Why promotions fail

The PoC-to-production promotion failure is the most common expensive engineering mistake. The prototype "works" in demo conditions and gets promoted without an engineering pass. The items most often skipped:

- No idempotency on write operations → duplicate submissions create duplicate data
- No request timeouts → one slow downstream call hangs the entire system
- No connection pooling → one traffic spike exhausts database connections
- Secrets hardcoded or in environment variables without rotation → cannot rotate without redeployment
- No migration strategy → the schema is whatever the first dev made it; changing it requires downtime
- No error handling on external calls → a third-party API failure crashes the whole flow

None of these are visible in a demo. All of them fail in production.

## The build-vs-promote decision

When a prototype reaches the end of its useful life as a prototype, there is a choice: promote it (engineering pass to make it production-grade) or rewrite it (start from the design decisions that the prototype validated).

**Promote when:** the architecture is sound, the failure modes are addressable without structural changes, and the volume of production-hardening work is proportional to the value of the working code.

**Rewrite when:** the architecture has load-bearing assumptions that do not hold at production scale, the data model is wrong in ways that require a migration campaign, or the prototype's shortcuts are so pervasive that reading the code requires more context than starting fresh.

The rewrite decision is uncomfortable because it discards working code. The promotion decision is comfortable but often produces a production system built on PoC assumptions. Neither is automatically right — the question is which one is cheaper over the next 12 months.
