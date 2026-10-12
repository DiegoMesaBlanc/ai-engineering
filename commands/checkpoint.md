---
description: Write a user-level checkpoint of the current task state outside the target project source tree without modifying source files, commits, or remote state.
agent: orchestrator
---

# /checkpoint

Run the Session Resilience Workflow:

`workflows/session-resilience.md`

Record the current task state so it can be recovered after `/compact` or a
new session (`/new`).

## Input

Optional `$ARGUMENTS`:

- task identifier
- checkpoint path
- milestone or note label

If no argument is supplied, use the current repository, branch, and session
context.

## Mandatory Behavior

1. Resolve the repository root, task identifier, branch, base commit, HEAD,
   and working-tree state.
2. Write to `$HOME/.config/opencode/handoffs/<repo-slug>/<task-slug>.md`,
   outside the target project's source tree.
3. Include objective, completed work, decisions, changed files, validation
   evidence actually observed, blockers, and exact next actions.
4. Record fields that cannot be determined as unknown instead of inventing
   values.
5. Preserve uncommitted work. Do not stage, commit, reset, stash, checkout,
   or edit source files.

## Output

- checkpoint path
- task identifier and branch
- captured fields
- fields recorded as unknown
- confirmation that no source files were modified and no secrets were
  stored

## Rules

- Never store credentials, tokens, or secrets.
- Never overwrite a different task's checkpoint.
- Never modify source files as a side effect of checkpointing.
- Never commit, push, merge, or modify remote state.
