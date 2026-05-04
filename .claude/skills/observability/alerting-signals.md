# Alerting Signals

An alert is a claim that a human needs to take action right now. An alert that does not require action is noise — it trains the team to ignore alerts, which means the alerts that matter get ignored too.

---

## Alert on symptoms, not causes

**Alert on what users experience, not on system internals.**

- Alert on: error rate elevated, latency p99 above SLO threshold, availability below target
- Do not alert on: CPU above 70%, JVM heap above 80%, disk at 60%

Infrastructure metrics are useful for diagnosing the cause after an alert fires. They are poor alert signals because they correlate weakly with user impact: a service can be CPU-saturated and perfectly fast, or have low CPU and be returning errors.

**The SLO-based alerting model:** define what good service looks like (error rate < 0.1%, p99 latency < 500ms). Alert when current performance is consuming your error budget faster than sustainable. This focuses alerts on user-visible impact and reduces alert volume from noisy infrastructure metrics.

---

## Alert fatigue signals

**Alert fires and the team acknowledges without investigating.** The team has learned this alert does not require action. It should be either fixed or deleted.

**Alert fires multiple times before action is taken.** The alert threshold is too sensitive — it fires before the problem is actionable or before it will self-resolve.

**Alert fires at the same time as three other alerts.** Dependent alerts — multiple alerts triggered by the same root cause. Consider whether a single upstream alert is sufficient.

**Alerts that only fire at night or on weekends.** Not necessarily wrong, but review: is this a genuine production problem, or a batch job that runs off-hours and triggers a threshold set for interactive traffic?

**The on-call rotation consistently wakes up at 3am for the same alert.** Either the system has a recurring problem (fix it) or the alert threshold is wrong (tune it). Do not accept recurring 3am pages as normal.

---

## What every alert should answer

Before adding an alert, write the runbook entry for it. If you cannot answer these questions, the alert is not ready:

1. **What is happening?** (in plain language, not metric names)
2. **Who is affected?** (all users, a subset, internal only)
3. **What is the immediate action?** (restart, roll back, scale up, page the database team)
4. **What does "resolved" look like?**
5. **Is there a known false positive condition?** (a batch job, a scheduled maintenance window)

An alert without a runbook entry is an alert that will be mishandled under pressure.

---

## SLOs and error budgets

An SLO (Service Level Objective) defines the target reliability level. An error budget is the allowed deviation from perfect reliability within a time window.

**Example:** 99.9% availability over 30 days = 43 minutes of allowable downtime. If you have consumed 40 minutes in the first 20 days, you are burning your error budget 3x faster than sustainable — alert on the burn rate, not just the current availability percentage.

**Why burn-rate alerting is better than threshold alerting:** a 1% error rate for 5 minutes may be an experiment or a fluke. A 1% error rate sustained for 2 hours is consuming error budget at 10x the sustainable rate — that is worth waking someone up. Burn-rate alerts distinguish between these cases. Static threshold alerts do not.
