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

Run the relevant project checks:

- tests
- typecheck
- lint
- build
- security checks when appropriate

Report actual results and unexecuted checks honestly.

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
