---
name: testing-strategy
description: Select and execute project-appropriate testing and quality checks based on detected technologies, existing tooling, task scope, risk, and validation evidence.
---

# Testing and Validation Strategy

## Purpose

Determine the smallest sufficient validation strategy for a software
change while providing reliable evidence of correctness.

Validation must be adapted to the actual repository.

Do not assume a universal test framework, command, coverage threshold,
or quality gate.

## Core Principles

1. Detect the stack and available tools before selecting checks.
2. Prefer existing project configuration and scripts.
3. Start with targeted validation when appropriate.
4. Increase validation scope according to risk.
5. Do not install dependencies or introduce tools without justification
   and authorization.
6. Preserve the user's code and working tree.
7. Report actual execution results.
8. Never claim that a check passed when it was not executed.
9. Distinguish existing failures from failures introduced by the change
   when reliable baseline evidence is available.
10. A successful build alone does not establish that the task is complete.

## 1. Inspect the Project

Determine:

- language and runtime
- framework and version
- package manager or build system
- test framework and version
- unit test commands
- integration test commands
- E2E test commands
- type-checking commands
- linting and formatting configuration
- build commands
- coverage tooling
- security checks already configured
- CI/CD quality gates
- affected modules and applications

Use repository evidence, not assumptions based on framework popularity.

## 2. Determine Validation Risk

Classify the change:

### Trivial

Examples:

- documentation-only changes
- isolated wording changes
- nonfunctional formatting changes

Use relevant lightweight validation.

### Low

Examples:

- isolated UI changes
- a small localized behavior change
- low-risk refactoring with existing test coverage

Use focused tests or equivalent validation.

### Medium

Examples:

- business logic changes
- API contract changes
- state-management changes
- database access changes

Use focused tests and relevant integration or regression checks.

### High

Examples:

- authentication or authorization changes
- payment or financial behavior
- data migrations
- shared libraries
- public API contracts
- changes affecting multiple modules

Use broader regression validation and relevant security or integration
checks.

### Critical

Examples:

- destructive data migrations
- security boundaries
- safety-critical behavior
- production infrastructure changes
- high-impact changes with significant rollback risk

Require an explicit validation and rollback strategy.

Ask for human approval when a necessary validation could modify shared,
production, or otherwise sensitive resources.

Complexity and validation risk are related but not identical.

## 3. Select Tests

Prefer:

1. tests directly covering changed behavior
2. tests covering affected components or services
3. regression tests for relevant callers and dependencies
4. integration tests for changed boundaries
5. broader test suites according to risk and project policy
6. E2E tests when the affected user journey requires them

Do not run every test in a large repository by default if narrower,
reliable validation is sufficient.

Expand validation when:

- targeted tests fail
- shared behavior is affected
- dependencies or public contracts change
- a security-sensitive path changes
- CI policy requires broader testing
- the available evidence is insufficient

## 4. TDD Integration

Load `test-driven-development` when the task changes behavior and
meaningful automated tests can protect that behavior.

TDD is language-neutral.

Use the project's existing test tools and conventions.

For new behavior, prefer:

Red → Green → Refactor

When a regression test is appropriate, verify that it fails for the
expected reason before implementing the fix.

Never delete existing implementation or user work solely to comply
with TDD.

For documentation-only, formatting-only, or nonbehavioral changes,
use proportionate validation instead of meaningless tests.

## 5. Stack-Specific Discovery

Use these as discovery guides, not mandatory command lists.

### JavaScript and TypeScript

Inspect:

- package.json scripts
- package manager and lockfile
- tsconfig.json
- ESLint and formatter configuration
- test runner configuration
- framework build configuration

Use existing scripts for relevant checks.

Possible checks include:

- typecheck
- lint
- format verification
- focused tests
- integration tests
- build

Do not assume every repository defines scripts called `test`, `lint`,
`typecheck`, `check`, or `build`.

Do not run `npm`, `pnpm`, `yarn`, or `bun` arbitrarily when the repository
uses a different package manager.

### Angular

Inspect the Angular version, CLI configuration, workspace projects,
existing test configuration, and available npm scripts.

Run the checks configured for the actual project.

Do not assume that `ng test`, linting, or E2E support is configured
without checking the workspace.

### React, Vue, and Next.js

Use the framework's existing test and build configuration.

For Next.js, detect App Router or Pages Router and validate relevant
components, routes, server behavior, and build configuration as
appropriate.

Do not introduce a dedicated test framework solely for the task.

### Node.js

Inspect package scripts, runtime version, module system, test commands,
and API or service boundaries.

