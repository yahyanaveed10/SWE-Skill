# Coupling Types

Coupling measures how much one module depends on the internals of another. Lower coupling means changes are more localised. The types below are ordered from highest (most dangerous) to lowest (most desirable).

---

## Content Coupling (avoid)

**Signal:** One module directly reads or writes another module's internal data — bypassing its interface. A change to the internal structure of module B requires a change to module A.

**Ask:** Is this module accessing fields, memory, or internal state that belongs to another module? If the other module's internals change, how many places break?

**Direction:** Move behaviour to the module that owns the data, or expose the operation through a controlled interface.

---

## Common Coupling (avoid)

**Signal:** Two or more modules share global state (global variables, singleton with mutable state, shared configuration object). Neither module knows which other modules read or write the shared state.

**Ask:** If this shared value changes, which modules could be affected? Can you enumerate them? If not, the coupling is too diffuse to reason about.

**Direction:** Pass state explicitly as parameters. If shared state is genuinely needed, own it in a single authoritative module and expose it through controlled operations.

---

## External Coupling

**Signal:** Modules share an external format — a file format, a protocol, a data schema, or an external device interface. The coupling is to the external format, not to each other directly.

**Ask:** If the external format changes, how many modules need to change? Is there a single translation layer that absorbs format changes, or is the format parsed in multiple places?

**Direction:** Introduce a single parsing/serialisation boundary. External format changes then affect only that boundary.

---

## Control Coupling

**Signal:** One module passes a flag or mode to another that controls what the receiver does. The sender knows something about the receiver's internal behaviour.

**Ask:** Does the flag represent a genuine parameter (e.g. a comparison function) or does it select between fundamentally different behaviours? If fundamentally different, the receiver is doing two different things and should be split.

**Good form:** A sort function that accepts a comparison function — the sender controls ordering, not behaviour.
**Bad form:** A function that accepts a `mode` flag and branches internally into completely different code paths for each mode.

---

## Stamp Coupling

**Signal:** A complete data structure is passed to a module that only uses part of it. The receiving module is coupled to the shape of the whole structure even though it only needs a field or two.

**Example:** `calculateSalary(employee)` when only `employee.hourlyRate` and `employee.hoursWorked` are needed.

**Ask:** What does this function actually need from the object it receives? Could it accept those values directly instead of the whole object?

**Direction:** Pass only what is needed. This reduces the surface area of the dependency and makes the function easier to test in isolation.

---

## Data Coupling (target)

**Signal:** Modules interact only by passing simple data values — primitives or simple value objects. No shared state, no control flags, no structural dependencies.

**Example:** `calculateSalary(hourlyRate, hoursWorked)`.

**Ask:** Is this interaction as simple as it could be? Can the interface be reduced to the minimum data needed?

This is the target. It does not always mean functions with many primitive parameters — sometimes a well-defined value object is the right abstraction. The question is whether the receiver depends on more than it uses.

---

## Coupling metrics

| Metric | What it measures | Signal |
|---|---|---|
| CBO (Coupling Between Objects) | Number of classes a class is directly coupled to | High CBO → many dependencies, low modularity |
| RFC (Response For a Class) | Distinct methods executable in response to a message | High RFC → high complexity, high test effort |
| Ce (Efferent Coupling) | Modules this module depends on | High Ce → this module is fragile (depends on many others) |
| Ca (Afferent Coupling) | Modules that depend on this module | High Ca → this module is load-bearing (risky to change) |
| Instability | Ce / (Ca + Ce) → 0 to 1 | 0 = stable (depended on, depends on little); 1 = unstable (depends on many, few depend on it) |
| Propagation Cost | Fraction of system potentially affected by a change here | High → structural risk |

**Instability and the Stable Dependencies Principle:** High-level policy modules should have low instability (stable). Low-level detail modules should have high instability (easy to replace). If a stable module depends on an unstable one, a change in the unstable module forces a change in stable policy — architectural risk.

**Dependency cycles:** Any non-zero cycle count is a structural problem. Cycles prevent independent testing and deployment, and they make the instability metric meaningless within the cycle.
