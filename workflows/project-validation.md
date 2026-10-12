# Project Validation Workflow

## Purpose

Validate the current project and relevant changes using existing
tooling, risk-based scope, and reproducible evidence.

Do not modify application source code as part of this workflow, except
for automated formatting of files within the declared task scope, as
described in Phase 5. Never rewrite unrelated user-owned files.

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
- formatter configuration such as Prettier, Ruff format, or SQLFluff
- linter configuration such as ESLint, Angular ESLint, Ruff, or Checkstyle
- Angular CLI configuration
- Python project and test configuration
- Maven or Gradle build files
- workspace and monorepo configuration
- CI pipeline definitions

Identify commands that actually exist.

Do not invent or assume package scripts.

For every language and framework in scope, determine whether a formatter
and a linter are configured, together with their dependencies, commands,
and source-file coverage. Record missing mandatory tooling as
`SETUP_REQUIRED`, never as `NOT_APPLICABLE`.

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

1. mandatory formatting verification and linting
2. focused tests for changed behavior
3. relevant static checks
4. tests for affected boundaries
5. integration tests when warranted
6. full test suites or builds when justified
7. configured security and coverage checks when relevant

Use existing tooling and project conventions.

Formatting and linting are mandatory for every relevant language and
framework. Tests, type checking, and builds remain risk-based.

Do not add dependencies, change configuration, or install frameworks
merely to satisfy a generic checklist. The mandatory formatter and linter
defined in AGENTS.md are the explicit exception and are configured in
Phase 5.

## Phase 5 — Configure Mandatory Formatting and Linting

When Phase 2 found that a required formatter or linter is missing or
incomplete, configure it. This is mandatory, not optional.

1. Select compatible open-source tools for the detected stack.
2. Add configuration files and ignore patterns for generated artifacts and
   build output, without excluding relevant source code.
3. Add reproducible scripts appropriate to the ecosystem.
4. Add the development dependencies with the existing package manager and
   compatible versions.
5. Update the lockfile when installation succeeds.
6. Format files within the task scope when necessary and safe, without
   rewriting unrelated user-owned files.
7. Run the formatter's non-mutating check and the linter, capturing exit
   codes and output.
8. Disclose every configuration, script, and dependency change.

The mandatory formatting and linting requirement authorizes this
project-local setup. Do not install paid tools, introduce paid services,
or make unrelated dependency upgrades.

If a real environment limitation, unavailable network, permission
restriction, or incompatible dependency blocks setup, report `BLOCKED`,
explain the exact reason, and state what remains incomplete. Do not treat
`SETUP_REQUIRED` as a final successful result.

## Phase 6 — Inspect Command Safety

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

## Phase 7 — Execute

Before running an expensive or long validation, record a checkpoint as
described in `workflows/session-resilience.md` so the run can be recovered
after `/compact` or `/new`.

Run the selected commands.

Record:

- command
- purpose
- relevant scope
- result
- exit code when available
- useful failure evidence

Do not claim success without execution evidence.

## Phase 8 — Analyze Failures

Classify failures as:

- introduced by the current change
- pre-existing with supporting evidence
- configuration failure
- missing dependency
- environment issue
- permission issue
- inconclusive

Do not edit unrelated code to make unrelated checks pass.

## Phase 9 — Report

Produce:

### PROJECT VALIDATION

Project:
Branch:
Scope:
Risk:

| Check | Status | Evidence |
|---|---|---|
| Formatting | ... | ... |
| Lint | ... | ... |
| Tests | ... | ... |
| Type checking | ... | ... |
| Build | ... | ... |
| Security | ... | ... |
| Coverage | ... | ... |

Formatting and linting are always required when a formatter and linter
apply to the detected languages. Include other checks only when relevant.

Use the configured statuses:

- PASS, FAIL, BASELINE_FAILURE, BLOCKED, NOT_RUN, SKIPPED
- SETUP_REQUIRED: mandatory tooling was missing and the workflow is
  configuring it; this is not a final successful result.

Do not use `NOT_APPLICABLE` for mandatory formatting or linting merely
because a configuration, dependency, or script is absent. Do not convert
missing checks into PASS, and only qualify a passing report with
limitations when every mandatory requirement has been met and every
executed check is represented accurately.

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

This workflow validates and configures mandatory formatting and linting;
it does not implement application fixes.

Never:

- modify application source code, except for automated formatting of
  files within the declared task scope defined in Phase 5; never rewrite
  unrelated user-owned files
- install dependencies without authorization, except for the mandatory
  formatting and linting setup described in Phase 5
- fix unrelated files
- discard user changes
- create commits
- push branches
- create or publish PRs
- approve, merge, or publish review comments
