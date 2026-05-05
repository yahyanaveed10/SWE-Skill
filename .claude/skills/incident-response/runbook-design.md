# Runbook Design

A runbook is a document for a tired engineer at 3am. It exists because the system is on fire and the engineer needs to do the right thing fast, with imperfect memory of how this service works.

A bad runbook is worse than no runbook. It directs the engineer to take an action that no longer works (because the system changed) or omits the critical step (because nobody updated it).

---

## What a runbook is for

**One specific failure scenario.** Not a general guide to a service. The runbook for "API returning 500s" is different from the runbook for "queue depth exceeding threshold." If the same runbook is being used for many different problems, it is not a runbook — it is documentation.

**A specific person at a specific moment of stress.** The on-call engineer at 3am, who may not work on this service day-to-day, has just been paged, and needs to do the right thing in the next 10 minutes.

The runbook does not need to teach the system. It needs to enable correct action.

---

## What every useful runbook contains

**The alert that triggers it.** The exact alert name or signal. So an engineer paged by alert X can find the runbook for X without ambiguity.

**A one-sentence statement of what is happening.** "Database write replica has fallen behind the primary by more than 30 seconds." Plain language.

**Impact assessment.** What is broken for users. What is degraded but functional. What is unaffected. Helps the engineer decide whether this is a SEV1 or SEV3 in 10 seconds.

**The first three things to check.** Specific dashboards, log queries, or commands. Not "check the logs" — the URL of the dashboard or the exact query string.

**The mitigation actions, in order of preference.** Each action with the exact command or click-path. The first one to try, what to expect when it works, what to try if it does not.

**Escalation path.** When to escalate, to whom, with what information.

**Links to related runbooks.** This problem may unblock a different problem; the next runbook may need to be opened next.

---

## The hard rules

**Commands must be copy-pasteable.** Not "restart the affected pods" — `kubectl -n production rollout restart deployment/api`. The engineer at 3am should not have to translate prose into commands.

**Every command must be explained.** Why this command, what it does, what to expect. A sequence of opaque commands trains engineers to copy-paste without thinking, which is how production gets damaged during incident response.

**Destructive commands need explicit warnings.** Commands that delete data, drop connections, or change shared state get a `DANGER:` callout with the conditions under which they are appropriate. Better: include a verification step before the destructive command (`run X and verify result Y; only if Y is true, run Z`).

**Last-updated date is visible.** A runbook last updated 2 years ago is suspect. The reader should know to verify before acting if the runbook is old.

---

## How runbooks rot

**The system changes; the runbook does not.** A deploy renames a service. The runbook still references the old name. The engineer at 3am cannot find the dashboard.

**Prevention:** runbook updates are part of the engineering work that changes the system. A PR that renames a service includes the runbook update.

**Commands work in dev/staging but not in production.** The runbook was tested in dev with looser permissions. The on-call engineer in production gets a permission denied. Either fix the permissions or change the runbook to reflect the actual production command.

**Prevention:** runbooks are tested in production-equivalent conditions before being trusted. A "GameDay" or "fire drill" runs the runbook against a real (or carefully simulated) failure to verify it works.

**The runbook is for a problem that no longer happens.** The system was redesigned to prevent the failure mode. The runbook still exists. New engineers waste time reading it during incidents.

**Prevention:** when a class of failure is eliminated by design, retire the corresponding runbook.

---

## What runbooks should NOT contain

**Background on how the service works.** That is documentation, not a runbook. Link to it; do not embed it.

**The team's policies on coding style or process.** Off-topic.

**Aspirational instructions ("we should add monitoring for this").** Either it exists or it does not. Aspirational content makes the runbook a wishlist, not a guide.

**The engineer's interpretation of why this happens.** Speculation belongs in a postmortem, not a runbook. Runbooks describe *what to do*, not *why the system is bad*.

---

## Runbook discovery

A runbook nobody can find is no runbook at all.

**Link the runbook from the alert that triggers it.** When the alert fires, the page or notification should include the runbook URL. Engineers should not have to search.

**Standardise the location.** All runbooks for service X live in one well-known place. Not "some in the wiki, some in Confluence, some in this Slack channel."

**The runbook URL goes on the dashboard for the relevant signal.** Engineer investigating a latency spike on the dashboard sees the runbook for "latency exceeded threshold" linked next to the chart.
