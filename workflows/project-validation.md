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
- repository class: software project with application source code, or a
  documentation-only knowledge repository

If paths are provided, prioritize them.

Do not assume that every changed file belongs to the requested task.

Classify the repository before applying the mandatory formatter and
linter requirement. The mandatory setup defined in AGENTS.md applies to
the application code of software projects, per their detected stack. A
documentation-only knowledge repository is out of scope: do not create a
package.json, install formatter or linter tooling, or add scripts in it
solely to satisfy the policy. That determination is a scope
classification, not a `NOT_APPLICABLE` label for missing tooling.

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
`SETUP_REQUIRED`, never as `NOT_APPLICABLE`, when the repository is a
software project within the AGENTS.md scope. For a documentation-only
knowledge repository, apply the out-of-scope determination from Phase 1
instead of configuring mandatory tooling.

For each in-scope tool, also determine its lifecycle state. These states are
per-tool and are independent of the check-result statuses (`PASS`, `FAIL`,
`SETUP_REQUIRED`, `BLOCKED`) reported in Phase 9:

- detected: presence established from repository evidence — a configuration,
  script, manifest entry, or resolved dependency.
- configured: configuration files, ignore patterns, and reproducible scripts
  exist for the tool.
- installed: the tool's dependency is actually resolvable locally (resolved
  lockfile and available package), not merely declared.
- executed: the non-mutating check actually ran and produced an exit code.
- approved: authorization exists for the tool. The mandatory formatter and
  linter required by AGENTS.md are authorized by that policy itself; only a
  deviation — a paid tool, a non-standard dependency, or replacing a working
  configuration — requires an explicit decision.

Record the state actually established, and never promote a later state from
an earlier one: a detected tool is not configured, a declared dependency is
not installed, and an installed tool is not executed until its check has run.

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
2. the structural compliance check in `workflows/output-compliance.md`
3. focused tests for changed behavior
4. relevant static checks
5. tests for affected boundaries
6. integration tests when warranted
7. full test suites or builds when justified
8. configured security and coverage checks when relevant

Use existing tooling and project conventions.

Formatting and linting are mandatory for the application code of every
relevant software project, language, and framework in scope. The structural
compliance check in `workflows/output-compliance.md` is also mandatory for
software projects with source code: it verifies that generated code does not
declare functions where an arrow function applies, does not mix type
declarations with runtime behavior in the same file, and keeps contract files
grouped by consumer. Tests, type checking, and builds remain risk-based.

Do not add dependencies, change configuration, or install frameworks
merely to satisfy a generic checklist. The mandatory formatter and linter
defined in AGENTS.md are the explicit exception and are configured in
Phase 5.

When the change set includes CI/CD, security, or delivery definitions,
consult `workflows/secure-delivery.md` for delivery checks. `/delivery-check`
owns that scope; `/validate` validates the current change set and must not
silently install CI tooling or security scanners.

`/code-audit` is distinct from `/validate`: `/validate` executes tooling
(formatter, linter, tests, build) for the current change set; `/code-audit`
performs a senior/staff whole-cycle audit of generated code for architecture,
patterns, quality, security, data, tests, performance, a11y, i18n, SEO, UI,
and mobile — without running tools and without modifying code.

## Phase 5 — Configure Mandatory Formatting and Linting

When Phase 2 found that a required formatter or linter is missing,
incomplete, or not yet installed, configure it. This is mandatory, not
optional, and is established by the policy in AGENTS.md; do not request a
redundant approval for the standard setup that policy already authorizes.

Before configuring anything, confirm the repository is a software project
within scope. Never create a package.json, install formatter or linter
tooling, or add scripts in a documentation-only knowledge repository
solely to satisfy this policy.

1. Detect before configuring. Reuse the existing tools, configuration,
   scripts, and conventions, and fill only actual gaps. Do not overwrite a
   working configuration or add a redundant second tool.
2. Select tools for the detected stack using the default choices in
   AGENTS.md. Prefer a compatible already-installed dependency before adding
   a new one.
