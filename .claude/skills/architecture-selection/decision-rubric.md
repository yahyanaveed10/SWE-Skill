# Decision Rubric

Context dimensions that should drive architecture decisions. These are not a scoring matrix — they are questions to answer honestly before committing to a style.

---

## Team size and structure

**What it tells you:** Architecture and team structure mirror each other (Conway's Law). The architecture you choose will tend to be the architecture your team naturally produces.

**Questions:**
- How many engineers will work on this system simultaneously?
- Are they in one team or multiple teams with different release cadences?
- Will they need to deploy independently of each other?

**Signal:** Small single team → monolith or modular monolith is natural. Multiple teams with independent deployment needs → service boundaries aligned with team boundaries.

---

## Domain understanding

**What it tells you:** Service boundaries that are wrong are expensive to change once data and contracts are distributed. A wrong boundary in a monolith is a refactor; a wrong boundary in microservices is a migration.

**Questions:**
- Are the domain boundaries well understood and stable, or still being discovered?
- Have you run this system before, or is this a greenfield project where the domain is not yet fully understood?

**Signal:** Well-understood, stable boundaries → services are a viable choice. Still discovering the domain → keep it in a monolith until boundaries become clear.

---

## Deployment independence requirement

**What it tells you:** If teams do not need to deploy independently, the operational cost of microservices is pure overhead.

**Questions:**
- Do different parts of the system need to be updated at different rates?
- Does a change in one area require a coordinated deployment with other areas?
- Is the coordination cost of a monolith release actually a problem in practice?

**Signal:** No genuine deployment independence need → the organisational benefit of microservices is absent. The technical overhead remains.

---

## Consistency requirements

**What it tells you:** Distributed systems are eventually consistent by default. Strong consistency across service boundaries requires distributed transactions (2PC, Saga), which are complex and failure-prone.

**Questions:**
- Which operations must be atomic across multiple domains?
- What is the consequence of a partial failure (some data updated, some not)?
- Can the business tolerate eventual consistency, or is strong consistency required?

**Signal:** Strong consistency requirements across multiple domains → service decomposition makes those requirements very hard to meet. Consider whether the boundary should be inside a single service instead.

---

## Operational maturity

**What it tells you:** Microservices and event-driven architectures require significantly more operational infrastructure than a monolith. If the team cannot operate that infrastructure, the architecture creates risk, not benefit.

**Questions:**
- Does the team have experience with distributed tracing, service discovery, circuit breakers, and independent deployments?
- Is there a platform team that provides this infrastructure, or must the product team build it?
- What is the team's current incident response capability for distributed system failures?

**Signal:** Low operational maturity → the operational overhead of a distributed architecture will dominate engineering time. Start simpler.

---

## Scaling requirements

**What it tells you:** Different components need to scale independently only if they have genuinely different load profiles. If the whole system scales together, a monolith scales just as well.

**Questions:**
- Which specific components will have the highest load?
- Is the load profile for different components genuinely different, or does the whole system scale together?
- What are the actual numbers — requests per second, data volume — not theoretical maximums?

**Signal:** Uniform load profile → a single monolith can be scaled horizontally. Only when specific components have 10x+ different load profiles does component-level scaling become necessary.
