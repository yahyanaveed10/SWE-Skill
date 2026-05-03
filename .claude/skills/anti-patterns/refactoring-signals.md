# Refactoring Signals

When to refactor vs when to leave it. Refactoring has real costs — understanding when those costs are justified is as important as knowing how to refactor.

---

## Signals that favour refactoring

**The code is in the active change path.** If you are about to add a feature to a messy module, the cost of working around the mess will be paid repeatedly. Cleaning it first is often the cheaper option over a three-month horizon.

**The mess is causing bugs.** Anti-patterns that are causing production issues are generating ongoing cost. The cost of refactoring is bounded; the cost of ongoing bugs is not.

**The mess is blocking new development.** If adding a feature requires understanding the entire Blob or threading through the entire Big Ball of Mud, the productivity tax is active and measurable.

**Tests exist and cover the current behaviour.** Refactoring without tests is high risk — you are changing structure and hoping behaviour is preserved. With tests, you have a verification signal.

**The team understands the domain well enough to identify the right structure.** Premature structure extracted by people who don't yet understand the domain often produces the wrong boundaries. If the domain is still being learned, wait.

---

## Signals that argue against refactoring

**The code is not in the active change path.** Stable code that nobody touches generates no ongoing cost. Refactoring it creates risk with no near-term return.

**Tests do not exist.** The first investment is tests that cover the current behaviour. Refactoring without them moves you from "known broken structure" to "unknown broken structure".

**The deadline is immediate.** If a refactor cannot be completed and verified before a deadline, do not start it. A half-finished refactor is worse than the original anti-pattern.

**You do not yet understand why it is structured this way.** Some code that looks like an anti-pattern is the result of constraints that are not visible in the code — performance requirements, legacy protocol compliance, external system limitations. Read git history and comments before concluding the structure is wrong.

**The system is scheduled for replacement.** Improving a system that will be decommissioned in six months is rarely the best use of engineering time.

---

## Risk-proportional approach

Refactoring risk scales with:
- **Scope** — how much code changes
- **Test coverage** — how much of the change is verified
- **Coupling** — how many consumers depend on what is changing
- **Reversibility** — how easily the change can be undone if it causes a regression

Prefer:
- Small, incremental changes over large rewrites
- One structural boundary at a time over reorganising the whole system
- Changes with test coverage over changes without it
- Changes confined to a module boundary over changes that cross multiple boundaries

The smallest refactor that moves the code in the right direction is usually the right refactor. A 20% improvement that ships is worth more than a 100% improvement that never does.

---

## The "while I'm here" trap

The most common refactoring mistake is bundling structural changes with feature changes. This:
- Makes diffs unreadable
- Makes rollback impossible (which change caused the bug?)
- Makes reviews superficial (reviewer cannot focus on both the feature and the structure)

When a refactor is genuinely needed while adding a feature: separate commits, or a separate PR. The feature PR should contain only the feature. The refactor PR should contain only the refactor.
