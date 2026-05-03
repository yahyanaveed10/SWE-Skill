# Code Smells

Code-level anti-patterns. Each one describes a failure mode visible at the class or module level.

---

## Blob / God Class

**Signal:** One class owns the majority of the system's logic. Surrounding classes are mostly data containers with few methods. The class diagram has one large node and many small ones orbiting it.

**Root cause:** Sloth, haste. Developers add logic to the class that is already working rather than finding the right home for new behaviour.

**Ask:**
- What single responsibility does this class have? If you cannot state it in one sentence without "and", it has too many.
- Which of the data classes orbiting it could own some of this behaviour instead?
- Is this a controller that grew, or was it always designed to be central?
- What would break if you extracted one piece of logic out?

**Refactoring direction:** Move behaviour toward the objects that own the relevant data. The goal is not just splitting the file — it's identifying whose responsibility each piece of logic actually is. A split that produces two Blobs is not a fix.

**Watch for:** Hidden coupling through shared mutable state. After extraction, the new classes may still communicate through the same data structures, preserving the coupling in a less visible form.

**Trade-off:** Extraction increases the number of files and interaction points. On a small team or a short-lived project, a manageable Blob may be less expensive than a premature decomposition.

**When to stop:** If the logic genuinely belongs together (a Facade over a complex subsystem, for example), leave it. Not every large class is a Blob. Ask whether the size reflects accidental accumulation or deliberate encapsulation.

---

## Functional Decomposition

**Signal:** In an object-oriented codebase, classes are named after actions rather than concepts — `DataProcessor`, `FileHandler`, `RequestManager`. Each class has one method, often called `execute()` or `run()`. The structure resembles procedural code wearing OO clothing.

**Root cause:** Avarice, sloth. Developers with procedural backgrounds map each function to a class rather than modelling domain concepts.

**Ask:**
- What domain concept does this class represent? If the answer is "the act of doing X", that is a function, not a class.
- Are there data classes with no behaviour? Are there behaviour classes with no data? This split is the tell.
- Could these "action classes" be methods on the objects they operate on?

**Refactoring direction:** Build an analysis model first — understand what the code is trying to represent from the user's perspective. Then identify which data objects could own the behaviour currently living in the action classes. The refactoring is driven by domain understanding, not by class count.

**Trade-off:** Reorganising a large functional-decomposition codebase is high-risk because the structure is often deeply embedded in the call hierarchy. Verify with tests before moving anything.

**When to stop:** Some architectures (pipeline, command pattern) legitimately use action objects. Distinguish between deliberate use of a pattern and accidental procedural structure.

---

## Lava Flow

**Signal:** Dead code, commented-out blocks, classes nobody calls, configuration flags that are always the same value, experimental code from a prototype that shipped. Nobody removes it because nobody knows what it does or whether it is still needed.

**Root cause:** Haste (prototype shipped), apathy (nobody cleaned up), ignorance (original authors left).

**Ask:**
- Is this code reachable from any live path? Check call graphs, not just grep.
- Does removing it break any tests? If there are no tests, that is also a signal.
- Is there a comment explaining why it exists? If yes, is the reason still valid?

**Refactoring direction:** Remove confidently when you can confirm unreachability. For ambiguous cases: add a log statement or metric for 30 days and confirm zero hits before deleting. Do not comment out — remove.

**Trade-off:** Every piece of dead code you keep is future maintenance surface. Every piece you remove carelessly is a potential production incident. The cost of confirmation is lower than either.

**When to stop:** If the code is behind a feature flag that is intentionally off in production but active in staging, it is not dead — it is dormant. Document it clearly and move on.

---

## Spaghetti Code

**Signal:** Control flow is impossible to follow without a debugger. Functions call each other in circles or across unrelated modules. Global state is modified in multiple places. The execution path to produce a result cannot be described in a sentence.

**Root cause:** Haste, ignorance. Each individual change made sense locally; the aggregate structure was never owned.

**Ask:**
- Can you describe the entry point and the sequence of state changes from request to response?
- How many modules does a single user action touch? If it is more than five or six, ask why.
- Is there shared mutable state that multiple code paths read and write?

**Refactoring direction:** Define and enforce module boundaries first — separate concerns before cleaning up within a concern. Trying to clean up spaghetti code without first defining structure produces cleaner spaghetti.

**Trade-off:** Refactoring entangled code without tests is high risk. The first investment is test coverage of the current (broken) behaviour, then structural improvement.

**When to stop:** If the system works and no new features are being added to the affected area, the risk of refactoring may outweigh the benefit. Lava flow and spaghetti often coexist — address the one in the path of active change first.

---

## Copy-Paste Programming

**Signal:** Nearly identical blocks of code appear in multiple places. Logic is duplicated across files, services, or functions with minor variations — different variable names, slightly different conditions.

**Root cause:** Sloth, narrow-mindedness. Copying is faster than abstracting, until it is not.

**Ask:**
- How many times does this logic appear? Two is a warning, three is a pattern worth addressing.
- What varies between the copies? Is the variation incidental (naming) or essential (actual behaviour difference)?
- If the shared logic has a bug, how many places need to change?

**Refactoring direction:** If the variation is incidental, extract into a shared function or module. If the variation is essential (the copies genuinely do different things that happen to look similar), leave them separate — forced abstraction over genuinely different concerns produces worse code than duplication.

**Trade-off:** Abstraction introduces indirection. The right question is: does the cognitive cost of indirection outweigh the maintenance cost of duplication? For logic that changes together, extract. For logic that looks the same but changes independently, do not.

**When to stop:** Two copies is not always a problem. The rule of three is a reasonable heuristic: when a third copy appears, the abstraction boundary is becoming clear. Extract then.
