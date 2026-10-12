# Task to Code Workflow

## Goal

Transform an engineering task and available design context into a validated
implementation on a user-provided task branch.

## Phase 1 — Detect

Determine:

- project state
- repository
- task provider
- design provider
- Git provider
- current branch
- working-tree status

## Phase 2 — Understand the Task

When a task reference is available:

- retrieve the task
- extract business intent
- extract functional requirements
- identify acceptance criteria
- inspect dependencies and comments
- identify design references

Load Task Context only when relevant.

## Phase 3 — Analyze the Design

When design context exists:

- identify the relevant frame, page, component, or region
- retrieve the smallest useful design context
- map design requirements to existing components
- identify missing information

Do not assume Figma.

## Phase 4 — Analyze the Repository

For an existing project:

- inspect repository structure
- detect language and framework
- detect application architecture
- detect monorepo or multi-repository organization
- detect microfrontend composition when present
- identify conventions and reusable abstractions

For a new project:

- determine the minimum viable architecture
- establish repository structure
- determine whether a workspace is justified
- evaluate whether microfrontends provide a concrete benefit

Do not introduce a monorepo or microfrontends without evidence.

## Phase 5 — Ask for the Branch

Before implementation, ask for the exact branch name if it was not provided.

Do not generate the name automatically.

## Phase 6 — Prepare the Branch

- validate the name
- detect branch collisions
- identify the correct base
- preserve uncommitted work
- create the branch or isolated worktree
- verify the active working location

Do not implement until the requested branch is ready.

## Phase 7 — Complexity and Architecture

Evaluate:

- task complexity
- affected modules
- repository strategy
- frontend composition
- architecture impact
- relevant design patterns
- risks and testing needs

Use only the analyses relevant to this task.

Require human approval for significant architectural changes.

## Phase 8 — Plan

Provide a concise implementation plan including:

- files and modules affected
- existing abstractions to reuse
- proposed new components only when justified
- validation
- risks

## Phase 9 — Implement

Implement on the requested branch.

Preserve existing conventions.

Do not expand the scope silently.

## Phase 10 — Validate

Load `skills/testing-strategy/SKILL.md`.

1. Inspect the current changes and project tooling.
2. Determine validation risk.
3. Select the smallest sufficient validation plan.
4. Run relevant checks using existing project commands.
5. Broaden validation when risk, failure evidence, or project policy
   requires it.
6. Record commands, outcomes, and limitations.
7. Report failures and checks that were not executed.
8. Never claim that validation passed without evidence.

Prefer focused tests for changed behavior before running broad suites.

Use existing test frameworks, type checkers, linters, formatters, build
commands, and configured security tools.

For monorepos, target the affected project or packages when reliable
workspace tooling is available.

Never install dependencies, add frameworks, change configuration, or
execute potentially destructive validation commands without appropriate
authorization.

Preserve unrelated and uncommitted user changes.

A passing build alone does not establish completion.

## Phase 11 — Review

Review the resulting implementation for:

- correctness
- requirements
- architecture
- security
- performance
- maintainability
- testing
- scope

## Phase 12 — Git Delivery

Inspect:

- current branch
- status
- diff
- changed files
- unrelated changes

Propose a commit message.

Create a commit only under the established human-approval policy.

Do not push automatically.

## Phase 13 — PR Preparation

When requested, generate:

- copy-ready PR title
- copy-ready PR description
- testing summary
- risks and limitations
- reviewer notes

The user creates the PR/MR manually.

Never create or publish the PR/MR.

## Phase 14 — Engineering Insights

After implementation and validation, report meaningful engineering
observations related to the task.

Include only observations with practical value, such as:

- relevant improvement opportunities
- technical debt discovered during the task
- reusable abstractions worth considering
- architectural risks
- testing gaps
- security or performance concerns

Separate these observations from the task deliverable.

Do not implement unrelated improvements automatically.

Explain the expected benefit and relevant trade-offs.

Keep the report concise and proportional to the task complexity.
