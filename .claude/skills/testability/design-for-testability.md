# Design for Testability

Concrete design moves that improve testability. These are design decisions made at write time, not workarounds applied at test time.

---

## Dependency injection

The most impactful single move for testability. Instead of creating dependencies inside a class:

```python
class OrderService:
    def __init__(self):
        self.repo = PostgresOrderRepository()  # hard-coded, untestable
```

Receive them from outside:

```python
class OrderService:
    def __init__(self, repo: OrderRepository):  # injectable, testable
        self.repo = repo
```

In production, inject the real implementation. In tests, inject a lightweight substitute.

**Ask:** Which dependencies does this class create directly? Could they be received instead?

---

## Seam identification

A seam is a place in the code where you can change behaviour without modifying that code. Seams are where tests reach in.

Common seam types:
- **Interface seams** — the production code depends on an interface; tests supply a different implementation
- **Parameter seams** — the production code receives what it needs as parameters; tests supply controlled values
- **Preprocessor/config seams** — behaviour is controlled by configuration; tests set specific configuration

When code has no seams, it must be modified to be tested. When modification is not acceptable (legacy code, generated code), the first step is to introduce seams without changing observable behaviour.

---

## Pure functions

Pure functions — functions that return a value based only on their inputs, with no side effects and no dependency on external state — are trivially testable. No setup, no mocking, no isolation concerns. The test calls the function with inputs and checks the output.

**Ask:** Which parts of this code have no real need for side effects? Separate them from the parts that do. The pure parts become easy to test. The side-effecting parts become narrow and explicit.

---

## Extract and delegate

When a method is doing too much, it is hard to test any one part of it. Extract the sub-operations into smaller methods or collaborators that can be tested independently. The top-level method then delegates to them and can be tested at a higher level or via integration.

---

## Humble Object

When a component is inherently hard to test (a UI renderer, a file writer, a network listener), extract the logic from it and put it in a separate, testable object. The hard-to-test component becomes a thin wrapper that does nothing but delegate — a "humble object" with no logic worth testing.

The logic goes into the extracted object, which is tested. The humble object is not tested or tested minimally.
