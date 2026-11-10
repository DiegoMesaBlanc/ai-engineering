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
- Go modules, Cargo workspace, .NET solution, Composer, and Gemfile
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

When the project has `scripts/check-docs`, run it as a documentation gate.
It is part of the validation of a project that adopted the `docs-gate`
module of `workflows/engineering-baseline.md`: a non-zero exit is a finding,
and a missing script is not a failure for a project that never adopted that
module.

Use existing tooling and project conventions.

Formatting and linting are mandatory for the application code of every
relevant software project, language, and framework in scope. The structural
compliance check in `workflows/output-compliance.md` is also mandatory for
software projects with source code: it verifies that generated code does not
declare functions where an arrow function applies, does not mix type
declarations with runtime behavior in the same file, and keeps contract files
grouped by consumer. A `FAIL` of that check blocks every gate that runs it (commit,
validation, and delivery check) and is not reported as a warning; a violation that
pre-existed the change set is a `FINDING`. Tests, type checking, and builds remain
risk-based.

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
incomplete, or not yet installed, propose it. The policy requires a
**configured linter** for every language and framework in scope; a project
without one is not validated.

The setup is mandatory, and it is an installation, so it is an
**installation package** like any other: the formatter, the linter, and
their plugins are assembled as one set. A setup with a single package is
installed directly; a setup with several is presented once with the
installation list and installed after the user's evaluation, through the gate
of `workflows/dependency-assessment.md`. What makes the setup standard is the
recommendation, not the absence of evaluation.

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
   - **JavaScript and TypeScript: keep the configuration standard.** Use the
     detected ESLint flat or legacy configuration and enable the core rules
     `func-style`, the `no-restricted-syntax` selector for function
     expressions, and `@typescript-eslint/consistent-type-imports` when
     `typescript-eslint` is already part of the setup. Do not create a custom
     rule file, a local plugin, or a checker script, and do not add a
     dependency for the structural policy. The placement rules are verified by
     `workflows/output-compliance.md`, not by project tooling.
   - **Existing code with `function` declarations:** do not convert it to
     satisfy the arrow functions rule, and do not let it turn the build red.
     Enable the rule for code written from now on and apply one of the patterns in
     `references/linter-rules.md` ("Existing projects with legacy
     declarations"): freeze the named legacy files with a documented override,
     or lint the change set. The legacy declarations are recorded as findings,
     and the conversion is a task of its own.
   - **Other ecosystems:** use the ecosystem-standard formatter and linter.
     Where that linter ships a documented rule for separating types from
     runtime (Checkstyle `OneTopLevelClass`, StyleCop `SA1402`,
     PHP_CodeSniffer `PSR1.Classes.ClassDeclaration`), enabling it is optional
     configuration; the system's compliance check remains the authority.
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
   rewriting unrelated user-owned files. Every file in a change set is named in
   its approved scope manifest (M-71), and a formatter or linter run that
   rewrites a file the approved scope does not name is reverted for that file
   before the change set is proposed (`SCOPE-001` in
   `workflows/output-compliance.md`). Run the formatter in its non-mutating check
   mode first, so the write is limited to the files the task owns.
8. Run the formatter's non-mutating check and the linter, capturing exit
   codes and output. A violation that pre-existed the change set is a
   finding, not a gate failure; only the violations the change set introduces
   block the task. When a whole-project script is the only formatter or linter
   entry point, intersect its failing files with the change set: a failing file
   inside the change set fails the gate; failing files outside it are
   `BASELINE_FAILURE` with their count and first location, and the gate on the
   change set is reported on its own result.
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

**Verify that the linter is configured and actually running.** A lint command
that exits successfully proves nothing when the configuration is empty or
missing. Confirm:

- JavaScript and TypeScript: the resolved ESLint configuration contains
  `func-style` and the `no-restricted-syntax` selector, and a Prettier
  configuration exists. There is no custom rule file, plugin, or checker
  script in the project.
- Other ecosystems: the ecosystem-standard linter configuration exists and
  runs.
- The structural placement rules are not verified here. They are verified by
  `workflows/output-compliance.md` over the change set.

Report a missing formatter or linter as `SETUP_REQUIRED` even when the build or
the lint command exits successfully.

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

A typecheck whose every error is located under `node_modules` is a
configuration failure, not a dependency defect. The project's TypeScript
configuration resolves dependency source files, and `skipLibCheck` does not
cover `.ts` sources. The fix belongs in the project's configuration; a
dependency is never edited. The failure stays a failing gate with that
classification.

## Phase 8a — Project Health Probe

This is a read-only health probe. It runs when the system enters a software
project for the first time in a session, and when `/validate` covers a whole
project. It reports every pre-existing failure with its evidence, and it never
blocks a task unrelated to the finding (M-53).

Run each check, and record its command, status, and evidence:

1. **Gate scripts.** Read the project's manifest for formatter, linter,
   typecheck, and test scripts. A missing formatter or linter is a finding with
   status `SETUP_REQUIRED` (Phase 5), never `NOT_APPLICABLE`.
2. **Tracked generated artifacts.** A file that a build, typecheck, or cache
   regenerates must not be versioned:

   ```sh
   git ls-files | grep -E '(\.tsbuildinfo$|(^|/)(dist|build|\.next|coverage|node_modules)/)'
   ```

   Each match is a finding with its path. Removing it from the index is a write;
   propose it, do not perform it.
3. **Dependency audit.** Enumerate every lockfile and every workspace root of
   the project (`find . \( -name package-lock.json -o -name pnpm-lock.yaml -o
   -name yarn.lock \) -not -path '*/node_modules/*'`, or the ecosystem equivalent),
   and run the audit in each root separately. Report each root with its path;
   findings from one root are never attributed to another, and a total is the
   sum of the reported roots, never a re-attribution. When the network is
   authorized and available, run the configured audit (for example
   `npm audit --audit-level=high`) in each root. Report every critical and high
   finding with its fixed version when one exists. A lockfile project with no
   configured audit is a finding. When the vulnerability database is unavailable,
   the check is `NOT_RUN`, never clean. A suggested fix that moves a package to an
   older major version is not a remediation: a dependency remediation never
   downgrades (M-57). A remediation diff is checked by `DEP-001` in
   `workflows/output-compliance.md`, which fails a manifest change that moves a
   dependency to an older major. The fix is the fixed version of the package, the direct
   dependency that brings it in, or a documented override of that one package, and
   a blocker without a fixed version is reported as such.
4. **Gates.** Run the formatter's non-mutating check, the linter, the typecheck,
   and the tests. A pre-existing failure is `BASELINE_FAILURE` with its count and
   first location. A typecheck with errors only under `node_modules` is a
   configuration failure (Phase 8). A test that needs an environment variable or
   database the environment lacks is `NOT_RUN`, and the missing name is recorded.
   A typecheck or build that writes a cache is recorded as a side effect of the
   probe, and the cache is disabled when the tool allows it.
5. **Duplicated source trees.** When sibling projects contain source trees with
   the same name and different content, run `diff -rq` on them and record each
   as a `DUPLICATE-TREE` finding, following `skills/repository-strategy/SKILL.md`.
6. **Whole-project safety sweep (M-49, M-52, M-54 to M-63).** Run
   `project_safety_sweep` from `workflows/output-compliance.md` from the project
   root. It scans every tracked and untracked file, not only the change set, and
   prints what it scanned and what it excluded, so the probe never skips a file
   by hand. It reports, each with `file:line`: dead exports (`DEAD-EXPORT-001`),
   no hardcoded environment endpoint and no fixed shared temporary path (M-58,
   `SAFE-003`), outbound calls with no timeout (`SAFE-001`), cleartext endpoints
   and schemes that accept
   `http://` (`SAFE-002`), a URL path that interpolates an identifier whose value
   is not encoded (`SAFE-004`; the rule is that a value is encoded with the encoder
   of its context), cache keys and sign-out paths without identity partitions and
   payment data in plain storage (`SAFE-005`), a `401` that does not end the session
   (`SAFE-006`), a build manifest that enables backup (`SAFE-007`; an app that holds
   personal or payment data does not allow backup of personal data), and an identity
   read from a client-controlled header, query, or connection parameter (`AUTH-001`),
   a wildcard origin combined with credentials and a development tool enabled by a
   literal `true` (`SAFE-008`, M-62: a wildcard origin is never combined with
   credentials), and an upload written with the default file mode (`SAFE-009`, M-63:
   a file that holds personal or sensitive data is written with an owner-only file
   mode). Each is a finding
   that never blocks an unrelated task, and the probe never reports one as passing.
   A missing manifest is `NOT VERIFIED` for M-61, never clean.
   The same sweep also reports schema DDL outside a migration (`DB-001`, M-70),
   a migration with no deploy-order test (`DB-002`, M-69), and every universal,
   artifact, compatibility, performance, or technical-property claim in the
   documents and the source comments as `NOT VERIFIED CLAIM-001` (M-65, M-68, M-69).
   The same sweep reports every non-composition module that imports a concrete module
   (`DIP-001`, M-96: a concrete module is imported only by the composition root) and every
   cited artifact that does not resolve inside the repository (`ARTIFACT-001`, M-97: a
   cited artifact resolves inside the repository).
   A pre-existing DDL statement in a store file is a finding of the probe, never a
   pass, and a performance or plan claim is never reported as verified by the probe.
7. **Outbound timeouts and cleartext endpoints (M-54, M-55).** The sweep of item 6
   covers `SAFE-001` and `SAFE-002` over the project source. Every external call
   has an explicit timeout (M-54), so an outbound call with no timeout is a
   `SAFE-001` finding, and `http://` is rejected for every non-local build (M-55),
   so a cleartext endpoint or a URL setting that accepts any scheme is a `SAFE-002`
   finding. Each is reported with its `file:line`, without blocking an unrelated
   task, and the probe never reports them as passing. A rejected credential ends
   the session (M-60): the sweep's `SAFE-006` finding covers it, and the device
   behavior is `NOT VERIFIED` until it is run.

The report lists every check with its status and evidence. A failing check is
never reported as passing, and a finding in unrelated code is reported with its
file and line without blocking the task.

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

A failing formatter, linter, typecheck, or test on the change set is never
reported as a passing validation. The task is not reported as complete while
those gates fail (M-37): the status is `VALIDATION FAILED` or `VALIDATION
BLOCKED`, and the failing command and its output are reported.

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
