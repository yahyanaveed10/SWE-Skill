# Triage and Mitigation

The judgment-heavy part of incident response. The decisions made in the first 15 minutes determine whether an incident lasts 30 minutes or 5 hours.

---

## Severity classification

Severity is a function of **scope of user impact** and **whether the system is degrading**. Not of how interesting the bug is.

A useful three-level scheme:

**SEV1 — Critical.** Significant user impact right now: a tier-1 service is down, a security breach is in progress, data is being lost or corrupted. Wake people up. All hands on deck. Communicate externally.

**SEV2 — Major.** Substantial degradation but the service is partially functional, or impact is limited to a subset of users. Engage the on-call team immediately, but routine business hours response may be acceptable depending on time of day.

**SEV3 — Minor.** A failure with limited impact, or a problem trending toward an SEV2 if not addressed. Track and fix during normal hours.

**Heuristic:** if you are debating between two severities, pick the higher one. The cost of over-classifying is some unnecessary attention. The cost of under-classifying is users being affected longer than they should be.

**Re-classify as you learn.** Initial severity is a guess. As scope becomes clearer, upgrade or downgrade explicitly and announce the change.

---

## Establish roles immediately

In any incident larger than one person can handle alone, establish roles in the first five minutes:

- **Incident Commander (IC)** — runs the incident. Makes decisions on what to try next. Does NOT do the technical investigation themselves; they coordinate and decide.
- **Operations / Tech Lead** — does the actual technical work. Reports to the IC.
- **Communications Lead** — handles updates to stakeholders, status pages, customer support. Frees the IC and Ops from interruption.
- **Scribe** — records the timeline of what was tried, what was observed, what was decided. The scribe's notes become the foundation of the postmortem.

The IC is the most important role and the easiest to skip. Without an IC, multiple engineers act independently — sometimes contradicting each other, sometimes duplicating work, often missing important signals because no one is looking at the whole picture.

The IC does not need to be the most senior engineer. They need to be the person willing to ask "what are we doing right now and why?" every five minutes.

---

## The mitigation decision tree

When user impact is happening, work through these in order. Stop at the first that applies.

1. **Is this caused by a recent deploy?** → Roll back the deploy. Do not investigate first.
2. **Is this caused by a recent config or feature flag change?** → Revert the config or disable the flag.
3. **Is this caused by a specific dependency being unhealthy?** → If a circuit breaker or fallback exists, force it open. If not, route traffic away from the dependency or shed load against it.
4. **Is this caused by overload (traffic spike)?** → Shed load (rate limit, return 503 for non-critical paths), scale up if scaling is fast enough, or throttle the source of the spike.
5. **None of the above clear?** → Begin focused diagnosis with the IC tracking time. Set a time-box (e.g. 15 minutes) — if no clear root cause emerges, escalate or try a less surgical mitigation (broader rollback, partial outage announcement).

The order matters. Most production incidents are caused by the most recent change. Investigating before reverting wastes time.

---

## Rollback judgment

Rollback is the default mitigation for deploy-caused incidents. There are exactly two cases where rollback is unsafe:

**Database migration ran during the deploy.** Rolling back the application code without rolling back the migration may leave the new code reading old schema or vice versa. If migrations are expand/contract (see data-and-api-design), rollback of the application code is usually safe — the new schema is backward-compatible by design. If migrations are not expand/contract, rollback may require a forward-fix instead.

**Data has been written that the old version cannot read.** New code may have written records in a new format that the old code does not understand. Rolling back will cause errors when the old code encounters those records.

Outside these two cases, **rollback is safer than fix-forward** during an active incident. The new code has already proven it has a problem; the old code was working until 20 minutes ago.

---

## Communication during the incident

**Update stakeholders on a fixed cadence — every 15 or 30 minutes — even if there is no progress.** "Still investigating, no new information" is more useful than silence. Silence prompts repeated questions to engineers who are trying to work.

**Status page updates should be in plain language and oriented to user impact.** Not "ECS task replacement is in progress" — "Some users may experience errors when checking out. We are working to restore service. Next update at 14:30."

**Internal incident channel discipline:** one designated channel per incident. No side conversations in DMs (they fragment context). The scribe keeps a running timeline pinned at the top.

**Distinguish facts from speculation in the channel.** "Database connection count is at 5000 of 5000" is a fact. "I think the connection pool is leaking" is speculation. Mixing them creates confusion later when reading the timeline.

---

## Time-boxing and escalation

Set time-boxes on diagnosis attempts. "We will try X for 15 minutes; if it does not work, we move to Y" is much better than "we are trying X" indefinitely.

Escalate when:
- The incident has been going for longer than expected without a clear path to mitigation
- The blast radius is growing
- A decision is needed that exceeds the IC's authority (taking down a region, contacting customers individually, paying for emergency capacity)

Escalation is not a failure. Refusing to escalate because "I should be able to handle this" is.
