# SLO Design and Error Budget Management

The discipline of defining what reliable service means for a specific system, measuring against that target, and using the gap as a release control. From Google SRE Book chapters 4-5.

---

## SLI, SLO, SLA — three different things

**SLI (Service Level Indicator):** the metric you measure. "Fraction of HTTP requests with status < 500" or "p99 latency of /checkout endpoint."

**SLO (Service Level Objective):** the target value for the SLI over a time window. "99.9% of HTTP requests succeed, measured over 28 days."

**SLA (Service Level Agreement):** the contractual commitment to customers, with consequences for breach. SLAs are usually less strict than internal SLOs (you should breach the SLO and notice before you breach the SLA).

Engineers operate against SLOs. Customers see SLAs. SLIs are the underlying measurements that inform both.

---

## Defining SLIs from user pain

The wrong way to pick an SLI: copy what other teams measure. The right way: identify the user journey and what failure looks like for it.

**For a request-driven service:** what does "the request worked" mean for the user? For an API, that is usually: returned a non-error response within an acceptable time. For a streaming service: stream did not buffer beyond a threshold. For a batch job: completed within its window.

**Each SLI captures one aspect of one user journey.** A single service may have multiple SLIs — request availability, request latency, data freshness — because users care about different things on different paths.

**SLI design checklist:**
- Is the SLI measured from the user's perspective? (A backend service with 100% uptime that has 50% error rate from the gateway is not 100% available to users.)
- Is the threshold meaningful to the user? ("Latency under 500ms" — does the user notice the difference between 400ms and 500ms? Pick the threshold based on the user, not on what the system can achieve today.)
- Is the SLI bounded? (A ratio between 0 and 1, or a percentile of a measured value. Not a raw count.)

---

## Picking an SLO target

The SLO target is a number between "users notice failure" and "perfect, which is impossible." Two common mistakes:

**Setting it too high.** A target of 99.99% (52 minutes of downtime per year) sounds good. But achieving it requires expensive engineering — multi-region deployment, careful rollouts, robust automation. If users would be fine with 99.5% (43 hours per year), spending engineering time chasing 99.99% is overengineering.

**Setting it too low.** A target of 99% (3.65 days per year) is easy to achieve but probably not what users would tolerate for a payment system or a critical workflow.

**Heuristic:** what error rate would cause a meaningful fraction of users to complain or churn? Set the SLO at or just better than that threshold. Then verify the system can sustainably hit it; if not, either invest to improve, or lower the SLO and tell stakeholders.

**The 9s ladder:**
- 99% — 3.65 days/year unavailable. Internal tools, non-critical features.
- 99.9% — 8.76 hours/year. Most consumer-facing services.
- 99.95% — 4.38 hours/year. Important services, paid products.
- 99.99% — 52 minutes/year. High-stakes services (payments, identity, critical infrastructure).
- 99.999% — 5.26 minutes/year. Rare; usually only achieved by tier-1 cloud providers and requires exceptional engineering.

---

## Error budgets

If the SLO is 99.9% over 28 days, the error budget is the allowable 0.1% of "bad" events — about 40 minutes of downtime in 28 days, or about 0.1% of requests allowed to fail.

The error budget is the credit you spend on:
- Risky deploys (a deploy that breaks 1% of requests for 5 minutes spends some budget)
- Planned maintenance
- Experiments and feature rollouts
- Operational mistakes

**The agreement that matters:** when the error budget is exhausted, risky changes stop until the budget recovers. This converts SLO from documentation into a release control.

**Why this works:** without an error budget, "more reliability" and "ship faster" are in permanent tension. With it, the team has a quantitative way to decide: budget intact → ship freely; budget low → slow down; budget exhausted → freeze risky changes until the next window.

---

## Burn rate alerting

Static threshold alerting fires when current error rate is "too high right now" — but does not distinguish between a brief blip and a sustained problem. Burn rate alerting solves this.

**Burn rate** = how fast the error budget is being consumed relative to sustainable.

If your SLO is 99.9% and you are seeing 1% errors, your burn rate is 10x sustainable — you will exhaust 30 days of error budget in 3 days.

**Multi-window multi-burn-rate alerting** (Google SRE recommended pattern):

| Page urgency | Burn rate | Time window | Why |
|---|---|---|---|
| Page now (severe) | 14.4x | 1 hour | At this rate, you exhaust a 30-day budget in 2 days. Wake someone up. |
| Page now (moderate) | 6x | 6 hours | At this rate, you exhaust the budget in 5 days. Investigate now. |
| Ticket (slow burn) | 1x | 3 days | Sustained slow degradation. Investigate during business hours. |

The two-window check (a short window AND a longer window both above threshold) prevents alerting on a 5-minute spike that would otherwise look severe.

**Why this is better than threshold alerting:** a 1% error rate for 2 minutes is probably noise. A 1% error rate for 4 hours is consuming budget at a rate that warrants attention. Burn rate distinguishes between them; static thresholds do not.

---

## Using the error budget as a release gate

The structural integration of SLO into engineering process:

**Budget healthy (>50% remaining):** ship freely. Take risks. Try experiments.

**Budget consumed faster than sustainable burn rate:** investigate, slow down risky changes.

**Budget exhausted:** freeze risky changes (anything that could spend more budget). Allow only changes that recover reliability or fix the underlying issue. The freeze ends when the budget recovers (rolling window) or at the next budget reset.

**Why teams resist this:** "we have to ship X by deadline." But the SLO and budget were agreed in advance. If shipping X requires spending budget you do not have, the choice is between shipping X with reduced reliability (and accepting the SLO miss) or delaying X. Both are explicit decisions, not silent erosion of reliability.

---

## Common SLO mistakes

**Measuring availability that has nothing to do with users.** A 99.9% SLO on "service uptime" may be 100% green while the service is silently returning wrong results to half its users. Measure user outcomes, not service health.

**SLOs that are easier to measure than to achieve.** If the SLI is the average response time and the average is 200ms but the p99 is 5000ms, half your users are getting a great experience and a meaningful fraction is getting a terrible one — but the SLO looks fine. Use percentiles (p95, p99) for latency SLOs, not averages.

**One SLO per service.** Different user journeys have different reliability needs. Checkout and search may be in the same service but require different SLOs. Define multiple SLOs when the user journeys are distinct.

**SLOs nobody acts on.** An SLO that is missed every week and nobody changes anything is theatre. Either commit to the SLO (act when it is missed), or lower it to a level you actually defend.
