---
name: incident-response
description: Engineering discipline for production incidents — during and after. Use when a production system is broken or degraded, when triaging a page or alert, when deciding whether to roll back vs. fix forward, when running an incident as commander or responder, when writing or reviewing a postmortem, or when designing a runbook. Covers severity triage, mitigation vs. root-fix decisions, war-room coordination, blameless postmortem structure, and runbook design. Does not cover detection (see observability) or design-time failure prevention (see resilience-engineering).
---

# Incident Response

Observability tells you something is wrong. Resilience prevents some failures from happening in the first place. Incident response is what you do when something is wrong *right now* — and what you learn from it after.

The discipline is distinct from the design-time discipline of resilience engineering. During an incident, the question is never "what is the cleanest fix" — it is "what stops user impact fastest with the least new risk."

For triage and during-incident decisions see [triage-and-mitigation.md](triage-and-mitigation.md).
For postmortem structure and culture see [postmortem-discipline.md](postmortem-discipline.md).
For designing runbooks that are useful at 3am see [runbook-design.md](runbook-design.md).

## The core distinction: mitigate vs. fix

**Mitigation stops user impact.** Roll back the bad release. Disable the feature flag. Shift traffic to a healthy region. Throttle the source of load. None of these fix the underlying problem — they buy time to fix it without users continuing to be affected.

**Root-fixing addresses the cause.** This belongs after mitigation, not during. Engineers who try to root-cause in production while users are still suffering extend the incident by hours.

The default during an active incident: **mitigate first, fix after.** Reverse this only when you have strong evidence that mitigation will not work or will make things worse (e.g. rolling back through a database migration that already partially ran).

## What an incident is

An incident is any unplanned event that degrades or threatens to degrade a service against its SLO. Not every error is an incident. Not every alert is an incident. The signal is **user-visible impact or imminent risk of it**, sustained beyond a brief blip.

If you are unsure whether something is an incident, treat it as one and downgrade if it isn't. The cost of declaring an incident that turns out to be nothing is small. The cost of not declaring one and having users affected for an extra hour is large.

## The phases

1. **Detect** — observability signals or user reports indicate something is wrong. (Detection is the observability skill, not this one.)
2. **Triage** — classify severity, identify scope of impact, mobilise responders.
3. **Mitigate** — stop user impact through whatever means necessary, even ugly ones.
4. **Diagnose** — find root cause (in parallel with mitigation if possible, after if not).
5. **Fix** — address the root cause properly, in non-emergency mode.
6. **Learn** — postmortem to extract what should change so this class of incident is less likely or has less impact next time.

The phases are not strictly sequential. Diagnose often runs in parallel with mitigate. But mitigate should never wait for diagnose to complete.

## What this skill does not cover

- The tooling for paging and alerting (PagerDuty, Opsgenie configuration) — tooling-specific
- Detection itself — see `observability`
- Designing systems that fail gracefully — see `resilience-engineering`
- The organisational/HR side of on-call rotations and burnout
