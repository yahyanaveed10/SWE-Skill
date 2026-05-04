# Test Smells

Symptoms of poor testability visible in the test code itself. Each smell points to a design problem in the production code.

---

## The test knows too much about the implementation

**Signal:** The test verifies which methods were called, in what order, with what arguments — even when the observable outcome is what matters.

**What it signals:** The test is coupled to the implementation, not the contract. It will break on any refactoring that preserves behaviour but changes how that behaviour is achieved.

**Ask:** What is the actual contract of this unit? Can the test be rewritten to verify the outcome rather than the mechanism?

---

## The setup is longer than the test

**Signal:** Creating the object under test requires constructing a complex graph of dependencies, mocking many collaborators, and seeding substantial state.

**What it signals:** The unit under test has too many dependencies (controllability problem). It is probably doing too much.

**Ask:** Which of these dependencies are actually exercised by this test? Could the unit be split so each part has fewer dependencies?

---

## Testing the mock, not the code

**Signal:** The test passes regardless of what the production code does, because the key assertions are on mock expectations rather than real outcomes.

**Example:** A test that asserts `repository.save()` was called, but does not check whether the saved data is correct.

**What it signals:** The mock is standing in for something that should be directly observable. The observability is missing from the design.

---

## The flaky test

**Signal:** The test passes most of the time but fails intermittently, often without a clear cause.

**Root causes:**
- Shared state not reset between tests (isolation problem)
- Timing dependencies — the test assumes operations complete in a certain order
- Real network or real clock used in a context where timing is non-deterministic
- Tests running in parallel with shared mutable state

**Response:** Do not quarantine and ignore. Flaky tests must be fixed. A team that accepts flaky tests loses the ability to trust test failures.

---

## The test that tests nothing

**Signal:** The test runs without error but makes no meaningful assertion. Assertions are trivially true, missing, or test only that the code did not throw.

**What it signals:** The test was written to increase coverage metrics, not to verify behaviour.

**Ask:** What would have to be true about the code for this test to fail? If the answer is "nothing realistic", the test is not verifying anything.

---

## The integration test doing unit test work

**Signal:** A test starts a database, a message broker, or a network service to test logic that has no real dependency on those systems — the logic could be tested with a simple in-memory substitute.

**What it signals:** The isolation boundary is not drawn at the right level. The slow, fragile integration test is doing work that a fast, isolated unit test should do.

**Ask:** What does this test actually need from the external system? Can that be provided by a lightweight substitute?