3. Configure concretely for the detected framework and versions, and prevent
   formatter/linter conflicts:
   - JavaScript and TypeScript: Prettier as the single formatter; keep the
     ESLint configuration in the format the detected ESLint major expects
     (flat config for the current major, legacy eslintrc only while the
     project still uses it); disable the rules that overlap with Prettier
     instead of letting the two disagree.
   - **JavaScript and TypeScript: install the structural compliance rules**
     from `references/linter-rules.md` when configuring or repairing the
     ESLint configuration. The project must end up with `func-style`,
     the `no-restricted-syntax` selector for function expressions, and the
     local `structural/no-mixed-types-and-runtime` rule, plus the contract
     consumer script. A linter that runs without these rules is incomplete
     even when it reports no error.
   - **Other ecosystems: install the type-per-file rule when the ecosystem
     linter can express it**, per the coverage matrix in
     `references/linter-rules.md`: Checkstyle `OneTopLevelClass` for Java,
     StyleCop `SA1402` for C# and .NET, and PHP_CodeSniffer
     `PSR1.Classes.ClassDeclaration` for PHP. When the ecosystem has no
     equivalent rule, do not invent a plugin: the rule stays covered by the
     structural compliance check and the report states that explicitly.
   - Angular: Angular ESLint together with Prettier; do not add a generic
     ESLint configuration that conflicts with the Angular one.
   - Ionic: the tooling of its underlying Angular or React stack.
   - Python: a single Ruff configuration (Ruff format and Ruff check) in the
     project's existing configuration file; do not add Black or Flake8 when
     Ruff already covers the project.
   - Java: the project-standard formatter and linter already referenced by
     the build.
   - SQL: the dialect-aware formatter and linter when the dialect is known.
   - other ecosystems: the ecosystem-standard formatter and linter; do not
     invent compatibility.
   Add configuration files and ignore patterns for generated artifacts and
   build output, without excluding relevant source code.
4. Scope the change in monorepos. Identify the workspace tool and the
   packages the change affects, and limit configuration, scripts, and
   dependencies to those packages (or to a root configuration only when it
   genuinely applies to all). Do not reconfigure or reformat unrelated
   packages.
5. Add reproducible scripts appropriate to the ecosystem, using the existing
   package manager or build system.
6. Add the development dependencies with the existing package manager and
   compatible versions, project-locally; never install globally. Update the
   lockfile when installation succeeds.
7. Format files within the task scope only when necessary and safe, without
   rewriting unrelated user-owned files.
8. Run the formatter's non-mutating check and the linter, capturing exit
   codes and output.
9. Disclose every configuration, script, and dependency change, together
   with the resulting tool lifecycle states.

When tooling is missing, produce an explicit proposal instead of a silent
omission: name the selected tool for the detected stack, the reason, the
configuration and scripts to add, and the local dependencies required. If
setup is authorized and the environment allows it, apply it; if a real
restriction blocks it, report `BLOCKED` and state what remains incomplete.

The mandatory formatting and linting requirement authorizes this
project-local setup. It does not authorize paid tools, paid services, or
unrelated dependency upgrades, which still require an explicit decision.

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

**Verify that the linter is actually enforcing the structural rules.** A
successful lint run proves nothing when the configuration is empty or lacks
the structural rules. Confirm, per the coverage matrix in
`references/linter-rules.md`:

- JavaScript and TypeScript: the resolved ESLint configuration contains
  `func-style`, the `no-restricted-syntax` selector for function
  expressions, and the local `structural/no-mixed-types-and-runtime` rule,
  and the contract consumer script exists and is wired into the project
  scripts.
- Java: the Checkstyle configuration enables `OneTopLevelClass`.
- C# and .NET: the analyzer configuration sets `SA1402` to error.
- PHP: the PHP_CodeSniffer ruleset references `PSR1.Classes.ClassDeclaration`.
- Python, Go, Rust, Kotlin, and Ruby: the ecosystem linter cannot express
  the rule. Confirm that the structural compliance check covers it and
  report it as covered by verification rather than as an installed rule.

Report a missing rule as `SETUP_REQUIRED` even when the lint command exits
successfully.

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

Formatting and linting are always required within the defined scope when
a formatter and linter apply to the detected languages of a software
project. Include other checks only when relevant.

Use the configured statuses:

- PASS, FAIL, BASELINE_FAILURE, BLOCKED, NOT_RUN, SKIPPED
- SETUP_REQUIRED: mandatory tooling was missing and the workflow is
  configuring it; this is not a final successful result.

Report the per-tool lifecycle states (detected, configured, installed,
executed, approved) separately from these check-result statuses, and list
any tooling proposal, block, or formatter/linter error. For each formatter
and linter in scope, state the tools that were detected, configured,
installed, executed, and approved, so a configured-but-not-installed or
installed-but-not-executed tool is never presented as a verified check.

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
