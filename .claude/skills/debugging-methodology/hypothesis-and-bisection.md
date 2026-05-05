# Hypothesis Formation and Bisection

The two highest-leverage debugging techniques. Both work by **narrowing the search space**.

---

## The hypothesis loop

A debugging session is a series of hypotheses, each tested against observation. The cycle:

1. **Observe** — what is actually happening? (Not what the code is supposed to do — what it is doing.)
2. **Hypothesise** — propose a specific cause that would explain the observation.
3. **Predict** — if the hypothesis is true, what else should be true? What change would eliminate the bug?
4. **Test** — make the change or run the experiment.
5. **Update** — was the prediction correct? Refine, replace, or commit to the hypothesis.

**A good hypothesis is specific, testable, and rules out alternatives.** Not "something is wrong with the database" — "the connection pool is exhausted because connections are not being released after the timeout path."

**A good test distinguishes between hypotheses.** If both your current hypothesis and a leading alternative would produce the same test result, the test is not informative. Design the test so the result tells you which is true.

---

## When you have no hypothesis

If you cannot articulate a specific hypothesis, you do not yet have enough information. Stop forming hypotheses — go observe more.

**Useful observation strategies:**
- Add logging at multiple points along the code path; reproduce; read the logs in order
- Use a debugger to step through the failing path
- Diff the behaviour against a working case (a passing test, an earlier commit, a different environment)
- Read the actual error message and stack trace fully, not just the headline

**The signal that you do not have enough information:** every hypothesis you form is equally plausible to its alternatives. You cannot decide which to test first. Get more observations before continuing.

---

## Bisection — the most powerful debugging technique

Bisection halves the search space at each step. Whenever the search space is large and ordered, bisection is the right tool.

**Bisect on time (when did it break?):** `git bisect`. Start with a known-good commit and a known-broken commit. Git checks out the midpoint; you test; you mark it good or bad; git picks the next midpoint. In log₂(N) steps, you have found the breaking commit.

```bash
git bisect start
git bisect bad HEAD
git bisect good v1.2.0
# git checks out the midpoint; you test
git bisect good   # or: git bisect bad
# repeat until git identifies the commit
git bisect reset
```

**Bisect on code (where in the code is it failing?):** comment out half the suspect code; reproduce. If the bug remains, it is in the half that is still active. Halve again.

**Bisect on data (which record triggers the bug?):** if a batch job fails on one of 10000 records, halve the input. Run on the first 5000. If it still fails, the bad record is in the first 5000. Continue.

**Bisect on configuration (which setting is causing the bug?):** if the bug appears with one config and not another, identify the differing keys. Half by half, find the responsible key.

**The discipline of bisection:** the test at each step must be deterministic. If your test sometimes passes and sometimes fails on the same code, bisection produces wrong results. Make the test reliable before bisecting.

---

## Hypothesis traps

**The "I know what the bug is" trap.** Confirmation bias: you have a hypothesis, you find evidence consistent with it, you commit to a fix. The fix does not work. The actual bug was elsewhere; the evidence was incidental. Fix: design tests that would *disprove* the hypothesis, not just confirm it.

**The "this code looks wrong" trap.** Code that looks suspicious is not necessarily related to the current bug. Editing code because it looks bad is refactoring, not debugging — do it on a different branch and verify each change is unrelated to the bug fix.

**The "the framework / library / OS has a bug" trap.** Almost always wrong. Mature frameworks are tested by millions of users; your bug is almost certainly in your code, not theirs. Only after exhausting your code as the cause is "the framework has a bug" a reasonable hypothesis. Even then, search the project's issue tracker before assuming.

**The "I'll just change this and see" trap.** This is the failure mode that wastes the most time. Each change made without a hypothesis is a guess. Five changes later, the bug may be hidden by a side effect of one of them, or revealed by a different combination, and you no longer know which change was the actual fix. Discipline: revert all changes between hypothesis tests.

---

## Working backward from the symptom

If the symptom is X, what *could* cause X? Enumerate possibilities. Narrow by elimination.

Example: "API request returns 500."
- Possible causes: application code threw an exception; downstream service errored; database connection failed; request timed out; load balancer misrouted.
- Observation: stack trace in logs shows a NullPointerException in `processOrder`.
- Narrowed: application code threw an exception. Specifically in `processOrder`.
- Sub-hypothesis: an order field expected to be non-null was null.
- Observation: the failed request had `customer_id = null`.
- Cause: the new validation that should have rejected null `customer_id` was added but does not run for this code path.

Each step narrows the search. The discipline: do not jump to the fix until you have followed the chain to the actual cause.

---

## When to stop bisecting

You have not necessarily found the root cause when you have found the failing line. The failing line may be the *symptom* of a deeper cause: a missing initialisation, a wrong assumption, a race condition.

**Ask after locating the failing line:** why did this line fail? What invariant was violated? Could the invariant violation happen in other code that calls this function?

The fix at the failing line is often correct, but the deeper cause may produce other bugs in other places. Decide deliberately whether to fix the symptom or the cause — not by accident.
