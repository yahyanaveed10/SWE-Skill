# Requirements and Ambiguity

The cheapest bug to fix is one that is caught before it is implemented. The most expensive is one shipped to production. Most of the cost of a feature is determined by how clearly the requirement was understood before the code was written — not by how clever the code is.

Agents are particularly bad at this. Given a vague requirement, an agent will produce code that satisfies a *plausible* interpretation, with no surfacing of which interpretation was chosen or what other interpretations were possible.

---

## What ambiguity looks like

A requirement is ambiguous when reasonable engineers, given the same statement, would build different things.

**Common ambiguity patterns:**

**Underspecified scope.** "Add a notification when something important happens." What counts as important? Who gets the notification? Through what channel? With what frequency limits?

**Unclear acceptance criteria.** "It should be fast." How fast? Measured how? At what percentile? Under what load?

**Missing edge cases.** "Allow users to upload files." What file types? What size limit? What happens to existing files with the same name? What happens during the upload if the user navigates away? What happens if the upload fails halfway?

**Conflicting requirements.** "It should be secure" and "it should be easy to use" — both true, but they pull in opposite directions. Which one wins when they conflict?

**Implicit constraints.** The requirement does not mention that the system must work offline, but the user assumed it. The requirement does not mention concurrent edits, but they happen in production.

**Domain terms used loosely.** "Customer" — does that include users who registered but did not buy? "Order" — does that include cancelled orders? Domain terms have precise meanings in a specific business; using them imprecisely produces bugs that are hard to spot in code review.

---

## Decomposing a vague requirement

The technique: enumerate the questions, do not answer them.

Given "Add a notification when something important happens," produce:

> Questions to resolve before this can be implemented:
> 1. What events count as "important"? (Listed by example or by rule?)
> 2. Who receives the notification? (The user the event affects? Their team? Admins?)
> 3. Through what channel? (In-app? Email? Push? SMS?)
> 4. With what frequency limit? (Bursts of similar events — coalesced or sent individually?)
> 5. What happens if the user is offline? (Queued? Dropped after a TTL?)
> 6. Can users opt out? (Per-channel? Per-event-type?)
> 7. What is the latency requirement? (Real-time? Within minutes? Best effort?)
> 8. What evidence will tell us this is working in production?

The list of questions is the artefact to share with the requester. They answer; you build. Without that round trip, the engineer is guessing on each question, and the guesses do not match the requester's intent.

**Why agents should produce this list rather than answer it:** the engineer (or product owner) has access to business context the agent does not. The agent's job is to surface the questions, not to silently make the assumptions.

---

## Identifying hidden constraints

Some requirements have implicit constraints that, if violated, produce immediate complaints. They are not stated because they feel obvious — but agents do not share the human's sense of what is obvious.

**Routine hidden constraints to check for:**

**Authentication and authorisation.** Almost every feature implicitly requires that the right users can do it and the wrong users cannot. Often unstated.

**Data persistence.** The implicit assumption is that data does not disappear on a server restart. For features that are explicitly ephemeral, that should be stated.

**Multi-user concurrency.** What happens when two users do this at the same time? Often the implicit assumption is "it just works" — which usually means race conditions in production.

**Internationalisation, time zones, locale.** Dates, times, currencies, names — all have locale-specific representations. Defaults to the developer's locale produce bugs for everyone else.

**Performance and scale.** "It should work for 100 users" is different from "it should work for 100 million." The number is rarely stated; the system is built for the developer's intuition.

**Failure modes.** What does the system do when a downstream service is down? When the database is slow? When the user's network drops mid-request? Often unstated; the user assumes the system "just works" even under failure.

**The discipline:** before declaring the requirement complete, walk through this list and explicitly note which constraints apply, which don't, and which are unknown.

---

## Writing acceptance criteria

A useful acceptance criterion is testable and unambiguous. The format that works:

> Given [precondition], when [action], then [observable outcome].

Examples:
- *"Given a user with no items in their cart, when they navigate to /checkout, then they see a message 'Your cart is empty' and the checkout button is disabled."*
- *"Given a user with 3 items in their cart and a valid payment method, when they submit the order, then the order is created in 'pending' state, the payment is captured within 5 seconds, and the user sees a confirmation page."*

The criterion specifies: what state the system must be in, what happens, and what the observable result is. A test could be written from this directly.

**Bad acceptance criteria:**
- "The checkout works" — not testable; "works" is undefined
- "Users can place orders" — true at infinite levels of abstraction; specifies nothing
- "It should be intuitive" — opinion, not measurement

---

## When to stop clarifying

Endless clarification is itself a form of avoidance. At some point the engineer has enough information to build a useful first version. The question is when "enough" is enough.

**Signals that you have enough:**
- You can write the acceptance criteria for the core flow
- The major hidden constraints are surfaced and addressed (auth, persistence, concurrency, failure modes)
- You can sketch the data model
- The questions remaining are about edge cases, not the central behaviour

**Signals you don't have enough yet:**
- You cannot describe what the user sees on the happy path
- The requester says "yes" to every interpretation you propose, including contradictory ones
- The team would build different things if they each interpreted the requirement independently

**The 'parking lot' technique:** for non-blocking questions (edge cases, future enhancements), record them in a "parking lot" attached to the work item. They are not lost; they are deliberately deferred. The team can ship the core feature while keeping the deferred questions visible.

---

## When to push back

Some requirements should not be implemented as stated. The engineer's job is to surface this when it applies.

**Signals to push back:**

**The requirement is technically infeasible at the proposed scale or budget.** "Real-time analytics across 1 billion records with 100ms latency for $50/month."

**The requirement contradicts itself.** "Hide all sensitive data from logs" and "log every action a user takes including their inputs."

**The requirement violates a hard constraint (legal, security, ethical).** "Send users a marketing email without their consent" — even if a stakeholder asks for it.

**The requirement is solving the wrong problem.** "Add a button to manually retry failed jobs" might be solving "the user is annoyed by failed jobs" — when the actual fix is to make jobs not fail in the first place. Surface the alternative; let the requester decide.

The discipline: surface concerns clearly with reasoning, propose alternatives, let the decision rest with whoever owns the requirement. Do not silently build something the requester would not have asked for had they understood what you were building.
