# Reuse Strategies

Four strategies for incorporating existing solutions into a new system. Each has a different cost structure, risk profile, and appropriate context.

---

## Adopt a library

**What it means:** Take a dependency on a published package that provides a specific capability. Your code calls the library's API.

**Cost of adoption:** Integration time, reading documentation, understanding edge cases.

**Ongoing cost:** Tracking updates, managing breaking changes, security patches.

**Risk:** The library's behaviour, performance, and maintenance trajectory are not under your control. A library that goes unmaintained or changes API can block your own release.

**Fits when:**
- The capability is well-defined, stable, and not core to your product's competitive differentiation
- The library is widely used, actively maintained, and has a clear versioning policy
- Switching costs are low — you can wrap the library behind your own interface

**Does not fit when:**
- The library does 80% of what you need and the remaining 20% requires forking or working around it
- The library's transitive dependencies conflict with your stack
- The capability is so central to your product that losing control of it is a real risk

---

## Adopt a framework

**What it means:** Structure your code according to the framework's conventions. The framework calls your code (inversion of control).

**Cost of adoption:** Higher than a library — you must understand the framework's mental model, conventions, and lifecycle. Mistakes are harder to detect because the framework drives execution.

**Ongoing cost:** Framework major versions are often breaking. Migration cost is high and tends to be non-negotiable.

**Risk:** **Framework lock-in is real.** Once your architecture conforms to a framework, removing it is a significant rewrite. Evaluate frameworks as though you will use them for five years.

**Fits when:**
- The framework is mature, with a large community and a track record of stable evolution
- The problem domain is well-served by the framework's opinion (web MVC, ML training pipelines, etc.)
- Your team already has expertise in it

**Does not fit when:**
- The framework is new and its maintenance trajectory is unknown
- Your use case is at the edges of what the framework was designed for
- The framework imposes architectural constraints that conflict with your non-functional requirements (performance, deployment model)

---

## Copy and adapt

**What it means:** Take existing code (from an open-source project, a previous project, or a teammate's implementation), copy it, and modify it to fit your context.

**Cost of adoption:** Low upfront — you are not learning a new API.

**Ongoing cost:** You now own the code. Bugs, security issues, and improvements in the original are no longer automatic.

**Risk:** Silent divergence. The copy and the original evolve independently. If the original fixes a critical bug, you may not know.

**Fits when:**
- The existing code solves exactly the problem you have, in a context that differs only in ways you understand
- The code is small enough to own and maintain
- A library or framework would bring more overhead than value

**Does not fit when:**
- The copy will grow into a de facto fork. That fork now needs maintenance.
- The original is under a licence that does not permit copying
- You are copying to avoid understanding the problem. Understanding is not optional.

---

## Extract an abstraction

**What it means:** Identify duplication in your own codebase, find the shared underlying pattern, and extract it into a shared module.

**Cost of adoption:** Design cost — finding the right abstraction boundary is hard. Wrong abstractions are worse than duplication.

**Ongoing cost:** The abstraction now has multiple consumers. Changes to it affect all of them.

**Risk:** Premature extraction. An abstraction extracted before the pattern is fully understood will have the wrong shape and will need to be torn apart when a third or fourth use case doesn't fit.

**Fits when:**
- There are at least three concrete instances of the pattern (two is not enough to know the shape)
- The consumers will change together — if they would change independently, the abstraction is wrong
- The shared logic is genuinely identical in semantics, not just superficially similar in syntax

**Does not fit when:**
- There are only two instances and no near-term expectation of a third
- The code looks the same but serves different purposes that will diverge
- The team does not yet understand the domain well enough to know where the boundary should be

---

## Rule of three

For copy-paste specifically: the first time you write something, write it. The second time, note the duplication. The third time, extract the abstraction. Two copies is often not enough signal to know what the right abstraction is — extracting too early produces the wrong boundary.