Use focused unit and integration tests for changed behavior.

### Python

Inspect pyproject.toml, requirements files, lockfiles, test configuration,
and existing scripts.

Use the test runner, formatter, linter, and type checker already
configured by the project.

Do not assume pytest, Ruff, mypy, or another tool is installed.

### Java

Inspect the Java version, pom.xml or Gradle files, wrappers, test
configuration, and existing build tasks.

Use the repository's established Maven or Gradle workflow.

Do not assume that every project uses Spring or the same testing framework.

### React Native, Ionic, and Capacitor

Use the existing framework tests and TypeScript checks when available.

Add platform-specific validation only when the change affects native
behavior, plugins, platform configuration, or delivery requirements.

Do not generate release builds, sign applications, or publish packages
unless explicitly requested.

### Databases

Inspect the database, driver, ORM, migration strategy, and test setup.

Validate migrations and queries using the intended test environment
when available.

Never run destructive tests or migrations against production resources
without explicit authorization.

### Monorepos

Identify the affected project or package.

Prefer established affected-project or workspace-filtered commands.

Use full-repository validation when required by CI policy or when the
change's risk justifies it.

Do not assume that a monorepo uses Nx, Turborepo, or a particular workspace
tool.

## 6. Static Analysis and Formatting

Run configured linting, type checking, and formatting checks when relevant.

Do not automatically fix unrelated files merely to achieve a clean
validation result.

Report whether a failure comes from:

- source code
- configuration
- missing dependencies
- environment
- unrelated pre-existing errors
- unavailable permissions

Do not confuse formatting checks with source-code correctness.

## 7. Build Validation

Run the existing build command when relevant to the task.

Before executing a command, inspect what it does if it could:

- deploy to an environment
- publish a package
- modify a database
- change remote resources
- delete files
- regenerate tracked code
- perform another significant side effect

Ask for authorization before risky actions.

A local build is not the same as a production deployment.

## 8. Security Validation

Use existing security checks when configured and relevant.

Examples may include:

- dependency auditing
- SAST
- secret scanning
- security-focused tests
- infrastructure validation

Do not add security tools, contact external services, or initiate
potentially paid scans without justification and authorization.

Never expose credentials or secrets in validation output.

## 9. Coverage

Use existing coverage tooling when the task, repository policy, or CI
requires it.

Do not impose a universal coverage threshold on every project.

Prefer meaningful coverage of changed behavior and important failure
paths over arbitrary percentages.

Report missing coverage tooling when it prevents a required check.

## 10. Baseline and Regression Analysis

Before validation:

- inspect repository status
- understand the current changes
- use existing CI results or baseline test evidence when available

If a failure is known to exist before the task, report it as a baseline
failure only when evidence supports that conclusion.

Do not switch branches or destroy the current workspace merely to
establish a baseline.

Do not claim a regression is introduced by the task without adequate
evidence.

## 11. Dependency and Network Policy

Do not automatically install missing dependencies.

If dependencies are unavailable:

- report which check is blocked
- determine whether the necessary dependency already exists
- explain the minimal installation or configuration required
- obtain approval before a significant or network-dependent change

Never use a different test runner merely to avoid reporting a missing
dependency.

## 12. Execution Evidence

For each check record:

- check name
- command or mechanism
- scope
- execution status
- exit code when available
- concise result
- relevant evidence or failure
- limitations

Use these statuses:

- PASS: executed successfully
- FAIL: executed and failed
- BASELINE_FAILURE: reliable evidence shows a pre-existing failure
- BLOCKED: could not run because a requirement was unavailable
- NOT_RUN: available but not executed
- SKIPPED: deliberately omitted with a reason
- NOT_APPLICABLE: irrelevant to the current change

Do not label checks PASS based on configuration alone.

## 13. Final Report

Return:

### VALIDATION SUMMARY

- Project:
- Task:
- Scope:
- Risk:
- Validation commands:
- Tests:
- Type checking:
- Lint:
- Formatting:
- Build:
- Security:
- Coverage:
- Baseline limitations:

### Findings

List failures, blocked checks, and relevant warnings with evidence.

### Recommendation

Choose:

- VALIDATION PASSED
- VALIDATION PASSED WITH LIMITATIONS
- VALIDATION FAILED
- VALIDATION BLOCKED

Explain the smallest necessary next step.

## 14. Completion Rule

A task is not automatically complete because the code compiles.

Completion requires appropriate validation, review of the changed behavior,
and honest reporting of remaining risks and unexecuted checks.
