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

## Structural signals worth tracking

**Too many outbound dependencies (high efferent coupling):** This module depends on many others — it is fragile. A change anywhere in its dependency chain can break it. Signal: the import list is long and growing.

**Too many inbound dependencies (high afferent coupling):** Many modules depend on this one — it is load-bearing. Changing it has wide blast radius. Signal: before changing this module, check who calls it; the answer will be "everyone."

**Instability mismatch:** A stable, widely-depended-on module should not itself depend on volatile, frequently-changing modules. If it does, volatility propagates upward into what should be stable. Signal: your core domain logic imports from your infrastructure layer.

**Dependency cycles:** Any cycle between modules is a structural problem — it prevents independent testing, independent deployment, and makes the dependency graph impossible to reason about. Signal: you cannot test module A without also loading module B, which also requires module A. Tools like `madge` (JS), `pydeps` (Python), or `go list -deps` expose cycles.
