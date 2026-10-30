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
- Phase plan, with the phases completed and the one in progress.
- The current checklist, with the state of every step.
- The pending approval gate, when one is open.
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
7. Re-emit the phase plan and the current checklist, and re-populate the
   client's native task surface when it exposes one, **before the next tool
   call**. A resumed session that continues silently has lost the contract.
8. Never claim a prior validation passed without recorded evidence; when
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
- **The operating contract survives compaction.** The summary preserves the
  phase plan, the current checklist, and any pending gate. A summary that keeps
  the objective and drops them leaves the session without the rules it was
  following.
- **After a compaction, a server restart, or an automatic model switch**, the
  plan and the checklist are re-emitted and the native surface re-populated
  before the next tool call, so the session continues under the same contract.
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
tests, or validation. **Never trade correctness, safety, or validation for
lower token use.** The canonical wording is in `AGENTS.md`.

---

## Phase 7 — Suspected Hangs

If a session appears stalled during compaction or a long operation:

- Report the observed symptoms, timestamps, and affected step.
- Do not assert an unverified root cause.
- Recommend the smallest safe next step, such as completing a checkpoint
  when possible, then recovering in a new session with `/resume-work`.
- Do not claim that closing and reopening the terminal is required.

---

## Phase Plan

Before the work starts, in every task, the agent shows the **phase plan**, so
the user knows the sequence, the outcome of each phase, and where the work can
stop. A task with one phase still shows its one-phase plan: a plan with zero
phases is a defect, and there is no maximum number of phases.

The obligation is **independent of the entry point**. A command, a plain
message in a new session, a resumed checkpoint, a subagent, and an automation
all start the same way: the phase plan and the initial checklist of the first
phase are emitted **before the first tool call**. Inspecting the repository in
order to plan later is a tool call, so it happens after the plan, never
before. A command only restates the obligation at its invocation point; a
session that shows the plan only when a command was used has failed the rule.

The **phase plan** is the sequence of phases. It is not the **task plan**,
which is the planning artifact of `workflows/task-planning.md` and says what
the task does. The phase plan says in which order it happens, and the
**phase checklist** of the phase in progress says what is being done inside
it. All three can be shown together: the phase plan for the position, the
checklist for the detail.

### Format

```
[   ] 1. Detect and analyze the repository
[   ] 2. Agree the requirements and the approach
[   ] 3. Evaluate the architecture
[   ] 4. Implement
[   ] 5. Review and validate
```

- `[   ]` the phase has not started, `[ x ]` it is complete, `[ ! ]` it is
  blocked.
- Each phase names **the outcome it produces**, not only its name.
- The phases are the canonical phases of `workflows/task-to-code.md`,
  **restricted to the ones this task actually needs**. The sequence is
  derived from the task: a documentation-only change runs a short plan, a
  new service runs a long one, and a task small enough to fit in one phase
  shows a one-phase plan, never no plan.
- The plan is shown once, before the work starts. It is refined only when
  the scope changes, and the refinement is presented and approved again.
- The plan is session state, and it does not require a commit.

### Phase boundary report and phase gate

At the end of each phase, the report states:

- what the phase produced, with the checklist of that phase;
- what remains: the pending items of the phase and the phases that follow;
- what the next phase adds, and the exact resume point.

When another phase remains, the boundary is an **approval gate**. The agent
asks, in the current conversation:

> Do you approve starting phase N?

The next phase starts only after an affirmative answer that follows that
question. These are not approval for the transition: continuing the work, the
original task request, an earlier "continue", an earlier approval of the plan,
silence, a timeout, a resumed checkpoint, or an automated retry. Approval
covers exactly the next phase; if its scope or the plan changes, it is
presented again and approved again. If the user declines or asks for changes,
the next phase does not start; the plan is adjusted and the question asked
again.

The last phase has no transition to approve: its result is delivered, and the
commit approval gate applies if a commit is proposed. A single-phase task
therefore has a one-phase plan, its checklist, and no transition gate.

A phase that ends with pending work is not a failure to report as done.
The boundary report is what makes stopping a deliberate choice, and stopping
requires no commit.

## Phase Checklist

While a phase is in progress, the agent shows **what it is doing, step by
step**, so the user can follow the work without committing after every
phase.

The **phase checklist** is not the **phase plan**. The plan lists the phases
the task goes through; the checklist lists the steps of the phase in
progress. A checklist is never the plan restated, and the plan is never
passed off as a checklist.

### Format

The lines below illustrate the **format only**. Their content is an example
of how a line reads, never a template: the steps are derived from the work
of the phase and are never reused.

```
[ x ] Creating the store reducers for the global cart state
[   ] Wiring the MSAL sign-in flow to the session endpoint
[   ] Building the shopping cart view over the store selectors
```

- `[ x ]` the step is done and verified.
- `[   ]` the step is pending.
- `[ ! ]` the step is blocked or failed, followed by the reason.

The canonical rendering is `[   ]` with three spaces, so the states align. The
checks accept a single space (`[ ]`) as the same pending state, because a
spacing difference is never the defect worth reporting; the meaning of the
state is.

The checklist is **session state**. It is shown in the conversation and is
not written to the repository, so following the work never requires a
commit.

### Derivation

