# Session Resilience Workflow

## Goal

Allow long-running engineering tasks to survive context compaction
(`/compact`) and a new session (`/new`) without losing verified progress,
reconstructing state manually, or closing and reopening the terminal.

This workflow is the canonical definition of checkpoint and recovery
behavior. `AGENTS.md` states the policy; `commands/checkpoint.md` and
`commands/resume-work.md` are client adapters that delegate here.

Checkpointing and recovery are read-only with respect to the target
project's source tree. The only file written is the checkpoint itself, in
the user-level handoff directory.

---

## Phase 1 — Resolve the Target

Determine:

- repository root
- task identifier, when available
- branch
- base commit
- current HEAD
- working-tree state

Use local Git inspection. Do not modify the working tree to gather this
information.

If the repository root or branch cannot be established, report the missing
information instead of guessing.

---

## Phase 2 — Handoff Location

The checkpoint is written to the user-level handoff directory **outside the
target project's source tree**, using a provider-neutral root that is not
tied to any single client:

```
${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}/handoffs/<repo-slug>/<task-slug>.md
```

Rules:

- `AI_ENGINEERING_STATE_HOME` overrides the root when configured and valid;
  otherwise the default `$HOME/.ai-engineering` is used.
- Derive `<repo-slug>` from the repository root directory name.
- Derive `<task-slug>` from the task identifier. When no task identifier
  exists, `/checkpoint` generates a unique checkpoint identifier and uses it
  as the task slug. Never use the branch name as the task slug: two tasks
  on the same branch would reach the same path.
- A checkpoint may be updated only when it unambiguously belongs to the
  current task, identified by the same repository root and the same task
  identifier or generated checkpoint identifier. Never overwrite a
  checkpoint that belongs to a different task.
- Never write checkpoints inside the target project's source tree or into
  the canonical ai-engineering repository's tracked files.
- Re-checkpointing the same task may update that task's own checkpoint.

### Safe identifier normalization

Normalize every identifier that becomes a path component before deriving
a checkpoint path:

- Accept task and checkpoint identifiers composed only of ASCII letters,
  digits, `-`, `_`, and `.`, with a bounded length (up to 128 characters).
