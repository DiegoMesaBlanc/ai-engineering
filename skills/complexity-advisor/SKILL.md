---
name: complexity-advisor
description: Analyze task complexity and determine the simplest appropriate implementation strategy while avoiding unnecessary abstractions, refactoring, and architectural changes, and selecting design patterns from the task's design forces.
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

Do not add an abstraction or a pattern without a design signal; record the
selection as 0 patterns when no force is present.

## Low

Examples:

- isolated component change
- simple endpoint modification
- localized validation
- small bug fix

Recommendation:

Prefer the existing architecture and a simple implementation. Check the
existing mechanisms first, and select only the pattern that a present signal
requires.

## Medium

Examples:

- multiple modules affected
- multiple implementations of a behavior
- non-trivial state flow
- new integration
- meaningful domain logic

Recommendation:

Analyze existing abstractions and apply `references/design-patterns.md` to
the signals that are actually present. Do not introduce a pattern without a
signal; the count is derived.

## High

Examples:

- architectural change
- cross-cutting functionality
- complex concurrency
- major integration
- substantial domain modeling

Recommendation:

Perform explicit architecture analysis and the full pattern-selection
procedure: profile, signals, compositions, conflicts, stack translation,
and alternatives.

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

## Proportional rigor (SDD)

Complexity determines the required planning rigor. It does not by itself
authorize or block work.

| Complexity | Rigor | Planning depth |
|---|---|---|
| Trivial, Low | R1 Brief | objective, scope, brief verifiable criteria, validation |
| Medium | R2 Moderate | criteria, solution design, planned tests, risks |
| High | R3 Complex | explicit expected behavior, technical decisions, test strategy, risks |
| Critical | R4 High-risk | R3 plus a validation and rollback strategy and security, data, and concurrency focus |

Recommend the rigor level for the plan. The procedure and the required
content per level are defined in `workflows/task-planning.md`; the retained
result belongs to `skills/task-context/SKILL.md`.

The rigor level never triggers human approval by itself. Approval is
triggered by the Human Decision Gates in AGENTS.md: risk, irreversible or
high-impact operations, security or data impact, cost, scope or
backward-compatibility changes, shared or production resources, and
project rules.

## Pattern selection scales with rigor

The pattern procedure itself is owned by `skills/architecture-advisor/SKILL.md`
("Design Patterns") and `references/design-patterns.md`. This section defines
only how much analysis each complexity level requires.

Complexity scales the depth of the pattern analysis, never the evidence bar.
The task's **design forces** are the only input; the count is an output.

| Complexity | Pattern analysis |
|---|---|
| Trivial | No pattern unless a signal is explicit; usually 0. |
| Low | Check existing mechanisms; select only the patterns one signal requires. |
| Medium | Apply the signal matrix; record selected and rejected candidates. |
| High | Full procedure: profile, signals, compositions, conflicts, stack translation, alternatives. |
| Critical | High plus reliability, security, data-consistency, and rollback patterns, with approval. |

A task may need 0, 2, 32, or 100+ patterns. The complexity level never caps
the set and never pads it.

## Quality is not proportional

Complexity and rigor govern only the depth of planning and testing. They
never lower the quality of the delivered code.

Trivial and Low tasks still require professional implementation: existing
conventions, clean and cohesive code, explicit error handling, boundary
validation, type safety, security, and tests for changed behavior. Prefer a
direct implementation, not a careless one. Never implement a task in a
junior way because it looks simple.

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

## Refactoring and Abstraction Justification

Before extracting a new component, module, function, or abstraction, verify that at least one justification threshold is met with evidence:

1. **Reuse** — the extracted unit is used in ≥2 real call sites now (not hypothetically).
2. **Size/complexity** — the unit exceeds the **project's own** established threshold (e.g., a component >300 lines, a function >50 lines when the project has no convention, a module exceeding the project's typical size).
3. **Required change** — the task itself changes the unit, and the change is cleaner with the split.
4. **Testability** — the original cannot be adequately tested without the split (evidence required).

**Type/contract extraction**:
- Extracting a shared contract/type used across ≥2 modules/layers meets the reuse threshold.
- Extracting single-consumer props, local view models, or private helper types does **not** meet the threshold; keep co-located.
- Language idiom overrides file-count aesthetics (e.g., Go consumer-side interfaces, Java one-public-type-per-file, Python Protocols).

Negative triggers (do NOT extract/split):
- Single call site or no reuse.
- No threshold exceeded; purely aesthetic or "clean code" rationale.
- Barrel export or alias introduced solely for import tidiness with no consumer benefit.
- The working unit is cohesive and used in one place.

If no threshold is met, do not perform the refactor. Record the observation as an Engineering Insight instead.
