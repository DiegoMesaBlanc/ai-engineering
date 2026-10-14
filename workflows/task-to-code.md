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
- inspect dependency manifests, lockfiles, and installed direct dependencies
- detect microfrontend composition when present
- identify conventions and reusable abstractions

For a new project:

- determine the minimum viable architecture
- establish repository structure
- determine whether a workspace is justified
- evaluate whether microfrontends provide a concrete benefit
- establish a formatter and linter with reproducible commands, following
  the Formatting and Linting Policy in AGENTS.md

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

## Phase 7 — Intent, Alignment and Architecture

Identify the engineering intent from the request:
- Architecture or solution design.
- Time and cost estimation.
- Requirements analysis or technical investigation.
- Software implementation.
- Validation, debugging, or code review.

Apply intent-specific behavior:
- Architecture-only or estimation-only requests: do not start implementation or modify application source code unless separately authorized. Produce proportionate analysis (options, trade-offs, assumptions, uncertainty, risks, unresolved questions). When moving to implementation later, reuse this work.
- Other modes: proceed.

If software implementation:

1. Understand business objective, task/design/repo context.
2. Identify acceptance criteria, constraints, non-goals, material unknowns.
3. Infer routine technical details from codebase, conventions, official docs, policies.
4. Ask concise questions only for important business requirements missing/conflicting/cannot be inferred. Do not ask about choices inferrable from evidence or already available.
5. Present concise summary of agreed behavior and proposed approach.
6. Confirm alignment before implementation.

Evaluate:

- task complexity
- affected modules
- repository strategy
- frontend composition
- architecture impact
- relevant design patterns
- risks and testing needs

Use only the analyses relevant to this task.

Require human approval for significant architectural changes per the Human Decision Gates in AGENTS.md. Do not treat routine component choices as significant architectural changes.

## Phase 8 — Plan

Before the plan is finalized, complete the dependency assessment described
in `workflows/dependency-assessment.md` and record its outcome.

Provide a concise implementation plan including:

- files and modules affected
- existing abstractions to reuse
- the dependency outcome (`NO_NEW_DEPENDENCY`, `REUSE_EXISTING`,
  `NEW_DEPENDENCY_PROPOSED`, or `BLOCKED`) with its evidence
- proposed new components only when justified
- formatting and linting setup when the project lacks it
- the Secure Delivery Workflow selected when the task involves CI/CD,
  security, or delivery controls
- validation
- risks

A new dependency must be authorized through the approval gate in
`workflows/dependency-assessment.md` before it is added or installed. Do not
treat approval to implement the task as authorization to introduce
libraries.

## Phase 9 — Implement

Implement on the requested branch.

Preserve existing conventions.

Proceed autonomously for routine work after alignment. Do not pause for user approval for each file edit, test, formatting change, local refactoring, or ordinary corrective adjustment within the approved scope. Do not weaken quality, security, testing, or validation standards.

Provide concise status updates at meaningful milestones (alignment complete, milestone completed, important new finding or decision required, validation results, final diff/summary). Each update must explain what changed, the evidence available, and the next step. An update is not a request for approval unless a decision gate applies.

If new evidence changes a material requirement or approved design, pause and explain the reason before continuing beyond the approved scope. Otherwise, proceed autonomously.

Use checkpoints according to `workflows/session-resilience.md`. Do not require restarting the terminal.

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

When the change set touches CI/CD, security, or delivery configuration,
follow `workflows/secure-delivery.md` for scoped delivery checks and
respect its read-only and authorization boundaries.

Formatting and linting are mandatory for the application code of the
software project, for every detected language in scope. If the project
lacks a configured formatter or linter, configure them as part of the
task, as described in the Formatting and Linting Policy in AGENTS.md and
in `workflows/project-validation.md`. Report missing tooling in scope as
`SETUP_REQUIRED`, never as `NOT_APPLICABLE`. A documentation-only
knowledge repository is out of scope and must not be converted into a
software project to comply.

Prefer focused tests for changed behavior before running broad suites.

Before an expensive validation, and at logical phase boundaries, record a
checkpoint as described in `workflows/session-resilience.md` so the task can
continue after `/compact` or `/new`. Keep compaction enabled and limit
context growth with focused reads and bounded output.

Use existing test frameworks, type checkers, linters, formatters, build
commands, and configured security tools.

For monorepos, target the affected project or packages when reliable
workspace tooling is available.

Never install dependencies, add frameworks, change configuration, or
execute potentially destructive validation commands without appropriate
authorization, except for the mandatory formatting and linting setup.

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
