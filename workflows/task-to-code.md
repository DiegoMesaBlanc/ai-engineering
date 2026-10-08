# Task to Code Workflow

## Goal

Transform an engineering task and its available design context into a
validated implementation.

---

## Phase 1 — Detect

Determine:

- project state
- repository
- task provider
- design provider
- Git provider

---

## Phase 2 — Understand Task

Load:

`skills/task-context/SKILL.md`

Extract:

- business intent
- functional requirements
- acceptance criteria
- constraints
- dependencies
- design references
- technical references

---

## Phase 3 — Analyze Design

When a design reference exists:

Load:

`skills/design-analysis/SKILL.md`

Map:

Task
→ Design
→ Existing Components
→ Required Changes

---

## Phase 4 — Analyze Project

For existing projects:

- inspect repository
- detect stack
- determine architecture
- identify conventions
- identify reusable abstractions

For new projects:

- determine minimum viable architecture
- initialize project
- validate initial project
- continue with this workflow

---

## Phase 5 — Complexity

Determine:

- complexity
- risk
- affected modules
- architecture impact
- testing needs

---

## Phase 6 — Architecture

Only if justified:

- evaluate architecture
- evaluate design patterns
- determine reuse strategy
- identify risks

---

## Phase 7 — Plan

Create an implementation plan.

Include:

- affected files/modules
- reusable abstractions
- new components/services only when necessary
- tests
- validation
- potential risks

---

## Phase 8 — Human Decision

Ask only for unresolved meaningful decisions.

---

## Phase 9 — Implement

Implement according to:

- repository conventions
- selected architecture
- existing abstractions
- task requirements
- design requirements

---

## Phase 10 — Validate

Run appropriate:

- tests
- typecheck
- lint
- build
- security checks

---

## Phase 11 — Engineering Review

Review the resulting implementation for:

- correctness
- architecture
- maintainability
- testing
- security
- performance
- scope

---

## Phase 12 — Git

Inspect:

- status
- diff
- commits
- unrelated changes

Prepare a commit proposal.

Create commit only after approval.

---

## Phase 13 — Pull Request

When requested:

- detect provider
- prepare PR
- include task context
- include relevant design context
- include validation results
- request human approval
- create PR