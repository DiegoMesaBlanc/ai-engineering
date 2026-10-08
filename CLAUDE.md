# AI Engineering System

This repository is the canonical source of truth for my global AI Engineering System.

## Core principles

- Understand before modifying.
- Inspect before assuming.
- Detect the existing technology stack automatically.
- Detect the existing architecture before proposing changes.
- Reuse existing abstractions whenever appropriate.
- Prefer the simplest solution that adequately solves the problem.
- Do not introduce design patterns without a justified benefit.
- Do not introduce architectural complexity without measurable value.
- Do not refactor unrelated code unless explicitly requested or required for correctness.
- Preserve existing project conventions unless there is a justified reason to change them.
- Validate implementations with the project's available quality gates.
- Treat tests as part of implementation.
- Treat security as part of implementation.
- Keep documentation synchronized with the implementation.

## Language policy

Human interaction may be in Spanish or English.

All source code must use English.

All identifiers must use English.

All code comments must use English.

All JSDoc and TSDoc must use English.

Technical documentation language is selected according to the project or user request.

## Architecture policy

Architecture decisions must be proportional to project complexity.

Never introduce an abstraction only because a design pattern exists.

Before introducing a pattern, compare:

1. Existing project abstraction
2. Simple local solution
3. Existing framework capability
4. Design pattern
5. Architectural change

Prefer the least complex option that correctly solves the problem.

## Human-in-the-loop

Architectural changes, major refactors, technology changes, and significant increases in complexity require human approval before implementation.

## Skill usage

Load Skills only when their capabilities are relevant to the current task.

Do not load unrelated technology-specific knowledge.

## Git

Never discard user changes.

Never rewrite history unless explicitly requested.

Never force push unless explicitly requested.

Prefer small, logical, atomic commits.

Commit messages must accurately describe the actual change.

Before creating a commit, inspect the working tree and verify the staged changes.

## Task and Design Intelligence

When a development task is available from an external or local task
provider:

1. Read the task before implementation.
2. Extract business intent and acceptance criteria.
3. Identify design references.
4. Analyze relevant design artifacts when available.
5. Inspect the repository.
6. Map task and design requirements to existing code.
7. Prefer existing abstractions over new ones.
8. Do not invent missing requirements.
9. Ask only materially necessary questions.

A task source is not mandatory.

A design source is not mandatory.

The engineering system must work when either or both are unavailable.

---

## Pull Request Review

A Pull Request must not be reviewed only from its diff.

Review the:

- task
- design
- repository
- architecture
- source branch
- target branch
- exact PR source commit
- tests
- integration impact

Use an isolated workspace or Git worktree when possible.

Never replace the user's active workspace.

Review findings must be evidence-based and actionable.

Do not generate large numbers of low-value comments.

Prioritize correctness, security, architecture, requirements, and meaningful
maintainability issues over personal style preferences.