- Reject empty identifiers, path separators (`/` and `\`), the components
  `.` and `..`, and any other character outside the accepted set.
- Never concatenate an untrusted identifier directly into a path.
- After building the path, resolve it and verify that the resolved path
  still starts with the canonical handoff directory. Reject the write when
  the target escapes that directory or when any symlink along the path
  points outside it.
- Apply the same normalization to a task identifier used as the task slug
  and to any identifier supplied as a checkpoint or resume argument.

The handoff directory contains only checkpoint metadata. Never store
credentials, tokens, secrets, cookies, private keys, or private content
values in a checkpoint.

---

## Phase 3 — Checkpoint Contents

A checkpoint records at least:

- Repository root.
- Task identifier.
- Checkpoint identifier, when the task identifier is absent.
- Branch.
- Base commit.
- Current HEAD.
- Git status summary.
- Staged, unstaged, and untracked changes.
- Objective.
- Completed work.
- Decisions and rationale.
- Changed files.
- Validation evidence actually observed.
- Blockers.
- Exact next actions.

When a field cannot be determined, record it explicitly as unknown rather
than inventing a value.

---

## Phase 4 — Checkpoint Procedure

1. Resolve the target and handoff location (Phase 1 and Phase 2).
2. When no task identifier exists, generate a unique checkpoint identifier
   and use it as the task slug.
3. Inspect Git state: branch, base commit, HEAD, status, staged, unstaged,
   and untracked changes, and changed files.
4. Summarize objective, completed work, decisions, blockers, and next
   actions from the current session.
5. Record only validation evidence that was actually executed, including
   the command and its observed result. Do not record a validation as
   passed without evidence.
6. Write the checkpoint to the task-scoped handoff path.
7. Do not stage, commit, reset, stash, checkout, or edit source files as a
   side effect. Preserve all uncommitted work.

A checkpoint is a snapshot. It does not replace the working tree, and it
does not authorize any commit, push, merge, or remote action.

---

## Phase 5 — Recovery Procedure

1. Read the requested checkpoint.
2. Treat the checkpoint as a recovery guide, not as proof that the
   described state still exists.
3. Verify the actual repository root, branch, HEAD, and working-tree state.
4. Compare the verified state with the checkpoint and detect divergence,
   including a changed branch, moved HEAD, missing or additional changes,
   and validation that cannot be confirmed.
5. Preserve current work. Do not reset, stash, discard, or overwrite
   uncommitted changes to match the checkpoint.
6. Continue from the first incomplete action instead of repeating completed
   phases.
7. Never claim a prior validation passed without recorded evidence; when
   evidence is missing or stale, re-execute the relevant check or report it
   as not verified.

If the checkpoint's repository root, branch, or task does not match the
current workspace, report the mismatch and ask before proceeding.

When more than one checkpoint exists for the current repository and branch,
list the available checkpoints with their task or checkpoint identifiers and
ask which to resume instead of guessing.

---

## Phase 6 — Context Management

- Keep automatic compaction enabled.
- Prune or discard old tool output where the client supports it instead of
  disabling compaction.
- Do not introduce undocumented context-threshold configuration keys.
- Use focused file reads, bounded tool output, and concise subagent
  summaries to limit context growth.
- Checkpoint at logical phase boundaries, before expensive validations, and
  before a context reset when needed.
- Do not depend solely on a displayed context percentage to decide when to
  checkpoint.
- Never require closing and reopening the terminal as the normal recovery
  path.
- Do not add an experimental compaction plugin unless a demonstrated
  requirement cannot be met with stable built-in functionality.

### Token and Context Efficiency

Use the smallest context that is sufficient for a correct, validated result,
not the absolute minimum token count. Keep context spend proportional to the
task, independently of the client.

Before broad exploration, record a **Context Manifest** covering:

- target project root and evidence-only paths;
- task type and provisional rigor;
- Skills and references expected to be needed;
- likely files, modules, or documentation sections;
- exploration budget and stop condition.

#### Soft exploration budgets

These are initial defaults, calibrated from measured tasks rather than
intuition. They apply to **exploration before implementation** and exclude
mandatory edits, tests, formatting, build, and validation.

| Rigor | Exploration tool calls | Unique files read | Skills loaded |
|---|---:|---:|---:|
| R1 | 4 | 3 | 1 |
| R2 | 10 | 8 | 2 |
| R3 | 20 | 15 | 3 |
| R4 | explicit plan | explicit plan | explicit plan |

A budget is a soft stop-and-report threshold, not a hard failure. If
correctness, safety, or required validation needs more context, record the
reason in the working interpretation and continue. If the expansion changes
the task scope, stop and ask before proceeding.

#### Retrieval rules

- Prefer targeted searches and ranged reads over loading whole files or
  whole sites.
- Keep tool output bounded; summarize large results instead of pasting them
  back, and discard or prune stale output where the client supports it.
- Delegate broad or exploratory search to subagents and consume their
  concise summary rather than their full transcript.
- Do not re-read or re-derive information already established earlier in the
  session; reuse prior analysis when the task state has not changed.
- Load only the Skills, references, and documentation relevant to the
  current task; never preload the entire knowledge base.
- For review and audit work, inspect the diff first and expand only where
  the diff requires repository context.
- Checkpoint before a large expansion or an expensive validation so progress
  survives a context reset instead of being repeated.

#### Stop rules

- Two consecutive searches add no new evidence: summarize and decide.
- The exploration budget is reached: report what is known, what is missing,
  and the proposed next step.
- A new repository area is needed: update the Context Manifest before
  exploring it.
- Token savings would reduce correctness, safety, or validation: stop
  optimizing.

#### Measurement

Where the client exposes per-session or per-message usage, record input,
output, reasoning, cache reads and writes, tool calls, unique and repeated
file reads, Skills loaded, and compactions. Aggregate subagent sessions into
the parent task. Use `references/token-efficiency.md` for the metric
definitions and optional client-specific queries.

Metrics guide calibration; they never justify skipping required evidence,
tests, or validation.

---

## Phase 7 — Suspected Hangs

If a session appears stalled during compaction or a long operation:

- Report the observed symptoms, timestamps, and affected step.
- Do not assert an unverified root cause.
- Recommend the smallest safe next step, such as completing a checkpoint
  when possible, then recovering in a new session with `/resume-work`.
- Do not claim that closing and reopening the terminal is required.

---

## Safety

- Read-only with respect to the target project's source tree.
- The only write is the checkpoint in the user-level handoff directory.
- Never store credentials, tokens, or secrets.
- Never overwrite a different task's checkpoint.
- Never commit, push, merge, publish, or modify remote state.
- Never claim a validation passed without evidence.
