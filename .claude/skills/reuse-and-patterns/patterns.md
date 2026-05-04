# Design Patterns

Start from the problem, not the pattern name. If the justification for a pattern is "it's a best practice" or "it's cleaner," that is not a justification — it is cargo culting. The signal that a pattern applies is always in the problem.

The most common agent mistake: applying a pattern because the problem *resembles* the pattern's canonical example, without checking whether the problem actually has the property the pattern solves.

---

## Factory / Factory Method
**Signal:** The concrete type to create is not known until runtime, or creation logic is complex enough that constructors are doing too much.
**Overkill when:** There is one type and no plan to add more. A factory that always creates the same thing is pure indirection.

## Builder
**Signal:** Object construction has 4+ parameters (especially optional ones), making call sites unreadable.
**Overkill when:** The object is simple. A builder for a two-field struct is overengineering.

## Singleton
**Signal:** Exactly one instance must exist and be globally accessible — connection pools, hardware interfaces, global registries.
**Overkill when:** "Exactly one" is an assumption, not a requirement. Singletons introduce hidden global state and break testability. Prefer dependency injection.

## Adapter
**Signal:** You need to use an interface you cannot change (third-party library, legacy component) alongside code that expects a different interface.
**Overkill when:** You control both sides. Align the interfaces directly instead of adapting.

## Decorator
**Signal:** You need to add responsibilities to an object dynamically, and subclassing would produce a class explosion (one per combination of responsibilities).
**Overkill when:** The combinations are small and fixed. Two subclasses are simpler than a decorator chain.

## Facade
**Signal:** A subsystem has many interacting parts and consumers only need a simplified entry point for the common cases.
**Overkill when:** The facade becomes a God Class that owns logic rather than delegating. A facade should be thin.

## Strategy
**Signal:** Behaviour varies by context and the variation is growing or comes from outside the codebase (plugins, configuration, user selection).
**Overkill when:** There are exactly two variants and they will not grow. An `if` statement is more readable.

## Observer
**Signal:** A state change in one object should trigger reactions in others, without the source knowing who the observers are.
**Overkill when:** The relationship is fixed — A always updates B. A direct call is clearer than event infrastructure.

---

## The patterns most often applied wrong

**Singleton applied to anything that is convenient to access globally** — leads to hidden coupling and untestable code. Ask: could this be passed as a parameter instead?

**Strategy applied to two-case switches** — if there are only two strategies and no plan to add more, the pattern adds a class, an interface, and indirection for zero benefit.

**Facade becoming a service layer** — a facade that contains business logic is no longer a facade; it is a God Class with a better name.

**Factory when the caller could just call `new`** — if the caller knows the type and the construction is simple, a factory adds a level of indirection that makes the code harder to follow without making it more flexible.
