# Estimation

Estimation is communicating the relationship between scope and time under uncertainty. Most arguments about estimation are arguments about who gets blamed when the estimate is wrong — which is the wrong frame. The right frame is: how do we make decisions about what to do, given that we cannot know exactly how long things will take?

Agents are particularly bad at estimation because they have no historical data on their own velocity, no calibration between "this task feels like X hours" and how long it actually takes them, and no sense of the surprise factor in production codebases.

---

## The cone of uncertainty

Estimates have wider uncertainty bands earlier in the project and narrower bands as work progresses and unknowns resolve.

At the start: the estimate might be off by 4x in either direction. ("This will take 1-16 weeks.")

After preliminary investigation: 2x in either direction. ("3-12 weeks.")

After detailed design: 1.5x. ("4-9 weeks.")

After implementation begins: ±25%. ("5-7 weeks.")

**The trap:** treating an early estimate as a commitment. When pressed for a number at the start of a project, engineers give a single number ("6 weeks"). This number then becomes a commitment, and anything beyond it is "late." But the engineer's actual confidence at the start is "1-16 weeks." Communicating only the midpoint loses the information about uncertainty.

**The discipline:** when giving an early estimate, give a range and explain what would narrow it. "1-16 weeks. After spiking on the database design and the auth integration, I can give you a tighter range — probably within a week of investigation."

---

## Decomposition for estimation

A single number for a multi-week project is a guess. A sum of many small numbers is more accurate, because the errors partially cancel out.

**The technique:** break the work into pieces small enough to estimate concretely (typically a few hours to a few days each), estimate each, sum the estimates. Add a buffer for integration and unknowns.

**What "estimable" means:** you can describe the work at a level where surprise is unlikely. "Build the auth system" is not estimable — too many unknowns. "Add a login form with email/password validation, error display, and call to /auth endpoint" is estimable.

**The recursive process:** if a piece feels too big to estimate (the gut estimate has wide uncertainty), decompose it further until each piece is concrete.

---

## Why estimates miss

The reasons estimates miss are predictable:

**Unknowns that turn out to be hard.** "I'll integrate library X — should be a day." Turns out X has a quirk that requires three days to work around. Mitigation: spike on uncertain integrations before estimating.

**Scope creep.** The work as built includes things the original estimate did not. ("While I was in there...") Mitigation: explicit scope agreement, anything outside it is a new estimate.

**Integration cost.** Each piece is fast individually; making them work together takes more than the sum of the pieces. Mitigation: include integration as its own line item.

**Testing and review.** "Implementation is done" is not "shipped." Code review, fixing review feedback, writing tests, fixing test failures, deployment — all add real time. Mitigation: include these in the estimate.

**Incident response and unrelated emergencies.** The week the estimate covers includes some hours of unplanned work. Mitigation: estimate at less than 100% utilisation; assume some hours per week go to non-project work.

**Optimism bias.** Engineers tend to estimate the time required to write the happy-path code, then add a small buffer. Reality includes debugging, edge cases, and surprises. Mitigation: track actuals vs. estimates over time; if you systematically miss by 50%, multiply your initial estimates by 1.5.

---

## Communicating uncertainty

The way an estimate is communicated affects the decisions made from it.

**Single number:** "6 weeks." Heard as commitment. Sets up false sense of certainty.

**Range:** "5-9 weeks, most likely 6-7." Conveys uncertainty. Allows the receiver to plan for the worst case.

**Confidence-bracketed estimate:** "I am 50% confident in 6 weeks. I am 90% confident in 9 weeks." Tells the receiver where to put a deadline (the 90% bracket if the deadline is hard).

**Conditional estimate:** "If the auth integration is straightforward, 6 weeks. If we need to build our own session handling because the library doesn't fit, 10 weeks. I'll know after 1 week of investigation." Surfaces the variable that drives the difference.

**The conditional version is usually the most useful.** It tells the receiver what to watch for and gives them information about how to reduce uncertainty (e.g., spend a week investigating the auth question now).

---

## Updating estimates as work progresses

An estimate given at the start is almost certainly wrong. The discipline is to update it as information arrives, not to defend the original.

**Signals that an update is needed:**
- A piece took significantly longer than estimated; remaining pieces are similar in nature
- An unknown resolved in a way that changes other estimates
- New scope was added (or removed)
- The work is taking unusually long for unclear reasons (this is itself a signal — investigate before continuing)

**How to update:**
- Recalculate based on actuals so far + estimates for remaining work
- Adjust remaining estimates if the actuals reveal a systematic underestimate (e.g., everything is taking 1.5x longer — the rest probably will too)
- Communicate the updated estimate explicitly with the reason for the change

**Bad pattern:** silently working on a project that has slipped from a 6-week estimate to 12 weeks without communicating until the original deadline arrives. The receiver of the estimate has been making decisions on bad information.

**Good pattern:** "We're at week 3. The auth integration that I expected to take 3 days took 8. The rest of the work should still go as planned, but the total is now 7 weeks instead of 6. Want me to adjust scope to hold the original date?"

---

## When estimation is and is not useful

**Useful:**
- For scoping decisions (is this feature worth doing? at what cost?)
- For sequencing (which work comes first?)
- For coordinating across teams (when can we expect this to be ready?)
- For setting expectations with customers / stakeholders

**Less useful:**
- For driving day-to-day work (engineers performing to an estimate produces gaming, not better outcomes)
- For evaluating engineer productivity (estimates are best-effort, not commitments; judging engineers on hitting estimates incentivises padding)
- For comparing engineers (one engineer's "3 days" is another's "1 week" — both are correct estimates of their own work)

**Why agents specifically should not give confident estimates:** an agent can produce code quickly, but the iteration to make it correct, the integration with the existing system, the review cycles, and the unforeseen issues are not under the agent's control. An agent that says "this will take 2 hours" with confidence is misleading the user about how long shipping the change actually takes.
