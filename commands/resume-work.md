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

If no argument is supplied, inspect `$HOME/.config/opencode/handoffs/` and
select the checkpoint matching the current repository and branch, or ask
which checkpoint to resume.

## Mandatory Behavior

1. Read the requested checkpoint.
2. Verify the actual repository root, branch, HEAD, and working-tree state.
3. Treat the checkpoint as a recovery guide, not as proof that the
   described state still exists.
4. Detect divergence and preserve current work.
5. Continue from the first incomplete action instead of repeating completed
   phases.
6. Never claim a prior validation passed without recorded evidence;
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
