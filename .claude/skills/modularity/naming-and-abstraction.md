# Naming and Abstraction Depth

The signals that distinguish good abstractions from bad ones are mostly invisible in structural metrics. They are about the *quality* of the abstraction — what its name conveys, how much it hides, whether it makes the caller's job easier or harder. From John Ousterhout's *A Philosophy of Software Design*.

---

## Names as design

A name is a contract. It tells the reader what the function or variable does without requiring them to read the implementation. A bad name forces every reader to verify by reading the code — a per-reader cost that compounds.

**Signals of a poor name:**
- The name says what the function *is* rather than what it *does* (`Manager`, `Helper`, `Utility`, `Processor` — all generic; tell you nothing about behaviour)
- The name uses words that have no domain meaning (`data`, `info`, `value`, `result`)
- The name describes the implementation rather than the purpose (`linkedListSort` — the caller does not care how it sorts)
- Two names refer to the same thing in different parts of the codebase (`user`, `customer`, `account` — all the same entity)
- The name lies (`getUser` that also creates the user; `validateInput` that also normalises it)

**The rename signal:** if you cannot describe to a colleague what a function does in one sentence, the function may be doing too much, OR its name may not match its purpose. The fix is sometimes a better name; more often it is splitting the function so each part can have a precise name.

**Names should be longer the broader their scope.** A loop counter `i` is fine. A class field needs a descriptive name. A public API needs a name that reads correctly at the call site, in the documentation, and to people who do not know the implementation.

---

## Deep vs. shallow modules

A module's *interface* is what callers see. A module's *implementation* is what is hidden inside it. The depth of a module is the ratio between the two.

**Deep module:** simple interface, complex implementation. The interface hides genuine complexity from callers. Examples: a database driver (call `query("SELECT...")`, get rows; the implementation handles connections, parsing, network errors, retries). A compression library (call `compress(bytes)`, get smaller bytes; the implementation is enormous).

**Shallow module:** complex interface, trivial implementation. The interface exposes nearly as much complexity as it hides. Examples: a wrapper class with one method that delegates to another method on a held object. A `class StringHolder` that wraps a String and exposes get/set methods.

**The signal of a shallow module:** reading the implementation reveals nothing the interface didn't already imply. The module adds names and indirection without adding information hiding.

**Why deep modules are valuable:** they let callers ignore complexity. The caller of `compress()` does not need to understand compression algorithms. The caller of `query()` does not need to manage connections.

**Why shallow modules are harmful:** they spread the cognitive load. Callers must understand the wrapper, the wrapped object, and the relationship between them. The wrapper provides no information hiding because all it does is forward calls.

**The agent failure mode:** generating "clean" code by wrapping every concrete type in an abstract class, even when there is only one implementation and no plan for another. The abstraction is shallow — it hides nothing, costs maintenance, and makes the code harder to follow. See `reuse-and-patterns` on premature abstraction.

---

## Information hiding as the unit of design

The purpose of a module is to hide information — implementation details, data layout, algorithms — so that other modules don't need to know them. The amount and importance of what a module hides is the measure of its value.

**Strong information hiding:** the module's interface gives callers no clue how the module works internally. Changing the implementation does not require changes to any caller. The classic example: a hash table interface (`put`, `get`, `delete`) hides whether the implementation uses linked lists, open addressing, or something else.

**Weak information hiding (information leakage):** the module's interface or types reveal implementation details that callers come to depend on. Now changing the implementation breaks callers.

**Common forms of information leakage:**
- Returning a mutable internal data structure that callers can modify (now changing the data structure breaks them)
- Exposing methods that only make sense given the current implementation (`reHashTable()` on a hash map)
- Public configuration that ties callers to a specific implementation choice
- Side-channel information (timing, memory layout, error messages with internal details)

**Test the abstraction:** could you swap out the entire implementation tomorrow without changing any caller? If yes, information is well-hidden. If no, identify what would need to change in callers — that is what is leaking.

---

## Premature abstraction signals

Abstracting before the pattern of variation is clear produces abstractions that fit the wrong axes. The signal:

**One implementation, abstract base class.** If there is only one concrete implementation and no near-term plan for another, the abstract base is overhead without benefit. Add the abstraction when the second implementation appears, not before.

**The abstraction has the same shape as one specific implementation.** The "abstract" interface exposes methods that only make sense for one of the concrete implementations. The abstraction was not designed; it was extracted from one example.

**Multiple implementations have wildly different overrides.** The abstract method that all subclasses implement is empty in two of three classes, throws `NotImplementedException` in another, and only does meaningful work in one. The classes do not actually share behaviour — they share an interface that does not fit them.

**The fix is not always "delete the abstraction."** Sometimes the right move is to wait until the actual variation pattern is clear, then redesign. Premature abstraction is the cost; redesigning is the recovery.

---

## How this connects to coupling and cohesion

A deep module with strong information hiding has **low efferent coupling** (it depends on few things) and **high cohesion** (its methods all support its single purpose). The metrics from the structural signals are downstream of getting the abstraction right.

A shallow module that leaks information has **high coupling** (callers depend on its internals) and often **low cohesion** (it has no clear single purpose because it is just plumbing).

Naming and abstraction depth are how to design *toward* good coupling and cohesion — not metrics to evaluate after the fact.
