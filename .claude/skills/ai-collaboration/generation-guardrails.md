# Generation Guardrails

Hallucination modes in AI code generation and the signals that identify each. These are systematic failure patterns, not random errors — knowing them allows targeted verification rather than reviewing everything equally.

---

## API hallucination

**What it is:** The model generates calls to functions, methods, or endpoints that do not exist in the library or framework being used, or that have different signatures from what the model generated.

**Why it happens:** The model's training data contains many similar-looking API calls. It generalises the pattern and generates a plausible-but-wrong variant.

**High-risk contexts:**
- Less common libraries with sparse training data
- Libraries that changed APIs between versions (the model may have learned an older version)
- Internal or proprietary APIs the model has not seen
- Combining two libraries in an unusual way

**Verification approach:** For any generated function call that you do not recognise from memory, check the actual library documentation or source. Do not assume it exists because it looks reasonable.

---

## Logic hallucination

**What it is:** The code compiles and runs but produces incorrect results for inputs outside the obvious happy path. The logic is plausible but wrong.

**High-risk contexts:**
- Off-by-one conditions (loop bounds, slice indices, range endpoints)
- Edge cases the prompt did not mention (empty input, null, zero, max value)
- Concurrency conditions (race conditions, non-atomic read-modify-write)
- Floating point comparisons
- Date and time arithmetic (timezone handling, daylight saving, leap years)

**Verification approach:** Do not read the code and decide it looks right. Execute it against boundary cases. The model generates code that looks correct — looking correct is not the same as being correct.

---

## Security hallucination

**What it is:** The model generates code that contains security vulnerabilities — SQL injection, path traversal, missing authentication, exposed secrets — because it is completing a pattern without reasoning about trust boundaries.

**High-risk contexts:**
- Any code that takes user input and uses it in a database query, file path, shell command, or HTML output
- Authentication and authorisation logic
- Cryptography (key generation, encryption, hashing)
- Any code the prompt described as "for testing" or "just to get it working" — these patterns get promoted to production

**Verification approach:** Do not rely on the model to identify security issues in its own output. Apply the security-engineering skill's threat model to any generated code that handles external input or makes access control decisions.

---

## Dependency hallucination

**What it is:** The model generates an import or package name that does not exist, or that exists but has a different interface from what was generated.

**High-risk contexts:**
- Small or niche packages
- Packages with similar names to well-known packages (typosquatting-adjacent names)
- Python packages where the PyPI name differs from the import name (common source of confusion)

**Verification approach:** Before installing or using a generated dependency, verify it exists in the package registry, check its download count and maintenance status, and confirm the API matches what was generated.

---

## Context loss hallucination

**What it is:** In a long context, the model forgets constraints or requirements stated earlier in the conversation or codebase. Later generated code contradicts earlier decisions.

**High-risk contexts:**
- Long conversations with many intermediate steps
- Large codebases where the full context is not in the generation window
- Requirements that were added or refined during the conversation

**Verification approach:** For multi-step generation tasks, explicitly re-state the constraints at the point of generation. Do not assume the model is tracking all decisions made earlier in the conversation.

---

## Test correctness hallucination

**What it is:** The model generates tests that pass but do not test what they claim to test. Common forms:
- Test asserts a value that is always true regardless of the function's behaviour
- Test does not call the function under test
- Test uses mocks so aggressively that it tests nothing real
- Test input is chosen to hit the happy path, missing all boundary cases

This is distinct from the general hallucination modes above — test correctness hallucination is the most dangerous because it produces green CI without coverage. See [ai-for-testing.md](ai-for-testing.md) for detail.

---

## Verification calibration

Not all AI-generated output requires equal verification effort. Calibrate by risk:

**High verification burden:**
- Security-sensitive code (auth, input handling, cryptography)
- Database operations (especially writes and deletes)
- Code with complex logic and multiple edge cases
- Code using unfamiliar or niche libraries

**Lower verification burden:**
- Syntactic transformations (renaming, reformatting, type conversion)
- Boilerplate in well-known frameworks
- Documentation and comment drafts (easy to review, low operational risk)
- Code with obvious test coverage (the test tells you if the code is wrong)
