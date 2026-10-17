---
description: Validate the current project's changes using existing tools, targeted tests, and risk-based quality checks without modifying application behavior, except for the mandatory automated formatting of files within the declared task scope.
agent: orchestrator
---

# /validate

Run the Project Validation Workflow:

`workflows/project-validation.md`

## Input

Optional arguments:

- task URL or task identifier
- source file paths
- test file paths
- module or package name
- validation scope

Use `$ARGUMENTS` as the supplied scope when present.

If no argument is supplied, inspect current Git changes and validate the
relevant current task or change set.

## Mandatory Behavior

1. Resolve the project root.
2. Inspect Git status and current branch.
3. Identify changed files and existing project tooling.
4. Detect language, framework, and repository topology.
5. Verify that a formatter and linter are configured for the application
   code of the software project, for every detected language, and
   configure the mandatory ones when missing. Select tools per AGENTS.md,
   keep the detected configuration and version format, and prevent
   formatter/linter conflicts. When tooling is missing, produce an explicit
   proposal (selected tool, reason, configuration, scripts, local
   dependencies) instead of a silent omission. When the target repository
   has no application source code (a documentation-only knowledge
   repository), report that determination as out of scope and do not
   create a package.json or install tooling solely to comply.
6. Select relevant checks according to risk.
7. Execute safe, available validation commands.
8. Report actual outcomes and limitations.

Formatting and linting are mandatory within the scope defined in
`AGENTS.md`: the application code of the software project, per its
detected stack. Tests, type checking, and builds remain risk-based. Treat
missing formatter or linter tooling in scope as `SETUP_REQUIRED`, never as
`NOT_APPLICABLE`.

For each formatter and linter in scope, distinguish its lifecycle states —
detected, configured, installed, executed, approved — from the check-result
status. The standard mandatory setup is authorized by the AGENTS.md policy
and needs no redundant approval; a paid tool, a non-standard dependency, or
replacing a working configuration does require a decision. Report a real
block as `BLOCKED`, never as success.

Do not assume npm, pytest, Maven, Gradle, Vitest, or another tool is
available without repository evidence.

Do not install dependencies, modify configuration, fix code, or change
remote resources, except for the mandatory formatting and linting setup
defined in `AGENTS.md` and `workflows/project-validation.md`.

Before running a command with significant or uncertain side effects,
inspect it and request authorization if necessary.

Use focused tests when appropriate. Broaden validation when required by
risk, project policy, or failure evidence.

## Output

Return:

- checks executed
- status of each check
- per-tool lifecycle states (detected, configured, installed, executed,
  approved), reported separately from the check statuses
- tooling proposals, blocks, and formatter or linter errors
- actual commands
- failures and evidence
- baseline limitations
- checks that were skipped, blocked, or not run
- recommended next steps

Do not modify application source code, except for the mandatory automated
formatting of files within the declared task scope.

Do not create commits or PRs.
