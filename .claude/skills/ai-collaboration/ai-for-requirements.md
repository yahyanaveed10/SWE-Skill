# AI for Requirements

How to use AI generation to translate informal requirements into usable engineering artefacts — and where the translation produces incorrect or misleading output.

---

## The requirements translation problem

Informal requirements ("the system should handle user uploads") are ambiguous. Engineers must resolve the ambiguity before implementation. The resolutions are where most bugs originate — not in the implementation of what was specified, but in the implementation of the wrong specification.

AI generation can assist with requirements translation, but it will resolve ambiguities. The resolutions will look confident and complete. If the resolutions are wrong, the resulting tests and code will be wrong in exactly the same way — and all of them will be consistent, which makes the error harder to find.

**The core risk:** AI-assisted requirements processing trades one failure mode (unresolved ambiguity) for another (confidently resolved ambiguity). The second failure mode is worse if the resolution is not verified.

---

## What AI can usefully do with requirements

**Enumerate implicit questions:** Given a vague requirement, AI can generate the list of questions that need to be answered before the requirement is implementable. This is more useful than having the AI answer the questions itself — the output is a structured list of unknowns for a human to resolve.

Example prompt framing: "Given this requirement, what are all the questions that need to be answered before I can implement it? Do not answer the questions — just enumerate them."

**Identify conflicting requirements:** AI can compare a new requirement against a set of existing requirements and flag potential conflicts. The flags need human verification, but the enumeration reduces the chance of missing a conflict.

**Translate to test cases (draft):** AI can produce draft acceptance tests from a stated requirement. These drafts require review to ensure they test the stated behaviour and not a superficial proxy. See [ai-for-testing.md](ai-for-testing.md).

**Summarise and restructure:** AI is good at reformatting requirements into consistent templates (user story format, given/when/then, etc.) without changing their meaning. The reformatting makes gaps more visible.

---

## What AI is not reliable for in requirements work

**Resolving domain ambiguity:** What does "near real-time" mean? What counts as a valid email address? What should happen if a payment fails halfway through? AI will generate a resolution that looks authoritative but reflects the statistical distribution of similar requirements in its training data — which may not match this system's business rules.

**Identifying missing requirements:** AI can list requirements it sees; it cannot reliably identify the requirements that are absent. Missing requirements are the most expensive bugs because they are not found by testing what was specified.

**Prioritisation:** Which requirements are essential vs. nice-to-have is a business and stakeholder decision. AI can restate requirements in priority order based on stated criteria, but it cannot determine the criteria from the requirements themselves.

**Compliance and regulatory requirements:** Legal, regulatory, and compliance requirements have precise meanings that vary by jurisdiction. AI-generated interpretations of GDPR, HIPAA, PCI-DSS, or similar requirements should not be treated as authoritative.

---

## Traceability

A requirement that cannot be traced to a test, and a test that cannot be traced to a requirement, are both problems:
- Untested requirements: you do not know if you built it
- Untraceable tests: you do not know what the test protects

If AI generates tests from requirements, maintain explicit links between the requirement statement and the generated tests. When a requirement changes, identify which tests need to change with it. This discipline is hard to maintain if the generation process is informal — record the requirement → test mapping explicitly.

---

## Verification of AI-assisted requirements work

After using AI to process requirements (translate, enumerate, restructure), verify:

**Coverage check:** For every stated requirement, is there at least one test case or acceptance criterion? Does the test case cover the stated behaviour and not just a structural property?

**Ambiguity check:** For every resolution the AI made (implicit in the generated tests or specification), is that resolution the intended business behaviour? Who can confirm it?

**Conflict check:** Do any of the generated artefacts contradict each other or contradict existing system behaviour?

**Completeness check:** What scenarios are not covered? (This requires human domain knowledge — AI cannot reliably enumerate what it is missing.)
