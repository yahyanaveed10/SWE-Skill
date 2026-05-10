---
name: debugging-methodology
description: The discipline of finding root causes — hypothesis formation, bisection, reproduction. Use when investigating any bug or unexpected behaviour, when a fix is not converging after multiple attempts, when a bug only happens sometimes, when production behaves differently from local, or when an agent has been editing the same file repeatedly without solving the underlying problem. Covers the methodology, not the tooling — applicable across debuggers, profilers, and log analysers. Distinct from the /debug command (which is a process); this is the judgment.
---

# Debugging Methodology

Debugging is applied science: form a hypothesis about what is wrong, design a test that distinguishes hypothesis from alternatives, observe the result, update the hypothesis. Most debugging takes longer than necessary because the engineer skips one of these steps — usually the hypothesis or the test design.

The most common failure mode in agent debugging: editing code based on plausible-looking explanations without testing whether each edit changed anything. Five edits later, the bug is still there and the engineer cannot remember which edits were undone and which still matter.

For hypothesis formation and bisection see [hypothesis-and-bisection.md](hypothesis-and-bisection.md).
For making bugs reproducible before fixing see [reproduction-discipline.md](reproduction-discipline.md).
For debugging in production without a local reproduction see [production-debugging.md](production-debugging.md).

## The hard rules

**Reproduce before you fix.** A bug you cannot reliably reproduce is a bug you cannot reliably claim to have fixed. The reproduction itself is often half the debugging — narrowing the conditions that trigger the bug usually narrows what is causing it.

**Form a specific hypothesis before changing code.** Not "let me try this." Something like: "I believe the bug is X. If X is true, then Y will happen. If I change Z, the bug will go away." If the change does not produce the predicted result, the hypothesis was wrong — that is information, not failure. If you do not have a hypothesis, you are not debugging — you are guessing.

**Change one thing at a time.** When multiple changes happen simultaneously and the bug goes away, you do not know which change fixed it. The bug will return when you keep an unrelated change and discard the actual fix.

**Verify the fix actually fixed it.** Reverting the suspected fix should bring the bug back. If reverting the fix has no effect, the fix was not the fix; the bug is still present and will resurface.

## The signal that you have left debugging and entered guessing

- You have made multiple code changes without running a test in between
- You cannot articulate what hypothesis the most recent change tested
- You are making changes based on "this looks suspicious" rather than evidence
- You have spent more than 30 minutes without narrowing the search space

When any of these is true, stop. Revert. Restart with a deliberate hypothesis.

## The methodology in one paragraph

Reproduce the bug reliably. Observe what is happening (logs, debugger, prints). Form a hypothesis that explains the observation. Identify a test — usually changing one variable — that would distinguish your hypothesis from alternatives. Run the test. Observe the result. Update the hypothesis. Repeat until you have isolated the cause to a specific line or specific input. Fix it. Verify the fix removes the bug and reverting the fix brings the bug back.

Everything else in this skill is detail on how to execute this loop well.
