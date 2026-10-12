# Project Validation Workflow

## Purpose

Validate the current project and relevant changes using existing
tooling, risk-based scope, and reproducible evidence.

Do not modify application source code as part of this workflow.

Validation tools may create generated output. Inspect commands and
potential side effects before execution.

## Phase 1 — Identify Scope

Determine:

- project root
- current branch
- current working-tree status
- changed files
- supplied task or paths
- relevant modules and dependencies

If paths are provided, prioritize them.

Do not assume that every changed file belongs to the requested task.

## Phase 2 — Establish Available Tooling

Inspect relevant project manifests and configuration.

Examples:

- package.json and lockfiles
- TypeScript configuration
- test runner configuration
- Angular CLI configuration
- Python project and test configuration
- Maven or Gradle build files
- workspace and monorepo configuration
- CI pipeline definitions

Identify commands that actually exist.

Do not invent or assume package scripts.

## Phase 3 — Determine Risk

Use `skills/testing-strategy/SKILL.md`.

Determine:

- task scope
- complexity
- correctness risk
- architectural impact
- security impact
- regression risk
- required validation depth

## Phase 4 — Select Validation

Prefer:

1. focused tests for changed behavior
2. relevant static checks
3. tests for affected boundaries
4. integration tests when warranted
5. full test suites or builds when justified
6. configured security and coverage checks when relevant

Use existing tooling and project conventions.

Do not add dependencies, change configuration, or install frameworks
to satisfy a generic checklist.

## Phase 5 — Inspect Command Safety

Before executing commands, determine whether they may:

- deploy or publish
- modify a database
- push or merge
- delete files
- rewrite generated code
- modify remote resources
- incur external costs

Do not execute risky commands without authorization.

Never use production databases or environments for destructive tests.

If a validation requires network access or dependency installation,
report the requirement and obtain authorization when necessary.

## Phase 6 — Execute

Run the selected commands.

Record:

- command
- purpose
- relevant scope
- result
- exit code when available
- useful failure evidence

Do not claim success without execution evidence.

## Phase 7 — Analyze Failures

Classify failures as:

- introduced by the current change
- pre-existing with supporting evidence
- configuration failure
- missing dependency
- environment issue
- permission issue
- inconclusive

Do not edit unrelated code to make unrelated checks pass.

## Phase 8 — Report

Produce:

### PROJECT VALIDATION

Project:
Branch:
Scope:
Risk:

| Check | Status | Evidence |
|---|---|---|
| Tests | ... | ... |
| Type checking | ... | ... |
| Lint | ... | ... |
| Formatting | ... | ... |
| Build | ... | ... |
| Security | ... | ... |
| Coverage | ... | ... |

Include only relevant checks.

### Findings

...

### Skipped or Blocked Checks

...

### Recommendation

- VALIDATION PASSED
- VALIDATION PASSED WITH LIMITATIONS
- VALIDATION FAILED
- VALIDATION BLOCKED

Explain the reason and the minimum next step.

## Safety

This workflow validates; it does not implement fixes.

Never:

- modify application source code
- install dependencies without authorization
- fix unrelated files
- discard user changes
- create commits
- push branches
- create or publish PRs
- approve, merge, or publish review comments
