---
description: Validate the current project's changes using existing tools, targeted tests, and risk-based quality checks without modifying source code.
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
5. Select relevant checks according to risk.
6. Execute safe, available validation commands.
7. Report actual outcomes and limitations.

Do not assume npm, pytest, Maven, Gradle, Vitest, or another tool is
available without repository evidence.

Do not install dependencies, modify configuration, fix code, or change
remote resources.

Before running a command with significant or uncertain side effects,
inspect it and request authorization if necessary.

Use focused tests when appropriate. Broaden validation when required by
risk, project policy, or failure evidence.

## Output

Return:

- checks executed
- status of each check
- actual commands
- failures and evidence
- baseline limitations
- checks that were skipped, blocked, or not run
- recommended next steps

Do not modify source code.

Do not create commits or PRs.
