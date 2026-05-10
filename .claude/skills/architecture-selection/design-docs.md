# Design Documents (RFCs)

A design doc — sometimes called an RFC (Request for Comments), a one-pager, or a tech spec — is a written proposal of an approach, shared for review before building. Distinct from an ADR, which records a decision *already made*. The RFC is the discussion; the ADR is the conclusion.

Junior engineers consistently skip design docs. Staff-level engineers consistently write them. The reason is the same: writing the doc forces you to think clearly about what you are about to build. Many bad ideas die in the writing.

---

## When to write a design doc

Not for every change. Writing a design doc takes hours (sometimes days) — it must be earning its time.

**Write one when:**
- The work spans multiple weeks or multiple engineers
- The decision affects multiple teams or systems
- The approach is not obvious — there are real alternatives worth considering
- The decision is hard to reverse (data model, public API, infrastructure choice)
- The blast radius of getting it wrong is large (security, compliance, billing, user trust)

**Don't write one when:**
- The work is well-understood and a few hours of code
- The change is local to one module that one person owns
- The decision is reversible in a sprint
- A code change is faster than a doc + discussion

The signal: if the discussion would benefit from being permanent and async, write it down. If the discussion needs to happen right now in a meeting, just have the meeting.

---

## RFC vs. ADR

Both are written artefacts but serve different purposes.

**RFC (proposal):**
- Written *before* the decision
- Solicits feedback and alternative proposals
- Describes the problem, multiple options, and a recommended approach
- Status changes through draft → review → accepted/rejected
- Lives in a format that supports inline comments (Google Doc, Notion, GitHub Discussion, PR description)

**ADR (record):**
- Written *after* the decision
- Documents what was decided and why
- Lives in version control alongside the code (`docs/decisions/NNN-title.md`)
- Status changes through proposed → accepted → deprecated/superseded
- Optimised for someone reading the codebase 2 years later asking "why is it this way?"

**When you have both:** the RFC is the discussion artifact; the ADR captures the conclusion in the codebase. The ADR can be the "Decision" section of the RFC, copied to its own file.

---

## RFC structure

A useful RFC contains:

**Title and metadata.** Author, date, status, reviewers.

**Problem statement.** What is the problem we are trying to solve? What is the cost of not solving it? Who experiences the problem? Be specific — vague problem statements produce vague designs.

**Goals and non-goals.** What this work will achieve, and explicitly what it will NOT achieve. Non-goals are at least as important as goals — they bound the scope and prevent the doc from sprawling.

**Constraints.** What must the solution work within? Existing technology constraints, team constraints, budget, deadlines, compliance requirements.

**Proposed approach.** The recommended solution. Enough detail that a competent engineer could start implementing from it. Architecture diagrams where they help; concrete data models; sequence diagrams for non-obvious flows; specific technology choices.

**Alternatives considered.** Two or three other approaches that were evaluated. For each: how it works, why it was not chosen. This is not a token gesture — the alternatives section is where reviewers find better ideas, and where the doc shows it took the problem seriously.

**Trade-offs.** What does the chosen approach give up? Every design has costs. Stating them explicitly is honest and lets reviewers weigh them.

**Open questions.** Things the author is unsure about and wants reviewer input on. Specific questions, not general "thoughts welcome." A doc with no open questions either solved everything (rare) or did not surface enough.

**Implementation plan / sequencing.** How will this be built and rolled out? In what order? What are the milestones? What can be deferred? What is the rollback plan if it doesn't work?

**Success criteria.** How will we know if this worked? What metric, behaviour, or signal will we look at to judge?

---

## Writing for review

The audience matters. An RFC for the team that owns the area can assume domain context. An RFC that crosses team boundaries needs to provide more context — different teams have different vocabularies and different assumptions.

**Lead with the problem, not the solution.** A reader who does not understand why this matters will not be motivated to engage with the proposal.

**Use diagrams sparingly and well.** A diagram that clarifies a flow is worth a thousand words. A diagram that adds nothing the prose says is decoration. Sequence diagrams, system context diagrams, and data model diagrams are the most consistently useful.

**Be honest about trade-offs.** A doc that only lists positives reads as advocacy, not analysis. Reviewers trust authors who acknowledge what their proposal gives up.

**Resolve disagreements in the doc, not in chat.** If a reviewer raises a concern, the response goes in the doc. The doc becomes the record of how the disagreement was addressed (or why it was not).

**Update status as the doc moves.** Draft → in review → accepted (or rejected). Don't leave docs perpetually in "draft" — others can't tell if your draft is final or in flight.

---

## What an RFC is not

**A status report.** "We did X this quarter." That's not an RFC; that's a report.

**A demand.** "We are doing X." If the doc does not invite reviewers to push back, it is not soliciting comment — it is informing.

**A specification.** A spec describes exactly what to build. An RFC proposes an approach and asks whether it should be built. The spec is downstream of the accepted RFC.

**A meeting agenda.** If the discussion needs to happen synchronously, the doc is preparation, not the deliverable. The doc enables a faster meeting; it does not replace the meeting.

**Optional after acceptance.** An accepted RFC means the decision is made. Engineers building it should not be re-litigating the design choices in code review. If new information changes the picture, update the RFC or write a new one — don't quietly diverge from it.

---

## The cultural piece

A team that writes RFCs and reads them carefully ships better software than a team that does not, even when the same engineers are building. The reasons:
- The author thinks more clearly when forced to write
- Reviewers catch problems before code is written (cheap to fix)
- Distributed teams stay aligned without requiring everyone in the same room
- New engineers can read the doc instead of asking "why does this work this way?"
- Disagreements are addressed before code is committed, not in PR review

The cost is real — writing and reviewing docs takes time. The payoff is real — fewer expensive mistakes, faster onboarding, less rework. Teams that do not write design docs are usually paying these costs in other ways (rework, mistakes shipped, repeated meetings) without realising it.
