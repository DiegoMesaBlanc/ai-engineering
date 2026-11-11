# Self-Learning Workflow

## Goal

Let the AI Engineering System improve its own effectiveness over time by
turning observations from real work into durable knowledge: capture small
lessons, consolidate them into a trustworthy set, and promote the stable,
general ones into the canonical knowledge layer through an explicit human
approval.

This workflow is provider-neutral. It is not tied to OpenCode, Claude,
Codex, Aider, or any single client; clients only resolve the store path and
follow the phases below.

This workflow is the canonical definition of capture, consolidation, and
promotion behavior. It is never a retrospective of people or teams; it is
about the system's own engineering knowledge.

---

## Neutral Store

Learnings are written to a user-level directory **outside every target
project's source tree** and outside the canonical ai-engineering repository,
using a provider-neutral root:

```
${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}/learnings/<scope>/<entry>.md
```

Rules:

- `AI_ENGINEERING_STATE_HOME` overrides the root when configured and valid;
  otherwise the default `$HOME/.ai-engineering` is used.
- `<scope>` is either `global` for system-wide lessons or `<repo-slug>` for
  project-specific ones, derived from the repository root directory name.
- The store holds only learning metadata and prose. Never store
  credentials, tokens, secrets, cookies, private keys, personal data, or
  private content values.
- Never write learnings inside a target project's source tree or into the
  canonical repository's tracked files.
- Normalize every identifier used as a path component: accept only ASCII
  letters, digits, `-`, `_`, and `.`, with a bounded length; reject empty
  values, path separators, `.`/`..`, and other unsafe characters. After
  building a path, resolve it and verify it still starts with the neutral
  root; reject writes that escape it or follow a symlink outside it.

---

## Ledger

The ledger is the index of the store. It lets a task decide what applies and
whether a lesson is enforced without reading every entry.

- **File:** `${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}/learnings/index.tsv`,
  tab-separated, one line per entry, with a header line.
- **Columns:** `id`, `scope`, `status`, `check`, `source`, `date`, `path`. The
  entry prose stays in its own file under `learnings/<scope>/`; the index holds
  only the metadata.
- **ID:** `L-<yyyymmdd>-<slug>`, where the slug is ASCII letters, digits, and
  hyphens, with a bounded length. The same normalization applies to every path
  component (Neutral Store rules).
- **Statuses and transitions:**

| Status | Meaning | Reached by |
|---|---|---|
| `observed` | One signal, not yet confirmed | Capture from a finding |
| `aprendido` | Confirmed by evidence, has an entry, no enforcing check yet | Evidence confirmed |
| `enforced` | Has a check in the repository, and the check passes | Gap closure merged into the working tree and `system-audit` passes for that check |
| `repetido` | An enforced entry whose check failed on a task | Task check failed; the check is reopened as a gap |
| `superseded` | The evidence contradicts the entry | Evidence contradicts it; the replacement ID is recorded |

- **Deduplication:** before a new row is added, the normalized title is compared
  with the existing rows. A match is merged into the existing row, never added
  as a duplicate.
- **Scope:** an entry is `global` only when its evidence is not specific to one
  project. An entry whose evidence names one project keeps that project's scope.
- **Writes:** only the learning cycle of `workflows/task-to-code.md` writes the
  index. Each write replaces the file as a whole after the change is computed,
  so an interrupted run never leaves a half-written row.

## Phase 1 — Apply (Consume)

**Read the store before acting.** A lesson that is never consulted cannot
prevent a repeat, and learning that is not applied is not learning.

At the start of every task, before analysis or implementation:

1. Resolve the store root and the scope for the task: `<repo-slug>` for the
   project, plus `global` for system-wide lessons.
2. Read the entries of both scopes. They are small prose files; read them
   completely.
3. Apply every entry that is relevant to the task, and say which ones
   applied when reporting.
4. When an entry applies, follow it. **Repeating a lesson the store already
   records is a defect of the task, not bad luck.** The task is not complete
   until the repetition is corrected.
5. When an entry does not apply, say why in one line. A silent skip hides a
   contradiction between the store and the evidence.

An entry that the evidence contradicts is not obeyed: follow the evidence,
and report the entry as superseded in Phase 2. The store is evidence-ranked,
not authoritative.

A store that does not exist, or an empty scope, is not an error. Report
"no applicable learnings" and continue. Never fail a task because the store
is empty.

---

## Phase 2 — Capture

Record observations while they are fresh, as small independent entries.
Capture sources include:

- recurring corrections or preferences from the user
- project-specific conventions, quirks, or constraints
- implementation or design patterns that worked, or failed, and why
- recurring friction in the system's own workflows, Skills, or prompts
- validation surprises, tooling gotchas, and provider behavior

Each entry records at least:

- scope (`global` or `<repo-slug>`)
- a short, specific statement of the lesson
- the context in which it was observed
- the supporting evidence actually observed
- confidence and whether it is confirmed or a hypothesis
- the date and, when available, the task and commit it came from

