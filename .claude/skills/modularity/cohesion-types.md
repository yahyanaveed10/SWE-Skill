# Cohesion Types

Cohesion measures how strongly the elements within a module belong together. Higher cohesion means the module has a clear, singular purpose. The types below are ordered from lowest (most problematic) to highest (most desirable).

---

## Coincidental Cohesion (avoid)

**Signal:** Elements are grouped in the same module for no reason other than convenience or history — a "utils" file, a "helpers" module, a "misc" package. There is no common theme.

**Ask:** What is the shared responsibility that justifies these elements being together? If the answer is "nothing in particular", cohesion is coincidental.

**Direction:** Find natural groupings by responsibility. Utility modules often dissolve into several small, focused modules once the actual groupings become visible.

---

## Logical Cohesion

**Signal:** Elements perform similar operations (e.g. "all input validation", "all logging") but operate on different data and are selected by a control flag.

**Ask:** Is there a mode flag or type check that selects which operation to perform? If so, the module is a collection of related-looking but independent operations bundled for convenience.

**Direction:** Separate the operations. Logical cohesion often masquerades as good design because the operations feel related — but they have different reasons to change.

---

## Temporal Cohesion

**Signal:** Elements are grouped because they execute at the same time — initialisation routines, shutdown sequences, startup hooks. The connection is timing, not purpose.

**Ask:** Would these elements change together for the same reason, or do they only share a moment in time?

**Direction:** Temporal cohesion is often acceptable at system boundaries (startup, teardown). Inside domain logic, it signals that the elements belong in different modules that happen to be orchestrated together.

---

## Procedural Cohesion

**Signal:** Elements follow a defined sequence of steps and are grouped because they occur in order — not because they share data or purpose.

**Ask:** Is the grouping driven by "these steps happen after each other" rather than "these steps work on the same concern"?

**Direction:** Evaluate whether the sequence is a coordination concern (orchestrator) or a domain concern. If it is just sequencing, the grouped elements may belong in separate modules that an orchestrator calls in order.

---

## Communicational Cohesion

**Signal:** Elements operate on the same data or contribute to the same output, but do not necessarily follow a single sequence or share a single responsibility.

**Ask:** What data are these elements all operating on? Is that data a genuine domain concept that justifies its own module?

**Direction:** Often a step toward functional cohesion. Extract the shared data as a concept and let the module own operations on that concept.

---

## Sequential Cohesion

**Signal:** The output of one element is the input of the next. Elements form a pipeline over a shared data transformation.

**Ask:** Is this a genuine transformation pipeline where each step refines the output of the previous? Or is it just sequencing?

**Direction:** Sequential cohesion is good — it reflects a clear data flow. Verify that the pipeline steps are genuinely part of the same transformation and not logically distinct operations bundled for convenience.

---

## Functional Cohesion (target)

**Signal:** Every element contributes to a single, well-defined task. The module does exactly one thing and does it completely. You can state its responsibility in one sentence.

**Ask:** If I remove one element, does the module's responsibility change? If yes, that element is essential. If no, it may not belong here.

This is the target. A module with functional cohesion is easy to name, easy to test, easy to reason about, and changes for one reason.
