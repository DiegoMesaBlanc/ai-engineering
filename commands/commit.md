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
- An instruction to commit in `$ARGUMENTS`, in the prompt that triggered the
  command, or in an earlier turn is not commit approval either. The proposal
  and the approval question come first, every time.
- After the gates pass, present the proposed commit groups, messages, and the
  change set (files, line ranges, diff) so the adjustments can be reviewed,
  then ask for explicit approval.
- Run `git commit` only after the user replies affirmatively to that question
  in the current conversation.
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
- Never create a commit before the user reviewed the proposal and answered
  the approval question in the current conversation, including when
  `$ARGUMENTS` or the original prompt asks for the commit.
- The workflow is the authoritative source for gates, output format, and commit rules; do not restate them here.
- Return the report defined by the workflow.