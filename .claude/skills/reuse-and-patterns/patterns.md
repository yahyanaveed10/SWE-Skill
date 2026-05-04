# Design Patterns

Design patterns are solutions to recurring design problems. They are most useful as a vocabulary for recognising a problem and a starting point for solving it — not as templates to apply mechanically.

The signal that a pattern applies is always in the problem, not in the solution. If you start from "let's use a Strategy pattern here" rather than "we have a problem where behaviour varies by context and we need to select it at runtime", you are applying patterns to manufacture engineering credibility, not to solve problems.

Each entry: **signal the problem exists → what the pattern provides → when it adds value → when it adds indirection without value**.

---

## Creational Patterns

### Factory / Factory Method

**Signal:** The exact type of object to create is not known until runtime, or the creation logic is complex enough that constructors are doing too much work.

**Provides:** A single place where object creation decisions are made. Callers do not need to know the concrete type.

**Worth it when:** The set of types grows or changes, and you want creation decisions to be localised.

**Not worth it when:** There is one type and no plan to add more. A factory that always creates the same thing is indirection for its own sake.

### Builder

**Signal:** Object construction requires many parameters, some optional, and telescoping constructors are becoming unreadable. Or construction involves multiple steps that must happen in a specific order.

**Provides:** A readable, step-by-step construction API. Separates the construction of a complex object from its representation.

**Worth it when:** The object has more than ~4 construction parameters, or construction order matters.

**Not worth it when:** The object is simple. A builder for a two-field struct is overengineering.

### Singleton

**Signal:** Exactly one instance of a class must exist throughout the system's lifetime, and that instance must be accessible from multiple places.

**Worth it when:** The constraint is genuine — a global registry, a hardware interface, a connection pool.

**Not worth it when:** The "exactly one" constraint is an assumption, not a requirement. Singletons introduce hidden global state and make testing harder. Ask whether dependency injection or explicit passing achieves the same goal without the global state.

---

## Structural Patterns

### Adapter

**Signal:** You need to use an existing class whose interface is incompatible with what your code expects, and you cannot change either side.

**Provides:** A translation layer that makes the incompatible interface look like the expected one.

**Worth it when:** Integrating with a third-party library or legacy component with a fixed interface.

**Not worth it when:** You control both sides. If you control both interfaces, align them directly rather than adapting.

### Decorator

**Signal:** You need to add responsibilities to an object at runtime, and subclassing would produce an explosion of classes (one for each combination of responsibilities).

**Provides:** Composable wrappers that add behaviour without modifying the wrapped object.

**Worth it when:** Responsibilities can be mixed and matched independently (logging + caching + validation on the same object).

**Not worth it when:** The number of combinations is small and fixed. Two subclasses are simpler than a decorator chain.

### Facade

**Signal:** A subsystem is complex with many interacting parts, and consumers only need a simplified view of it.

**Provides:** A single entry point that hides subsystem complexity. Does not prevent advanced use of the subsystem — just simplifies the common case.

**Worth it when:** The subsystem has a coherent set of "normal use" operations that can be abstracted behind a simple API.

**Not worth it when:** The facade becomes a God Class (see anti-patterns) that owns logic rather than delegating. A facade should be thin.

### Proxy

**Signal:** You need to control access to an object — for lazy initialisation, access control, logging, or remote access.

**Provides:** An intermediary that has the same interface as the real object but adds the control concern.

**Worth it when:** The control concern is real and distinct from the object's own responsibility.

**Not worth it when:** The proxy is just a wrapper that forwards everything. If there is no actual control logic, it is just indirection.

---

## Behavioural Patterns

### Strategy

**Signal:** An algorithm or behaviour varies by context, and you want to select it at runtime without a chain of `if/else` or `switch` statements.

**Provides:** A family of interchangeable algorithms behind a common interface. The context delegates to the strategy.

**Worth it when:** The number of variants is growing, or variants come from outside the codebase (plugins, configuration).

**Not worth it when:** There are two variants and they will not grow. An `if` statement is simpler and more readable.

### Observer

**Signal:** One object's state change should trigger updates in other objects, without the source needing to know who those objects are.

**Provides:** A publish/subscribe relationship. The source publishes events; observers register and react.

**Worth it when:** The set of observers is dynamic, or the source should not have a compile-time dependency on its observers.

**Not worth it when:** The relationship is fixed and obvious. If A always updates B, a direct call is clearer than event infrastructure.

### Command

**Signal:** You need to encapsulate an operation as an object — to support undo/redo, queuing, logging of operations, or transactional behaviour.

**Provides:** Operations as first-class objects with an `execute()` interface.

**Worth it when:** Operations need to be stored, queued, undone, or composed.

**Not worth it when:** Operations are simple and immediate. Wrapping a one-line call in a Command object is overhead without benefit.

### Template Method

**Signal:** An algorithm has a fixed overall structure but certain steps vary by subclass. You want to define the structure once and let subclasses fill in the variable steps.

**Provides:** A base class that defines the algorithm skeleton; subclasses override specific steps.

**Worth it when:** The invariant structure is real and shared across multiple implementations.

**Not worth it when:** The "template" is so thin that it is just an interface. Composition via Strategy is usually preferable to inheritance via Template Method — it avoids tight coupling to a base class.
