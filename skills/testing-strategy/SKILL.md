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
   and authorization, except for the mandatory formatting and linting
   setup defined in AGENTS.md and Section 6.
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

### Go

Inspect `go.mod`, the toolchain version, and existing scripts. Use
`gofmt -l`, `go vet ./...`, `go test ./...`, and `go build ./...`; run
`golangci-lint` or `govulncheck` only when the project already configures
them.

### Rust

Inspect `Cargo.toml`, the toolchain file, and workspace members. Use
`cargo fmt --check`, `cargo clippy`, `cargo test`, and `cargo build`; run
`cargo audit` only when the project already configures it.

### C# / .NET

Inspect the solution, project files, and target frameworks. Use
`dotnet format --verify-no-changes`, the configured analyzers (StyleCop),
`dotnet test`, and `dotnet build`; run
`dotnet list package --vulnerable` when a security check is relevant.

### PHP

Inspect `composer.json`, the PHP version, and the test configuration. Use
the project's PHP-CS-Fixer (dry run), PHPStan or Psalm, PHPUnit, and
`composer audit`; a PHP library has no build step, so validation is static
plus tests.

### Ruby

Inspect `Gemfile`, the Ruby version, and the test configuration. Use
RuboCop, RSpec or Minitest, and `bundler-audit` when configured. A gem
build is a packaging step, not a correctness check.

### Svelte, Solid, and Vue

Use the detected Vite/Vitest setup, the framework compiler check
(`svelte-check`, `vue-tsc`, or the Solid TypeScript configuration), and the
project's formatter and linter. A component test lives next to the
component.

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

## 6. Static Analysis, Formatting, and Linting

Formatting and linting are mandatory within the scope defined in
AGENTS.md: the application code of the software project, per its detected
stack. Every in-scope project must have a configured formatter and linter
appropriate for its languages and frameworks, with reproducible commands
to run both. Documentation-only knowledge repositories are out of scope;
never create a package.json or install tooling in them solely to satisfy
this policy. Follow the canonical tooling policy in AGENTS.md:

- JavaScript and TypeScript: Prettier and ESLint with framework plugins.
- Angular: Prettier and Angular ESLint, including template linting.
- Ionic: the tooling of its underlying Angular or React stack.
- Svelte and Solid: Prettier, the framework compiler check, and ESLint.
- Python: Ruff format and Ruff check, plus the configured type checker.
- Java: google-java-format or Spotless, plus Checkstyle or the project's linter.
- Kotlin: ktlint or detekt, plus the Gradle build.
- Go: `gofmt` and `go vet`, plus `golangci-lint` when configured.
- Rust: `cargo fmt --check` and `cargo clippy`.
- C#/.NET: `dotnet format` and the configured analyzers such as StyleCop.
- PHP: PHP-CS-Fixer and PHPStan or Psalm.
- Ruby: RuboCop.
- SQL: dialect-aware formatting and linting such as SQLFluff.
- Other languages: the ecosystem-standard formatter and linter.

Detect existing configuration, dependencies, scripts, and source-file
coverage before selecting tools. Preserve suitable existing tooling and do
not force Prettier onto file types it does not support.

If the mandatory formatter or linter is missing or incomplete for an
in-scope software project, report it as `SETUP_REQUIRED`, not as
`NOT_APPLICABLE`, and configure it as part of the task:

1. select compatible open-source tools
2. add configuration files and ignore patterns for generated output
3. add reproducible, project-appropriate scripts
4. add development dependencies with the existing package manager
5. update the lockfile when installation succeeds
6. format files within the declared task scope when necessary and safe,
   without rewriting unrelated user-owned files
7. run the formatter's non-mutating check and the linter
8. disclose every configuration and dependency change made

Never install paid tools or introduce paid services, and do not make
unrelated dependency upgrades.

For JavaScript and TypeScript, expose scripts equivalent to `lint`,
`format`, and `format:check`, adapted to the existing package manager.
Adopt equivalent reproducible commands for other ecosystems through their
build system or scripts.

Run configured linting, type checking, and a non-mutating formatting check.
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

A successful build does not substitute for successful formatting or
linting.

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

The mandatory formatting and linting setup defined in Section 6 is an
explicit exception. Adding the required open-source development
dependencies with the existing package manager is expected and does not
require separate authorization. Do not make unrelated upgrades.

Never use a different test runner merely to avoid reporting a missing
dependency.

When a real environment limitation, unavailable network, permission
restriction, or incompatible dependency prevents mandatory setup, report
`BLOCKED`, explain the exact reason, and state what remains incomplete.

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
- SETUP_REQUIRED: mandatory tooling is missing and must be configured; an
  intermediate state that is never a final successful result
- NOT_RUN: available but not executed
- SKIPPED: deliberately omitted with a reason
- NOT_APPLICABLE: irrelevant to the current change

`NOT_APPLICABLE` must never be used for mandatory formatting or linting
within the defined scope merely because configuration, dependencies, or
scripts are absent. A documentation-only repository is an out-of-scope
classification, not a missing-tool classification.

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

Choose the validation status defined in `workflows/project-validation.md`
(`VALIDATION PASSED`, `VALIDATION PASSED WITH LIMITATIONS`,
`VALIDATION FAILED`, `VALIDATION BLOCKED`).

Explain the smallest necessary next step.

## 14. Completion Rule

A task is not automatically complete because the code compiles.

Completion requires appropriate validation, review of the changed behavior,
and honest reporting of remaining risks and unexecuted checks.
