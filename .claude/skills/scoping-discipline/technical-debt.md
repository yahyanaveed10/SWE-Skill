# Technical Debt

Technical debt is the cost of using code that is not as good as it could be. The metaphor is precise: like financial debt, it can be useful (taken deliberately to ship faster), it accumulates interest (slows down future work), and it must eventually be paid down or it consumes the team.

The misuse of the term is the source of most arguments about it. "All the code I don't like" is not technical debt. Code that was never well-designed is not debt — it is just bad code. Debt is a deliberate or detected shortcut whose cost is now being paid in slowed development.

---

## The Cunningham-Ward debt quadrant

Two axes: **deliberate vs. inadvertent**, and **prudent vs. reckless**.

|                | Deliberate | Inadvertent |
|----------------|------------|-------------|
| **Reckless** | "We don't have time for design." | "What's layering?" |
| **Prudent** | "We must ship now and deal with consequences." | "Now we know how we should have done it." |

**Prudent deliberate debt** — the team knew the right thing, chose a shortcut for a clear reason (deadline, validation), and tracked the debt for later payment. This is the only kind of debt that is unambiguously fine to take.

**Prudent inadvertent debt** — the team did their best with what they knew. After learning more (from the system in use, from a new requirement, from experience), they realise the design should be different. This is normal and unavoidable; the response is refactoring as you learn.

**Reckless deliberate debt** — the team knew better and chose to skip the work without understanding the cost. This is where most "we'll fix it later" debt comes from, and "later" usually never comes.

**Reckless inadvertent debt** — the team did not know the right approach and shipped without learning. The code is bad and the team does not yet realise it. This converges with prudent inadvertent debt as the team learns; the difference is whether there is a culture of seeking out and fixing it.

**The quadrant matters because the response differs:**
- Deliberate prudent: track it, pay it down on a schedule
- Inadvertent prudent: refactor as you encounter it
- Reckless: address the team's process and skill gaps that produced it

---

## Tracking debt

Debt that is not tracked is debt that is not paid. The goal of tracking is not exhaustive cataloguing — it is making the cost visible enough that engineering and product can decide whether to invest in payment.

**Useful tracking signals:**
- A backlog tag for debt items (separate from feature backlog)
- TODO/FIXME comments that include who, when, and why (not just "fix this") — but accept that comments rot; backlog items don't
- A periodic "tech debt review" (monthly or quarterly) where the team surfaces what is currently slowing them down

**The wrong way to track:**
- Aspirational refactoring lists with no priority and no owner — these never get done
- Counting individual code smells with a static analyser — most of the count is noise
- Treating "not idiomatic" or "not my preference" as debt — that's taste, not debt

**Test for whether something is debt worth tracking:** is there evidence that this is slowing down future work, or causing bugs, or making changes risky? If yes, it is debt. If no, it might just be code that someone doesn't like.

---

## Paydown patterns

**The boy scout rule.** "Leave the campsite cleaner than you found it." When touching code for a feature change, fix small adjacent issues — better names, removed duplication, clearer structure. Distributes paydown across feature work without dedicated debt projects. Works for small debt. Fails for large structural debt that needs concentrated effort.

**Strangler fig.** Gradually replace an old subsystem by routing new functionality through new code while old code continues to serve old functionality. Over time, less and less goes through the old system until it can be removed. The pattern for large legacy migrations.

**Refactor under test.** Add tests that characterise the current behaviour (even if the behaviour is wrong) before refactoring. The tests pin down what cannot change; refactoring can then proceed safely. Without tests, refactoring is gambling.

**Dedicated debt sprint / 20% time.** A scheduled period where the team works only on debt, or a fixed percentage of every sprint dedicated to debt. Works when debt is large enough to need concentrated work and the team will actually use the time for debt rather than features.

**Replace before maintain.** When a piece of code reaches a state where every change risks new bugs, replacing it from scratch may be cheaper than continuing to patch it. The decision rests on: how much of the current code's behaviour is necessary vs. accidental? If most behaviour is accidental, replace.

---

## Making the case for paydown

Engineers often want to fix debt; product and stakeholders often want to ship features. The case for paydown must be in their language, not "the code offends my aesthetics."

**Useful framings:**
- "We cannot ship feature X in less than 6 weeks because of debt Y. After paying down Y (4 weeks), feature X takes 2 weeks. Total time is the same; future features in this area are also faster."
- "Last quarter we had 3 incidents caused by debt Z. Each cost us [time/users/revenue]. Paying down Z would prevent this class of incident."
- "We are losing 2 days per sprint to friction in area W. Over the next year, that is 24 days. A 5-day refactor of W eliminates the friction."

**Useful evidence to gather:**
- Time spent on bug fixes per area of code (debt-heavy areas have more bugs)
- Frequency of changes in an area (high-change areas with debt produce the most slowdown)
- Recent incidents traceable to the debt

**Useless framings:**
- "The code is messy / not clean / not best practice" — true and irrelevant
- "I want to refactor this" — preference, not value
- "This will pay off eventually" — eventually is not a plan

---

## Preventing debt accumulation

Cheaper than paying down: not accumulating in the first place.

**Code review for debt risk.** A PR that ships a clear shortcut should explicitly note it ("this is a stopgap; tracked as TICKET-123 to address by Q3"). PRs that ship reckless debt without acknowledgement should be challenged in review.

**Architecture decision records (ADRs).** When a deliberate debt is taken (we chose to defer X), record it. Future contributors find the record and know it is intentional, not negligent.

**The agreement that the next time this code is touched, the debt is paid.** Couple debt to the next feature in the area. Avoids dedicated debt projects (which are hard to schedule) and ensures debt is paid before it compounds further.
