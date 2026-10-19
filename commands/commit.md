---
description: Generate and validate commits from the current change set using project-native quality gates (format, lint, typecheck, tests, build, security, AI sanity review), creating atomic commits with English messages. Never pushes or deploys.
agent: orchestrator
permission:
  edit: allow
  bash: allow
---

# /commit

Run the Commit Workflow:

`workflows/commit.md`

## Input

Optional arguments in `$ARGUMENTS`:

- File or directory paths to commit (relative to project root).
- `--staged` to limit scope to currently staged changes only.
- `--dry-run` to execute all gates and show proposed commits without creating them.
- `--full` to include integration and E2E tests in the gate sequence.

If omitted, scope is all uncommitted changes:
- Staged + unstaged tracked changes (`git diff HEAD`)
- Untracked files (`git ls-files --others --exclude-standard`)
- If the working tree is clean: `NOTHING TO COMMIT`.

## Adapter constraints

- Never push, deploy, or bypass gates.
- The workflow is the authoritative source for gates, output format, and commit rules; do not restate them here.
- Return the report defined by the workflow.