---
name: anti-patterns
description: Identifies and reasons through recurring software anti-patterns during code review and refactoring. Covers code-level smells (Blob/God Class, Functional Decomposition, Lava Flow, Spaghetti, Copy-Paste) and architectural smells (Golden Hammer, Design-by-Committee, Stovepipe, Big Ball of Mud). Use when reviewing existing code structure, planning a refactor, noticing a class or module is doing too much, or evaluating whether a design is causing maintenance pain. Does not cover performance-specific anti-patterns (N+1, premature optimisation) or security anti-patterns (hardcoded secrets).
---

# Anti-Patterns

An anti-pattern is a recurring solution that appears to work but generates decidedly negative consequences. The point is not "don't do this" — it's "you may not realise you're doing this, and it doesn't work."

For code-level smells see [code-smells.md](code-smells.md).
For architecture-level smells see [architectural-smells.md](architectural-smells.md).
For when to refactor vs leave it see [refactoring-signals.md](refactoring-signals.md).

## Root causes

Most anti-patterns trace to one of seven failure modes. Recognising the root cause matters because it determines whether the fix is technical (restructure the code) or organisational (change how decisions are made).

**Haste** — Shortcuts taken under time pressure. Technical debt accumulated by design, not accident.

**Apathy** — Known problems not addressed. "It works, don't touch it."

**Narrow-mindedness** — Refusing to consider approaches outside familiar territory. Often appears as reinventing what libraries already provide.

**Sloth** — Taking the easiest path over the correct one. Tends to produce incremental complexity that feels small per commit but compounds over months.

**Avarice (excessive complexity)** — Over-engineering. Too many abstractions, too much modelling of details that don't matter yet.

**Ignorance** — Lack of domain knowledge that leads to solving the wrong problem or solving the right problem with a known-bad approach.

**Pride (Not-Invented-Here)** — Refusing to adopt external solutions. Spending engineering time on problems already solved.

When you identify an anti-pattern, ask which root cause is active — that's what needs to change, not just the code.

## How to read each pattern

Each entry follows this structure:

**Signal** — how to detect it in code or design  
**Root cause** — which failure mode produces it  
**Ask** — reasoning prompts to confirm it and understand scope  
**Trade-off** — what you give up by fixing it vs leaving it  
**When to stop** — conditions under which refactoring is not the right move
