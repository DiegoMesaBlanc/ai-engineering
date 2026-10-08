---
name: testing-strategy
description: Determine and execute the appropriate testing strategy for a software change based on the detected stack, project conventions, risk, and task complexity.
---

# Testing Strategy

## Principles

Tests must reflect actual project behavior.

Prefer existing testing frameworks.

Do not introduce a new testing framework without justification.

## Determine

- existing unit test framework
- integration test framework
- E2E framework
- coverage tooling
- test conventions
- test commands

## Test selection

Use the smallest sufficient test strategy.

Trivial changes:

Run relevant validation only.

Low complexity:

Add or update focused tests.

Medium complexity:

Include unit and integration coverage when appropriate.

High/Critical:

Use broader validation and regression testing.

## Validation

Before declaring completion:

1. Run relevant tests.
2. Run type checking when available.
3. Run linting when available.
4. Run build validation when relevant.
5. Review failures.
6. Never claim success without evidence.
