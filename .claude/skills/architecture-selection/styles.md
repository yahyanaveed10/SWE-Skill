# Architecture Styles

Trade-offs for each major style. Each entry: what it is, fits when, does not fit when, migration cost if you need to leave it.

---

## Monolith

**What it is:** All functionality deployed as a single unit. Modules may be well-structured internally, but they are deployed together and share a process.

**Fits when:**
- Small to medium team (fewer than ~8 engineers working in the same codebase simultaneously)
- Low operational maturity — fewer deployments, simpler infrastructure
- Domain is not yet well-understood — boundaries are easier to refactor inside a monolith
- Consistency requirements are high — monoliths can use ACID transactions across the entire domain
- Speed of initial development matters — no distributed systems overhead

**Does not fit when:**
- Different components need to scale independently at vastly different rates
- Teams need to deploy independently without coordinating releases
- Different parts of the system have incompatible technology requirements

**Migration cost:** Moving from monolith to microservices is a well-understood (but expensive) migration. The Strangler Fig pattern — gradually replacing parts of the monolith with services — is the standard approach. Starting with microservices and finding you needed a monolith is much harder to reverse.

---

## Layered (N-tier)

**What it is:** Code is organised into horizontal layers — presentation, business logic, data access — with dependencies flowing downward (each layer only calls the layer below it).

**Fits when:**
- Small to medium applications with a traditional CRUD or request-response model
- Team is familiar with the pattern and the domain fits naturally into layers
- Consistency and transactional integrity across the full stack are required

**Does not fit when:**
- Business logic bleeds into the presentation or data access layer (the pattern becomes nominal, not real)
- Different features have radically different technical profiles that do not fit the same layer structure
- The system needs to be tested without the database or UI (layering does not enforce this — Hexagonal does)

**Watch for:** "Layer cake" where every change requires modifications to all three layers, or where the business logic layer is a thin pass-through to the data layer.

---

## Hexagonal (Ports and Adapters)

**What it is:** The domain/business logic is the centre. Interactions with the outside world (UI, database, external APIs) happen through explicitly defined ports (interfaces). Adapters implement those ports for specific technologies.

**Fits when:**
- Testability is a priority — the domain can be tested without any infrastructure
- The same domain logic must work with multiple delivery mechanisms or data stores
- The team wants to delay technology choices and keep the domain clean
- Long-lived systems where infrastructure changes are expected

**Does not fit when:**
- The domain is trivial — the overhead of ports and adapters exceeds the benefit
- Team is not familiar with the pattern — misapplied, it produces the same layering under a different name

**Advantage over layered:** The domain has no dependency on infrastructure. Infrastructure depends on the domain, not the reverse.

---

## Event-Driven

**What it is:** Components communicate by publishing and consuming events. The producer does not know who consumes its events. Consumers react to events asynchronously.

**Fits when:**
- Operations do not need to complete synchronously from the caller's perspective
- Different parts of the system need to react to the same state change without the producer knowing all consumers
- High write throughput — events can be buffered and processed at the consumer's pace
- Audit trail is valuable — events naturally produce an immutable history

**Does not fit when:**
- Operations require immediate, synchronous confirmation to the user
- Strong consistency is required — event-driven systems are eventually consistent by nature
- The team is not experienced with the operational complexity (message brokers, consumer lag, dead letter queues, idempotency)

**Ask:** Can this operation complete asynchronously, or does the caller need an immediate result? If it needs an immediate result, event-driven adds complexity without solving the problem.

---

## Microservices

**What it is:** The system is decomposed into independently deployable services, each owning its own data, communicating over the network (REST, gRPC, events).

**Fits when:**
- Multiple teams need to deploy independently without coordinating releases
- Different services have genuinely different scaling requirements
- The domain boundaries are well understood (bounded contexts are clear)
- Operational maturity is high — the team can run distributed tracing, service discovery, circuit breakers, health checks, and independent deployments

**Does not fit when:**
- The domain is not well understood — premature service boundaries are expensive to change
- Operational maturity is low — microservices require significantly more infrastructure and tooling than a monolith
- The team is small — the overhead of cross-service coordination eliminates the benefit
- Strong consistency across service boundaries is required — distributed transactions are complex and failure-prone

**The key question:** Do the benefits of independent deployability justify the distributed systems overhead? If the answer is not obviously yes, start with the monolith.

---

## Serverless / FaaS

**What it is:** Functions are deployed as individual units that scale automatically and are billed per invocation. No server management.

**Fits when:**
- Workload is event-triggered and highly variable (zero to many invocations)
- Operations are stateless and short-lived
- Team wants to minimise infrastructure management
- Cost model works: low average utilisation, high burst tolerance

**Does not fit when:**
- Operations are long-running (many FaaS platforms have execution time limits)
- Cold start latency is unacceptable for the use case
- The system needs persistent connections (WebSockets, long polling)
- The workload is consistently high — serverless pricing exceeds dedicated compute at high sustained load

---

## Pipeline (Pipes and Filters)

**What it is:** Data flows through a sequence of processing stages (filters), connected by channels (pipes). Each filter transforms data and passes it to the next.

**Fits when:**
- The task is fundamentally a data transformation sequence (ETL, media processing, document processing)
- Filters are independently testable and reusable
- The processing stages are well-understood and stable

**Does not fit when:**
- Processing requires complex branching or feedback loops between stages
- The system is primarily request-response rather than data transformation
