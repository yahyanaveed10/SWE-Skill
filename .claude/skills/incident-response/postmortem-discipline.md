# Postmortem Discipline

A postmortem is the engineering output of an incident. Its purpose is to extract what should change so that this class of incident is less likely or has less impact next time.

A postmortem is not: a record of who is at fault, a project status update, or a justification of what was done well. It is a structured record of what happened and what changes the system or process needs.

---

## Blameless as a precondition

Blameless does not mean no human action is mentioned. It means the postmortem treats human action as a signal about the system, not a target for criticism.

If "the engineer pushed the wrong config" is a contributing cause, the postmortem question is **why was it possible to push the wrong config without catching it?** — not "the engineer should have been more careful." The system, including its tooling, defaults, review process, and documentation, allowed the mistake. That is what changes.

The Verica/Jeli framing: **"how did this make sense at the time?"** People act based on the information they have, the time pressure they face, and the tools they have available. Reconstruct that context. If their action was reasonable given what they knew, the gap is in what was knowable — better signals, clearer dashboards, safer defaults.

A blame-oriented postmortem destroys the willingness of engineers to be honest about what they did. Without honest accounts, the timeline is wrong, contributing causes are missed, and the same incident happens again.

---

## Structure

A useful postmortem contains:

**Summary** — 2-3 sentences. What happened, who was affected, how long, current status.

**Impact** — concrete, quantitative. Number of users affected. Duration. Revenue or transactions lost. SLA implications.

**Timeline** — chronological record of what was observed, decided, and done. Times in UTC. Each entry has a source (alert fired, log message, human decision). The timeline is the single most important part of the postmortem — it is the substrate for analysis.

**Contributing causes** — the chain of conditions that made this incident possible. Multiple, not singular. Each cause links to a specific change in the timeline.

**What went well** — practices and tools that worked. Not flattery — actual signals about what to preserve. ("The runbook for X had the right command; the rollback completed in 4 minutes.")

**What did not go well** — concrete gaps. Detection delays, missing runbooks, slow tooling, communication breakdowns, knowledge that only one person had.

**Action items** — specific, owned, and dated. Each addresses a contributing cause or a "did not go well" item. Without action items, the postmortem is documentation. With them, it is engineering work.

---

## Contributing cause vs. root cause

The "single root cause" framing is a trap. Most incidents have multiple necessary conditions; removing any one of them would have prevented or limited the incident.

**Contributing causes** are the things that, in combination, made the incident possible. List them all. Examples for a typical incident:
- A deploy added a query without an index
- The query worked in dev because dev data was small
- The CI pipeline did not include a slow-query check
- Production query monitoring did not alert until p99 latency was already SLO-violating
- The on-call engineer did not know the rollback procedure for this service

Each of these is a contributing cause. Removing any one would have changed the outcome. The action items should target the ones with the highest leverage — not "find and fix the one true root cause."

**The 5-Whys trap.** Repeatedly asking "why" linearly produces a single chain that misses the parallel contributing factors. It also tends to end at "human error" or "we should have known better" — neither is actionable. Use 5-Whys as a brainstorming aid, not a structural framework.

---

## Action item classification

Every action item targets one of three categories:

**Detection** — would have surfaced the problem sooner. Add monitoring on X. Alert when Y exceeds Z. Report metric W.

**Prevention** — would have stopped the problem from occurring. Add validation in CI. Make the dangerous default the safer one. Require code review for changes to file X.

**Mitigation** — would have reduced the impact when the problem occurred. Add a circuit breaker. Document the rollback procedure. Add a kill switch for feature Y.

A balanced postmortem has action items in all three categories. Many postmortems over-index on prevention ("this should never happen again"). That is impossible — focus on the system's ability to detect and recover, which compounds over many future incidents.

**Each action item must have an owner and a target date.** Action items without owners do not happen. Action items without dates slip indefinitely.

---

## Postmortem hygiene

**Write the postmortem within 5 days of the incident.** Memory degrades fast. After a week, the timeline is reconstructed from logs and chat archives, with everyone's perception filtered by hindsight.

**The IC drives the postmortem but does not write it alone.** Multiple participants reduce the bias of any single perspective.

**Review postmortems publicly within the engineering org.** A postmortem read only by the team that owned the incident teaches no one else. Cross-team learning is the highest-leverage outcome.

**Track action item completion.** If postmortem action items consistently slip, the postmortem process is theatre. Either reduce the volume of items or increase the priority — do not let them rot.

---

## What is not a postmortem

- A status update for management ("here is what we did well")
- A document used to assign disciplinary action
- A prose narrative without a timeline
- A list of "we should be more careful next time" items

If the document does not produce concrete changes to the system or process, it did not earn the time spent on it.
