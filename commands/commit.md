---
description: Prepare and validate a commit proposal from the current change set using project-native quality gates (format, lint, typecheck, tests, build, security, AI sanity review), then create approved atomic commits with English messages only after explicit user approval. Never pushes or deploys.
agent: orchestrator
permission:
  edit: allow
  bash: allow
---

# /commit

Run the Commit Workflow:

`workflows/commit.md`

## Approval gate

This command **prepares and validates a proposal**. It does not create
commits by itself.

- Invoking `/commit` is not commit approval.
- After the gates pass, present the proposed commit groups, messages, and
  files, then ask for explicit approval.
- Run `git commit` only after the user replies affirmatively in the current
  conversation.
- If the user declines or requests changes, do not commit; adjust the
  proposal and ask again.

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
- Never create a commit before explicit approval in the current conversation.
- The workflow is the authoritative source for gates, output format, and commit rules; do not restate them here.
- Return the report defined by the workflow.