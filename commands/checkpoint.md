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

- task identifier, used as the checkpoint slug; when absent, `/checkpoint`
  generates a unique checkpoint identifier
- checkpoint path, accepted only when it resolves inside the authorized
  handoff directory `$HOME/.config/opencode/handoffs/`; reject any other
  destination
- milestone or note label

If no argument is supplied, use the current repository and branch, and
generate a checkpoint identifier when no task identifier exists.

## Mandatory Behavior

1. When no task identifier is supplied, generate a unique checkpoint
   identifier and use it as the checkpoint slug. Never use the branch name
   alone as the identifier.
2. Accept a supplied checkpoint path only when it resolves inside the
   authorized handoff directory; reject destinations inside the target
   project's source tree or outside the handoff directory.
3. Resolve the repository root, task identifier, branch, base commit, HEAD,
   and working-tree state.
4. Write to `$HOME/.config/opencode/handoffs/<repo-slug>/<task-slug>.md`,
   outside the target project's source tree.
5. Include objective, completed work, decisions, changed files, validation
   evidence actually observed, blockers, and exact next actions.
6. Record fields that cannot be determined as unknown instead of inventing
   values.
7. Preserve uncommitted work. Do not stage, commit, reset, stash, checkout,
   or edit source files.

## Output

- checkpoint path
- task identifier or generated checkpoint identifier
- branch
- captured fields
- fields recorded as unknown
- confirmation that no source files were modified and no secrets were
  stored

## Rules

- Never store credentials, tokens, or secrets.
- Never overwrite a checkpoint that belongs to a different task.
- Reject checkpoint paths outside the authorized handoff directory.
- Never modify source files as a side effect of checkpointing.
- Never commit, push, merge, or modify remote state.
