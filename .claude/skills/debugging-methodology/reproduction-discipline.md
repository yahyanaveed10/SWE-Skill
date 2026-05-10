# Reproduction Discipline

A bug you cannot reliably reproduce is a bug you cannot reliably fix. Most of the cost of debugging an intermittent or production-only bug is in establishing reproduction. Once you have a reliable reproduction, the fix is often quick.

---

## Why reproduction matters

**You cannot verify a fix without a reproduction.** If the bug only happened once last Tuesday and your "fix" never sees the trigger conditions, you do not know whether you fixed anything. The bug returns weeks later — possibly with the same fix already in place.

**Reproduction narrows the cause.** Establishing what conditions must hold for the bug to occur eliminates many alternatives. "Only fails on Tuesdays" — eliminates most code; investigate cron jobs, scheduled tasks, calendar logic. "Only fails when the user has more than 1000 records" — eliminates most code; investigate pagination, batch limits, integer overflow.

**Reproduction is documentation.** A reliable reproduction is the precise specification of the bug. It is the ground truth for the test that prevents regression.

---

## Making intermittent bugs deterministic

The most expensive bugs are the ones that happen sometimes. Strategies to make them deterministic:

**Identify the source of non-determinism.** Bugs are not actually random — they depend on inputs that vary. Common sources:
- Time (date logic, cache expiry, scheduled jobs, leap seconds)
- Concurrency (race conditions; see the concurrency skill)
- External state (database content, file system, network responses)
- Random number generators
- Order of iteration (Set/Map iteration order in some languages)
- Memory layout (heap addresses for security exploits)

**Control or fix the source.** Mock the clock. Seed the RNG. Pin the database state. Force a specific iteration order. Replace external calls with deterministic test doubles.

**Find the trigger.** If the bug happens 1 in 100 times, find what is different about the 1. Log enough state to compare a failing run to a passing one. The difference is the trigger.

**Run repeatedly.** If you cannot identify the trigger, run the test in a loop until it fails. Capture state at the moment of failure. (Slow but works for some race conditions.)

---

## Minimal reproduction cases

A reproduction is most useful when it is minimal — the smallest amount of code, data, and configuration that triggers the bug.

**Why minimal:**
- Easier to share for help (an issue with a 50-line reproduction gets answered; a 5000-line one does not)
- Easier to reason about — fewer moving parts to consider
- Reveals the cause — the irreducible kernel of the reproduction is usually adjacent to the bug

**How to minimise:** delete things until the bug stops reproducing, then put the last deletion back.

- Remove unrelated features one at a time
- Remove unrelated configuration
- Replace large input data with the smallest data that still triggers
- Remove dependencies (replace external API calls with mocks)
- Inline relevant code to remove indirection

The endpoint is when removing anything causes the bug to disappear.

---

## "Works on my machine" debugging

The bug reproduces in production but not locally. This is one of the most common debugging situations.

**Diff the environments.** What is different between your local environment and the failing one?
- Operating system (paths, line endings, file system case sensitivity, available system libraries)
- Runtime version (Node 18 vs. 20, Python 3.10 vs. 3.12)
- Dependency versions (lock files differ, transitive dependencies vary)
- Environment variables and configuration
- Available resources (memory limits, CPU count, disk space)
- Data (your local DB has 100 records; production has 100M)
- Concurrent load (your local has 1 user; production has 1000)
- Network conditions (your local has 1ms latency; production has cross-region)

**Bring the failing environment to your local machine.** Reproduce against a database snapshot. Use a container that matches production. Throttle your network. This is more efficient than trying to reason about differences abstractly.

**Or: bring the debugger to the failing environment.** Some bugs only appear at scale. Production-safe debugging tools (read-only queries, sampling profilers, distributed traces) let you investigate without bringing the failure to your laptop.

---

## Heisenbugs

Heisenbug: a bug that disappears or changes behaviour when you try to observe it.

**Common causes:**
- The debugger changes the timing of concurrent code, hiding the race condition
- Adding a `print` statement adds enough delay that the race condition no longer triggers
- Compiling with debug symbols changes optimisations and exposes (or hides) the bug
- The mere act of reading a variable in some languages forces evaluation of lazy code

**Debugging Heisenbugs:**
- Use less-invasive observation: structured logging that doesn't pause execution, sampling profilers, distributed traces
- Look for the cause based on what kind of observation makes the bug disappear (if `print` makes it disappear, the bug is timing-sensitive; if the debugger makes it disappear, the bug is timing-sensitive in a different way)
- Increase the probability of the race by adding artificial delays at suspect points

---

## Verification — the often-skipped step

Once you have a fix, verify it:

1. **Re-run the reproduction.** Bug should not occur.
2. **Revert the fix.** Bug should occur again. If it doesn't, your fix wasn't the fix — the bug was hidden by something else.
3. **Re-apply the fix.** Bug should not occur.

This three-step verification distinguishes "I changed something and the bug went away" (correlation) from "this change is the cause of the bug going away" (causation).

**Add a regression test.** The test should fail with the bug present and pass with the fix in place. Without the test, the next refactor or unrelated change can re-introduce the bug. Without verifying the test fails without the fix, you do not know that the test actually catches the bug.
