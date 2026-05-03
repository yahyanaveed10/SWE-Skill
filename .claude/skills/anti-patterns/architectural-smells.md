# Architectural Smells

Architecture-level anti-patterns. Visible at the system or component level, not at the class level. These are harder to reverse than code smells.

---

## Golden Hammer

**Signal:** The same technology, framework, or tool is applied to every problem regardless of fit. Phrases like "our database is our architecture" or "we use X for everything" are diagnostic. A team with deep expertise in one tool consistently reaches for it first without evaluating alternatives.

**Root cause:** Ignorance, pride, narrow-mindedness. Success with a tool in past contexts breeds over-generalisation.

**Ask:**
- Is this tool actually the best fit for this problem, or is it the tool we know?
- What would we choose if we were starting fresh with no existing investment?
- What are the specific constraints of this problem that make one tool better than another?
- What does the tool make difficult that a different choice would make easy?

**Refactoring direction:** Architectural change, not code change. The solution is process: actively invest in evaluating alternatives before committing. Introduce a lightweight architectural decision record (ADR) step for significant tool choices. Build components that insulate the system from proprietary features — so tool choices remain reversible.

**Trade-off:** Exploring alternatives costs time. A team that evaluates every tool choice rigorously moves slowly. The right investment is proportional to the reversibility of the decision.

**When to stop:** If the team's expertise in the golden hammer provides genuine productivity and the problem fits the tool reasonably well, the cost of switching may exceed the cost of imperfect fit. Be honest about whether "fits reasonably well" is true or a rationalisation.

---

## Design by Committee

**Signal:** A design has accumulated features from every stakeholder. It tries to satisfy every use case, every edge case, and every personal preference. The result is a specification or interface that is internally consistent but too complex for any team to implement completely or correctly. Classic diagnostic: the spec is growing faster than the implementation.

**Root cause:** Pride, avarice. Every contributor adds their concern. Nobody removes anything. The loudest voices win; the clearest design loses.

**Ask:**
- Is there a designated owner of this design? If the answer is "everyone", it is "nobody".
- Which features in the current design are there because of one person's interest, not a validated user need?
- If you implemented exactly half of this design, which half would users notice most?
- What can be removed without affecting the primary use case?

**Refactoring direction:** Process intervention, not code change. Assign a single owner with decision authority. Separate divergent sessions (generate ideas) from convergent sessions (make decisions). Groups larger than five rarely produce clean design — use smaller working groups and present results to the larger group for review, not for editing.

**Trade-off:** Fewer decision-makers moves faster but risks missing important constraints. The right trade-off depends on how well the owner understands all stakeholders' real needs vs. their stated preferences.

**When to stop:** Some complexity is legitimate — a standard or API that must serve genuinely diverse use cases will be large. The question is whether the complexity reflects real variation in requirements or political accumulation of features nobody validated.

---

## Auto-Generated Stovepipe

**Signal:** An existing fine-grained local interface was mechanically translated to a distributed interface without redesign. The result: many small, chatty remote calls; interfaces that expose internal implementation details; assumptions about shared state or local file access embedded in a remote protocol.

**Root cause:** Haste, sloth. Migrating to a distributed system by wrapping existing interfaces is faster than redesigning them for distribution — until the performance and coupling problems appear.

**Ask:**
- How many remote calls does a single user action require? If it is more than two or three, the interface granularity is wrong for distribution.
- Do these interfaces assume shared memory, shared file access, or co-location? Those assumptions break in a distributed context.
- Are the interfaces stable enough for separately compiled consumers to depend on?

**Refactoring direction:** Redesign for the distribution boundary, not wrap it. Define coarse-grained interfaces based on what distributed consumers actually need. Identify shared concerns (data formats, auth, error conventions) as first-class design concerns for the distributed interface, not afterthoughts.

**Trade-off:** Redesigning interfaces breaks existing consumers and requires a migration strategy. The cost of a clean interface up front is lower than the cost of a stovepipe that propagates through the system.

**When to stop:** If the system is not actually distributed (calls are in-process), this pattern does not apply. Check whether the distribution boundary is real before optimising for it.

---

## Big Ball of Mud

**Signal:** The system has no discernible structure. Modules depend on each other in all directions. There are no clear ownership boundaries. The dependency graph is a dense mesh. Changes require understanding large parts of the system because nothing is truly isolated.

**Root cause:** Accumulated haste, apathy, and turnover. No single decision produced the Big Ball of Mud — it grew through thousands of "just make it work" decisions.

**Ask:**
- Can you draw the module structure and have a new team member understand it in under ten minutes? If not, structure is missing.
- What is the highest-traffic change path — which modules change together most often? That clustering often reveals latent structure.
- Is there a layer or boundary that is almost respected but not quite? Enforcing that boundary is the first cut.

**Refactoring direction:** Do not attempt a full rewrite. Find and enforce one structural boundary at a time — the one in the path of the most active development. Use the strangler fig pattern: build the new structure alongside the old, migrating incrementally. The goal is not immediate cleanliness but a direction of improvement.

**Trade-off:** Incremental refactoring toward structure is slower than living with the mud. The question is whether the mud is accelerating development cost faster than the refactoring investment.

**When to stop:** If the system is in maintenance mode with no new feature development, the mud is stable. Refactoring without active feature pressure produces risk with no near-term return.
