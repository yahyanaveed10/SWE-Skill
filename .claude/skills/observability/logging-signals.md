# Logging Signals

Signals for identifying logging that will not help in a production incident — and what to do instead.

---

## What makes a log entry useful vs. noise

**Useful:** structured fields (JSON or key=value), a timestamp with millisecond precision, a severity level, a request/trace ID, and enough context to understand what was happening without reading the surrounding code.

**Noise:** `print("here")`, `console.log(data)`, `logger.info("done")`, bare exception messages without stack context or request context.

The test: if this log entry appeared in your incident dashboard at 3am, could you tell what failed, for whom, and what the system was trying to do? If not, the log entry is noise.

---

## The most common logging mistakes

**No request ID / trace ID on the entry.** You can see that an error occurred. You cannot correlate it to a user, a session, or the other log entries from the same request. Fix: inject a request ID at the entry point (API gateway, middleware) and attach it to every log entry in that request's lifecycle.

**Logging the exception message but not the context.** `Error: connection refused` tells you what failed but not where, why, or what the system was trying to do. Fix: log the operation being attempted, the target (URL, table name, service name), and the inputs that led to the failure (sanitised — no secrets or PII).

**Using print/console.log in production code.** Print statements go to stdout with no structure, no severity level, no timestamp (or system-only timestamp), and no request context. They cannot be queried. Fix: use a structured logger. The logging framework is not the important choice — the structured output is.

**Logging at the wrong severity.** `ERROR` for expected business conditions (a user not found, a duplicate submission) trains teams to ignore errors. `INFO` for security events makes them invisible. Signal:
- `DEBUG` — only useful for development; should be disabled in production by default
- `INFO` — normal operational events (request received, job started, job completed)
- `WARN` — unexpected but handled: a retry was needed, a fallback was used, a deprecated path was taken
- `ERROR` — something failed that should not have; requires attention
- `FATAL/CRITICAL` — the process cannot continue

**Logging secrets, PII, or credentials.** Passwords, API keys, session tokens, and personal data in logs create a security incident on top of whatever you were debugging. Fix: scrub or mask sensitive fields before logging. Log that a token was present and valid, not its value.

**Too much or too little.** Logging every row processed in a batch job produces gigabytes of noise. Logging nothing during a multi-step operation leaves gaps. Rule of thumb: log entry and exit of significant operations (a job start, an external API call, a database write), not internal iterations.

---

## What to log at each boundary

**Incoming request:** method, path, request ID, user/tenant ID (if authenticated), size. Not the full body unless debugging and the environment is non-production.

**Outbound call (HTTP, database, queue):** target, operation, duration, success/failure, error code if failed. Not the full payload.

**Background job / async worker:** job type, job ID, start time, end time, items processed, errors encountered.

**Authentication / authorisation events:** always. Who, what they tried to do, whether it was allowed. These are security audit events.

**State transitions:** when an entity moves from one state to another (order placed → payment pending → payment confirmed). Log the entity ID, the old state, the new state, and the triggering event.