The steps are **detected from the work of the current phase**, never taken
from a fixed template. The same phase produces different steps in different
projects, and the same project produces different steps in different phases.

**Source of the steps.** The derivation is a chain, and it always starts from
the **task plan** and the file responsibility map, never from the phase
plan:

1. The **task plan** of `workflows/task-planning.md`, and the **File
   Responsibility Map** when the task has one, name the artifacts the phase
   must produce and verify.
2. The **phase scope** narrows that list to what this phase covers.
3. One step is emitted per artifact, in the order it is executed, stating
   what is being done to it.

There is no per-phase template anywhere in the system. A step is never
carried over from another project, another phase, or another run. When a
phase has no plan yet, derive its steps from the phase's own scope, record
them in the first status update of that phase, and refine them if the scope
changes.

Each step is one action that produces or verifies a **concrete thing**:

- a file, a type, a contract, an endpoint, a reducer, a migration, a test, a
  configuration value, a document section;
- named specifically, in the order it is executed.

The artifact comes from the task plan the step was derived from, so the step
text states what is being done without repeating the path: "the store
reducers", "the MSAL sign-in flow", "the cart view".

A step that is only the name of a phase is **not a step**. Phase names are
not steps, and a checklist of phase names tells the user nothing. A checklist
whose steps are all generic is a defect.

### Emission and cadence

The checklist is a **live view of the work in progress**, not a summary shown
when the work is over. It leads every update that reports a change, and it is
re-emitted at every step transition:

| Moment | What is emitted |
|---|---|
| The phase starts | The full derived checklist, every step pending |
| A step is about to change the repository | The checklist, so the panel shows the step in progress before it runs |
| A step is completed | The checklist with `[ x ]` on that step, leading the update |
| A step is blocked or fails | The checklist with `[ ! ]` and the reason, leading the update |
| A new step is discovered | The checklist with the step inserted in execution order |
| Each user turn while the phase is open | The checklist as it currently stands |
| The phase ends | The final state, with every pending step reported |

Rules:

- The checklist leads the update. It is never appended as a closing summary
  after the work it describes.
- It is never held back until the task or the phase is finished. A change
  reported without its checklist is an incomplete update.
- The cadence is a lower bound, not a schedule: when several steps complete
  between two messages, the next message carries the checklist with all of
  them marked.
- A mutating step is announced by the checklist before it runs, not after it
  succeeds. A panel that only changes once the result is known is a history,
  not a live view.
- It is the same format everywhere: the creation tracker, an implementation
  phase, a review, and a validation run.
- A phase that ends with pending steps reports them, so the next session
  resumes from the same place.

### Native task surfaces

When the client exposes a native task or todo surface — a `todowrite` tool
that renders a persistent todo panel, or an equivalent — the mirror is
**mandatory, not optional**. It is populated with the initial checklist and
updated on the same transitions, before each mutating step, so progress is
visible without reading every message.

- The **conversation copy is the authority**. When the mirror is disabled,
  unavailable, or fails, the checklist continues in the conversation and the
  failure is reported instead of being silently dropped.
- A surface that exists but is never populated is a **defect of the session**,
  even when the conversation carries the checklist. Two calls at the start and
  the end are not a live mirror.
- A surface that renders only the client's own state — the list of configured
  MCP connections, a status bar, a model selector — is **not writable**. No
  instruction makes an agent able to write into it, and the agent never claims
  an update there that it cannot perform.
- A surface that only a client plugin can extend is reported as such, with the
  plugin as the path, instead of being simulated or described as done.

The mirror never replaces the cadence above: it makes the same checklist
persistent, and an out-of-date mirror is corrected on the next transition
rather than trusted.

### Scope

The checklist covers the phase in progress. It does not replace the phase
plan, the proposal, or the final report; it is the live view of the work
between them.

### Scenarios the checklist must cover

The derivation is framework, language, flow, and task independent, so the
checklist must work in every kind of phase. These scenarios are the
coverage, and each one yields a different checklist:

| Scenario | Shape of the phase |
|---|---|
| A new project from scratch | One slice at a time, with the tooling and the contracts |
| A bug fix in an existing project | The fix, then the test that proves it |
| A new API endpoint | The contract, the handler, the request that proves it |
| A new feature in an existing project | The contracts, then the service, then the view |
| A structural refactor | The extraction, then the consumers that change |
| Performance work | The measurement, then the change that answers it |
| A database migration | The migration, then the verification on an empty database |
| A dependency upgrade | The manifest change, then the compatibility check |
| A documentation-only change | The section that changes, and the link that proves it |
| An accessibility fix | The element, and the behavior that satisfies it |
| Localization | The catalog, and the layout that adapts to it |
| Adding tests | The test, and the behavior it locks |
| A CI/CD change | The pipeline step, and the check it runs |
| A cross-cutting configuration change | The setting, and the place that reads it |

A checklist whose steps are all phase names is a defect in any of them.

## Safety

- Read-only with respect to the target project's source tree.
- The only write is the checkpoint in the user-level handoff directory.
- Never store credentials, tokens, or secrets.
- Never overwrite a different task's checkpoint.
- Never commit, push, merge, publish, or modify remote state.
- Never claim a validation passed without evidence.
