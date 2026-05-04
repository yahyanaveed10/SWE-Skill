---
name: ai-collaboration
description: Guardrails for using AI-generated code, tests, and requirements. Use when reviewing AI-generated output before committing, identifying which types of AI generation are high-risk for hallucination, using AI to assist with test writing, or translating between informal requirements and testable specifications. Does not cover ML system design or MLOps (see ai-engineering).
---

# AI Collaboration

AI generation produces output that looks correct at the same confidence level regardless of whether it is correct. The failure modes are systematic, not random. Knowing them allows an agent — or a human using an agent — to apply the right level of verification to each type of output.

For AI generation hallucination modes see [generation-guardrails.md](generation-guardrails.md).
For AI assistance with testing see [ai-for-testing.md](ai-for-testing.md).
For AI assistance with requirements see [ai-for-requirements.md](ai-for-requirements.md).

## The core asymmetry

AI generation is fast. Verification is slow. The correct collaboration pattern is: generate fast, verify deliberately. The wrong pattern is: generate fast, skim fast, ship fast.

The asymmetry matters because:
- A wrong function with correct syntax costs more to debug in production than to verify at generation time
- A hallucinated API that does not exist is invisible until runtime
- A test that does not test what it claims to test passes CI and gives false confidence forever

## What AI generation is good at

- Producing syntactically correct code in a known pattern (boilerplate, CRUD, known idioms)
- Summarising existing code and identifying what it does
- Generating candidate tests for a known function signature
- Identifying candidates for refactoring (not deciding whether to refactor)
- Producing first drafts of documentation, comments, and specifications

## What AI generation is not good at

- Knowing which APIs exist vs. which ones sound like they should exist
- Understanding the actual runtime behaviour of the system (not what the code says, what the code does in practice)
- Distinguishing between what the requirements say and what the requirements mean
- Identifying what is missing (coverage gaps, edge cases not mentioned)
- Security-sensitive decisions (injection vectors, trust boundaries, privilege escalation paths)

For these, AI output is a starting point for human review, not a result.
