# Refactoring Discipline

Refactoring is structural change to code without changing its observable behaviour. The discipline is what makes the difference between safe, valuable refactoring and a multi-week PR that breaks things and never lands.

This file is about *how* to refactor. For when to refactor (which signals justify it) see `refactoring-signals.md`. For what to refactor away from (smells) see `code-smells.md`.

The agent failure mode this skill targets: making structural code changes without tests, in steps too large to verify, mixing refactoring with feature changes, or refactoring beyond the scope of the original problem.

---

## The hard rule: refactor under test

**Never refactor code that has no tests.** Refactoring without tests is gambling — you cannot tell whether your changes preserved behaviour or broke it silently.

If the code you want to refactor has no tests, the first task is to add tests. Specifically: **characterisation tests** (sometimes called "pinning tests" or "golden master tests") that document what the code currently does, regardless of whether that is what it should do.

A characterisation test:
- Calls the function with realistic inputs
- Captures the output, return value, side effects, exceptions raised
- Asserts that the next run produces the same outputs

This pins the current behaviour. Refactoring proceeds with confidence: any test failure means the refactoring changed behaviour. Any passing test means the refactoring preserved behaviour.

The tests do not need to validate that the behaviour is correct. They validate that the behaviour did not change. The correctness question is separate (and may be the next refactoring step, after the structural improvement is in place).

---

## Small steps with verification

Each refactoring step should:
1. Make one change
2. Run the tests
3. Commit if tests pass; revert if they fail

The smaller the step, the easier to debug when something breaks. A 200-line refactor that fails a test forces you to bisect to find the breaking step. A 5-line refactor that fails a test points you straight at the problem.

**Anti-pattern:** "I will refactor this whole module and then run the tests at the end." When the tests fail (and they will), you have no idea which of the 50 changes broke them. The fix is to revert the entire batch and start over.

**Pattern:** rename → run tests → commit; extract method → run tests → commit; inline variable → run tests → commit. Each commit is self-contained, reviewable, and reversible.

---

## Common safe refactorings

These have well-known mechanics, low risk when done one-at-a-time, and most IDEs can perform them automatically:

**Rename.** Change a name. Mechanics: change the declaration, change every reference. Modern IDEs do this reliably. Most common refactoring; usually safe.

**Extract method/function.** Pull a block of code into its own function. Mechanics: identify inputs and outputs of the block, create a function with those parameters and return value, replace the block with a call to the function. Tests verify behaviour is unchanged.

**Inline variable.** Replace a variable with its initialisation expression. Useful when the variable name does not add clarity over the expression itself.

**Move method.** Relocate a method to a different class. Mechanics: copy to new class, update callers, delete from old class.

**Replace conditional with polymorphism.** When a switch/if-chain branches on a type or kind, replace with polymorphic dispatch. Larger refactoring; build up via smaller steps.

**Extract class.** When one class is doing too much (a Blob), split it into multiple. Larger refactoring; usually built from many smaller refactorings.

---

## Risky refactorings

These have higher risk and require more care:

**Changing a public interface.** Anyone depending on the interface needs to update. The blast radius extends beyond your code to every consumer. Pattern: introduce the new interface alongside the old; deprecate the old; migrate consumers; remove the old. Across multiple PRs, with deprecation warnings.

**Restructuring inheritance.** Changing class hierarchies affects every subclass. Pattern: prefer composition; if you must change inheritance, do it in small steps with extensive test coverage.

**Refactoring across module boundaries.** Moving code between modules requires updating every reference. The risk is missing references, especially in dynamic languages where the references are not type-checked.

**Refactoring concurrent code.** Changes that look behaviour-preserving in single-threaded execution can introduce race conditions. Test under concurrency before and after.

---

## The Mikado method

For refactorings that touch many files, where every change reveals more changes that need to be made, the Mikado method gives a structured way to navigate.

**The process:**
1. Set a goal: "I want X to be true."
2. Try to make it true. Note any changes you needed to make first ("blocking" changes).
3. Revert your changes. Now try to make the blocking changes first.
4. For each blocking change, repeat: try to make it; note its blockers; revert; work on blockers first.
5. You build a tree of dependencies — what needs to happen before what.
6. Now execute from the leaves of the tree, where each step is small and self-contained.

The result: a series of small, safe refactorings that, in the right order, accomplish the original goal. Each step is committable individually. The branch never has to hold a half-finished refactor.

This is slower than just diving in, but it produces refactorings that actually land. The "dive in" approach often produces refactorings that are abandoned after weeks because they grew too large to merge.

---

## When to stop refactoring

Refactoring can become its own form of over-engineering. Signals that you have crossed from useful refactoring into unproductive structural churn:

**Each refactoring step requires more setup than it removes.** You are creating abstractions that are larger than the duplication they replace. The pattern is being applied to code that does not have the property the pattern requires.

**The tests now require complex mocking that the old code did not.** The refactoring may have introduced indirection that makes testing harder, not easier.

**You cannot articulate the benefit of the next step.** "It will be cleaner" is not a benefit. "Adding feature X will be 50% faster after this" is.

**The refactor has been open for more than a few days.** Long-lived refactor branches accumulate merge conflicts and lose touch with the main branch. Either land it or abandon it.

**You are refactoring code that nobody is going to change.** Code in stable, infrequently-modified areas does not benefit from improved structure as much as code in frequently-changed areas. Apply effort proportionally to expected change rate.

---

## Separating refactoring from feature work

**Hard rule:** refactoring and feature changes go in separate commits, often separate PRs. Mixing them produces:
- PRs that are hard to review (reviewer cannot tell which changes are behaviour-preserving and which are behaviour-changing)
- Bugs that are hard to attribute (was it the refactor or the feature?)
- Reverts that take both back when only one was problematic

**Pattern:**
1. Identify a refactor that would make the upcoming feature change easier
2. Land the refactor first as a separate PR — tests still pass, no behaviour change
3. Build the feature on top of the refactored code

**The advantage:** if the feature change has a problem, you can revert just the feature without losing the refactor. The refactor stands on its own merits and can be reviewed independently.

**The exception:** when a refactor is genuinely required to add the feature (the feature cannot exist without the structural change), they may go together — but the commits within the PR should still be separated.