Do not record speculation as fact. Do not record secrets or personal data.
Do not create a learning for a one-off event with no reusable signal.

---

## Evaluate Findings (every task)

The learning cycle of `workflows/task-to-code.md` ("Learning Cycle") classifies
each finding of the independent audit with this table. The class decides the
action; no class captures a one-off event.

| Class | Test | Action |
|---|---|---|
| Task defect | The finding is in code the task creates or changes, and no store entry covers it | Fix it in the task |
| System gap | An existing rule, check, or workflow of ai-engineering would not have caught it | Record it and close it (Phase 4) |
| Learned, not enforced | A store entry covers it, but its check did not catch it | Close the check gap (Phase 4) |
| Repetition | A store entry is enforced and its check failed on this change | Defect of the task, and a check bug to record |
| No action | Not reusable, or a one-off with no reusable signal | Report only |

A finding is a system gap only when the evidence names the missing rule or
check. Otherwise it stays a task defect.

## Phase 3 — Consolidate

Merge, deduplicate, and refine captured entries so the store stays small and
trustworthy:

- merge entries that state the same lesson; prefer the clearest wording
- resolve contradictions, preferring the better-evidenced entry and marking
  the weaker one superseded
- discard stale or disproven entries
- keep the scope correct; promote project-specific lessons to `global` only
  when evidence shows they are not project-specific
- keep the store compact; it is a curated set, not a log

---

## Phase 4 — Promote (Proposal Only)

A lesson is promotable when it is stable, general enough to matter beyond
one context, and supported by evidence.

Promotion means changing the canonical knowledge layer — a Skill, a workflow,
`AGENTS.md`, or a reference — to close a gap the evaluation found. It is applied
automatically with these limits:

- the change lands only in the working tree of the ai-engineering repository,
  on the branch the user prepared; it is never committed, pushed, or merged
  without explicit approval under the Commit Approval Policy
- present the change, the lesson it encodes, and the evidence in the report
- validate it with `workflows/system-audit.md` before it is reported
- follow the normal branch, dependency, architecture, and validation
  policies for any canonical change
- record the outcome; when a proposal is rejected, keep or revise the
  learning entry rather than silently discarding the observation

Never promote a lesson as a change that bypasses the system's own
guardrails, and never weaken a safety, quality, or cost rule in the name of
learning.

---

## Phase 5 — Maturity Model

Maturity describes the system's demonstrated reliability on a scope. It is
evidence-based and descriptive; it never grants authority and is never
self-declared.

### The ladder

| Level | Name | Demonstrated behavior |
|---|---|---|
| 1 | **Junior** | Needs explicit, detailed instructions. Frequent correction. Low autonomy. Repeats known lessons. |
| 2 | **Semi Senior** | Handles routine work with close review. Occasional correction. Applies most known lessons. |
| 3 | **Senior** | Autonomous on routine work. Reviews significant changes. Rare correction. Applies every applicable lesson. |
| 4 | **Staff** | Autonomous within policy. Surfaces risks and decisions. Consistent, evidence-based validation. Improves the system by capturing, consolidating, and promoting knowledge without weakening guardrails. |

### The execution floor is Staff, always

**Every task is executed to a Staff standard, however simple it is.** The
level above describes how much *direction and review* the work needs, never
how much *quality* it gets.

- **Execution quality is constant.** Professional best practices, typing,
  error handling, security, and appropriate tests are applied to every
  change, from a one-line fix to a new service.
- **Process rigor is proportional.** The depth of planning, the number of
  checkpoints, and the review intensity scale with the task's complexity and
  risk.
- A level below Staff lowers the amount of supervision, never the standard
  of the delivered code.

### Deriving and reporting the level

The level for a scope rises or falls on evidence:

- validation and review pass rates;
- the frequency and severity of corrections, including repeats of a lesson
  the store already records;
- adherence to policies, decision gates, and the commit approval policy;
- the honest reporting of unverified, blocked, or skipped work;
- the number of learnings promoted, and whether the promoted ones held.

Store the current level and its evidence at
`${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}/maturity/<scope>.md`,
with the date of the assessment. The learning cycle updates it after every
task with the counts it already has: tasks run, findings by class, repetitions,
enforced entries, and superseded entries. The file is created on the first task
that produces a count, never before. Report the level with its evidence in the
task report. Never claim a level the evidence does not support, and never
treat a higher level as permission to skip a check.

---

## Safety

- Provider-neutral; never depend on a single client's path or feature.
- Read-only with respect to target project source trees; the only write is
  the learning entry in the neutral store.
- Never store secrets, credentials, or personal data.
- Never overwrite or delete unrelated entries or another scope's store.
- The canonical repository is edited only by a gap closure of Phase 4: in its
  working tree, validated, and never committed without approval.
- Never weaken safety, quality, or cost guardrails in the name of learning.
- Never present a hypothesis, or an unverified result, as established
  knowledge.
