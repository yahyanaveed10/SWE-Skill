# PoC vs MVP vs Production

What each stage actually requires — and what is explicitly safe to skip.

---

## Proof of Concept (PoC)

**Purpose:** Does the idea work at all? Is the technical approach feasible?

**Audience:** Internal — the team or a stakeholder. Not real users.

**Safe to skip:**
- Authentication and authorisation
- Error handling beyond what is needed to demonstrate the happy path
- Automated tests
- Idempotency and retry logic
- Connection pooling, timeouts, circuit breakers
- Migration strategy — hardcode if needed
- Monitoring and logging
- Security hardening
- Pagination and performance at scale

**Must have:**
- Enough of the core behaviour to validate the hypothesis
- Clear documentation of what is hardcoded, faked, or skipped
- An explicit decision record: did the PoC validate the hypothesis? Yes/no/partially?

**The PoC trap:** The PoC works and looks good in the demo, so it gets promoted directly to production without an engineering pass. See `scope-signals.md`.

---

## Minimum Viable Product (MVP)

**Purpose:** Does this work for real users under real conditions?

**Audience:** A limited set of real users. Real data. Real failure modes.

**Must have (not optional at MVP):**
- Authentication and basic authorisation — real users means access control
- Error handling on all external calls — a third-party failure cannot crash the whole flow
- Request timeouts — every outbound call must have a timeout
- Basic structured logging — at minimum, request IDs and error context
- Database connection pooling
- Secrets in environment variables (not hardcoded) — even if rotation is not automated yet
- A schema migration strategy — even if manual for now
- Idempotency on write operations that users can trigger more than once
- Basic smoke tests — enough to catch a completely broken deployment before it reaches users

**Safe to defer to production:**
- Full observability (traces, metrics dashboards, alerting)
- Automated failover
- Comprehensive test coverage
- Rate limiting (unless abuse is a known risk at this scale)
- Performance optimisation beyond what is needed to serve the MVP load
- Full audit logging

**The MVP trap:** adding production-grade concerns to the MVP before validating that users want the feature at all. An MVP that takes twice as long to build because it has full observability and comprehensive tests is a delayed validation, not a safer product.

---

## Production

**Purpose:** Works reliably, securely, and maintainably for users you cannot directly monitor.

**Must have (everything from MVP plus):**
- Full observability: structured logging with trace correlation, metrics on the four golden signals, alerting on SLO-based thresholds
- Idempotency on all write operations accessible by users or external systems
- Retry logic with exponential backoff and jitter on all external calls
- Circuit breakers on calls to dependencies that can fail independently
- Rate limiting on public-facing endpoints
- Graceful degradation — what does the system serve when a dependency is down?
- Secrets rotation without redeployment
- Database migrations that can run without downtime (expand/contract pattern)
- Runbook for common failure scenarios
- Capacity estimate: what load can this handle before degrading?
- Security review: input validation, authorisation checks, sensitive data handling

**The production trap:** deploying to production before the observability is in place. The first production incident will happen without the tools to diagnose it.

---

## The promotion checklist (PoC → MVP or MVP → Production)

Before promoting to the next stage, answer each question:

| Concern | Safe to skip at PoC? | Safe to skip at MVP? | Required at production? |
|---|---|---|---|
| Authentication | Yes | No | Yes |
| Request timeouts | Yes | No | Yes |
| Idempotency on writes | Yes | No | Yes |
| Connection pooling | Yes | No | Yes |
| Structured logging | Yes | Basic | Full |
| Error handling | Minimal | External calls | Comprehensive |
| Automated tests | Yes | Smoke tests | Meaningful coverage |
| Schema migration strategy | Yes | Manual ok | Automated, zero-downtime |
| Secrets in config (not code) | Yes | Yes | Yes + rotation |
| Monitoring + alerting | Yes | Yes | SLO-based |
| Rate limiting | Yes | If abuse risk | Yes |
| Runbook | Yes | Yes | Yes |
