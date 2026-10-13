---
description: Recover a long-running task from a user-level checkpoint by verifying actual repository state and continuing from the first incomplete action.
agent: orchestrator
---

# /resume-work

Run the Session Resilience Workflow:

`workflows/session-resilience.md`

## Input

`$ARGUMENTS`:

- checkpoint path, task identifier, or branch

When a task identifier is supplied, normalize it the same way
`/checkpoint` does: reject path separators, `.`/`..` components, and other
unsafe characters before resolving the checkpoint. A supplied path must
resolve inside the authorized handoff directory defined in
`workflows/session-resilience.md`.

If no argument is supplied, inspect the authorized handoff directory
defined in `workflows/session-resilience.md`, list the checkpoints that
match the current repository and branch, and ask which one to resume.
Never guess or reuse the branch name to pick a checkpoint.

## Mandatory Behavior

1. Normalize a supplied task identifier into a safe file name before
   resolving the checkpoint, and verify that a supplied path resolves
   inside the authorized handoff directory. Reject identifiers or paths
   that would escape it.
2. Read the requested checkpoint.
3. Verify the actual repository root, branch, HEAD, and working-tree state.
4. Treat the checkpoint as a recovery guide, not as proof that the
   described state still exists.
5. Detect divergence and preserve current work.
6. When several checkpoints match, list them with their task or checkpoint
   identifiers and ask for the selection.
7. Continue from the first incomplete action instead of repeating completed
   phases.
8. Never claim a prior validation passed without recorded evidence;
   re-execute the relevant check or report it as not verified.

## Output

- checkpoint read
- verified state compared with the checkpoint
- divergence detected
- current work preserved
- first incomplete action
- validations re-executed or reported as not verified
- exact next actions

## Rules

- Never reset, stash, discard, or overwrite user work.
- Never create commits, push, merge, or modify remote state.
- Never claim an unverified result.
