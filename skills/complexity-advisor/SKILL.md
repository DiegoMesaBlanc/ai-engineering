---
name: complexity-advisor
description: Analyze task complexity and determine the simplest appropriate implementation strategy while avoiding unnecessary abstractions, design patterns, refactoring, and architectural changes.
---

# Complexity Advisor

## Complexity levels

Classify tasks as:

- Trivial
- Low
- Medium
- High
- Critical

## Trivial

Examples:

- typo
- simple configuration change
- isolated UI text
- obvious one-line fix

Recommendation:

Prefer direct implementation.

Do not introduce abstractions or patterns.

## Low

Examples:

- isolated component change
- simple endpoint modification
- localized validation
- small bug fix

Recommendation:

Prefer the existing architecture and a simple implementation.

## Medium

Examples:

- multiple modules affected
- multiple implementations of a behavior
- non-trivial state flow
- new integration
- meaningful domain logic

Recommendation:

Analyze existing abstractions and consider appropriate patterns.

Do not automatically introduce one.

## High

Examples:

- architectural change
- cross-cutting functionality
- complex concurrency
- major integration
- substantial domain modeling

Recommendation:

Perform explicit architecture analysis.

Present alternatives and trade-offs.

Require human approval before major architectural changes.

## Critical

Examples:

- security-sensitive architecture
- financial transaction flow
- migration with significant production risk
- distributed consistency
- irreversible infrastructure changes

Recommendation:

Require explicit human approval.

## Core question

Before introducing abstraction, ask:

"Does this abstraction eliminate more complexity than it introduces?"

## Decision priority

Prefer:

1. Existing project abstraction
2. Simple local implementation
3. Existing framework capability
4. Design pattern
5. New architectural abstraction

## Scope control

Do not expand the task merely because an improvement is visible.

Record unrelated improvements as Engineering Insights instead.
