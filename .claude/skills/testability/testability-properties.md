# Testability Properties

Three properties that determine how testable a piece of code is. Each has a design signal and a set of design moves.

---

## Controllability

**What it is:** The ability to put the system under test into the state needed for a test, and to supply the inputs needed to exercise a specific behaviour.

**Signal that it is missing:** The test setup is longer than the test itself. You are creating large object graphs, mocking many dependencies, or relying on data that must exist in a database before the test runs.

**Root causes:**
- Global state that cannot be reset between tests
- Hard-coded dependencies (the class creates its own collaborators rather than receiving them)
- Temporal coupling (the system must be in a specific sequence of states before the operation under test is valid)
- External dependencies (the real network, real filesystem, real clock) baked into the code path

**Design moves:**
- Pass dependencies in rather than creating them inside — dependency injection at the constructor or method level
- Extract the clock, random number generator, and external service calls behind an interface that can be replaced in tests
- Avoid global mutable state; if shared state is necessary, scope it explicitly and reset it explicitly
- Separate pure logic (no side effects) from side-effecting operations — pure logic is trivially controllable

---

## Observability

**What it is:** The ability to observe the result of an operation in a way that lets you verify correctness.

**Signal that it is missing:** The test passes but you are not confident it actually verified anything. You are asserting on internal state (private fields) rather than observable behaviour. Or there is no return value and no observable side effect to check.

**Root causes:**
- Methods that return void and produce side effects that are hard to observe from outside
- Results written to external systems (database, file, network) rather than returned
- Output buried in logging rather than surfaced through the return value or a captured output
- Verifying implementation details (which methods were called, in what order) rather than outcomes

**Design moves:**
- Prefer return values over side effects where possible — they are directly assertable
- For side effects that cannot be avoided, pass in a collaborator that captures the result (repository, event bus, output stream) — the collaborator can be replaced with a test double that makes the result observable
- Avoid verifying *how* something happened (method call counts, call order) unless the interaction itself is the contract. Verify *what* the outcome was.
- If you must test something that writes to a real external system, introduce an interface that in production writes to the real system and in tests writes to a capture structure

---

## Isolation

**What it is:** The ability to test a unit of code without triggering the execution of unrelated code — particularly code that is slow, non-deterministic, or has side effects.

**Signal that it is missing:** A unit test starts a database, makes network calls, depends on the filesystem, or fails non-deterministically. Tests that cannot be run in parallel because they share state.

**Root causes:**
- Direct instantiation of heavy collaborators inside the code under test
- Shared state between tests (global variables, singleton state, shared database)
- Reliance on wall-clock time or random numbers that produce non-deterministic results
- Test data that must be seeded into a real database before the test runs

**Design moves:**
- Make the boundary between the system under test and its collaborators explicit — through interfaces, not concrete types
- Replace heavy collaborators (database, network, filesystem) with lightweight test doubles at the boundary
- Scope shared state explicitly and provide a reset mechanism — or better, eliminate shared state
- Inject time and randomness as dependencies (a `Clock` interface, a `Random` interface) so tests can control them

---

## Automatability

**What it is:** The ability to run tests without human intervention — in CI, on every commit, without setup steps that require human judgment.

**Signal that it is missing:** "You have to run these tests manually." Setup requires a human to configure something, approve something, or observe something. Tests are skipped in CI because they are flaky.

**Root causes:**
- Tests that depend on external services that are not available in CI
- Tests that require manual configuration of environment variables or credentials
- Flaky tests that pass sometimes and fail sometimes, causing teams to ignore them

**Design moves:**
- Tests that require external services should use test doubles in CI and mark the real-service tests as integration tests that run separately with explicit setup
- Environment-dependent configuration should be injectable and have sensible test defaults
- Flaky tests must be fixed or quarantined — a flaky test in the main suite is a broken canary that trains teams to ignore test failures
