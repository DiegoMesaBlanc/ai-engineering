# External Audit Workflow

## Purpose

Audit a deliverable **we** produced, from outside the context that produced
it.

The auditor did not participate in the implementation, does not receive the
implementation reasoning, re-executes every claim instead of trusting it, and
judges the result against the original requirement and the canonical
engineering rules.

The question this workflow answers is not "is this code good?" but **"did we
actually deliver what was asked, and is it right?"**

This workflow is **not**:

- `workflows/code-audit.md` — that audits code written by someone else
  (a colleague, another AI, a supplied change set). Keep it for foreign code.
- `workflows/pull-request-review.md` — that reviews a colleague's PR.
- `workflows/project-validation.md` — that executes the project's gates.
- `skills/engineering-review/SKILL.md` — that reviews our own change inside
  the same implementation flow, so it is not independent.
- `workflows/system-audit.md` — that audits the AI Engineering System itself,
  not a deliverable.

## Independence Contract

Independence is structural, not a tone of voice. All seven rules are
mandatory.

| # | Rule | How it is satisfied |
|---|---|---|
| I-1 | **Separate session** | The audit runs in a context that did not produce the deliverable. OpenCode: the `external-auditor` subagent with `subtask: true`. Codex and Claude: a new process or session. Same client or a different client is irrelevant. |
| I-2 | **Cold input** | The auditor receives the requirement and the artifact, never the implementation reasoning. |
| I-3 | **Read-only** | No edits, fixes, commits, branches, pushes, or remote writes. |
| I-4 | **Adversarial stance** | The auditor assumes the deliverable is wrong until evidence proves otherwise. Absence of findings requires executed checks, not an impression. |
| I-5 | **Re-execution** | Every claim in the delivery report is re-run. A claimed PASS is a hypothesis, never evidence. |
| I-6 | **Frozen scope** | The scope manifest is recorded before inspection and does not change after seeing the code. |
| I-7 | **Declared limit** | Independence is contextual, not epistemic. The same model keeps the same blind spots. State this in the report. |

### Allowed input

- The artifact root and the scope (paths, `--all`, or a commit range).
- The original task or requirement wording and its acceptance criteria.
- The design artifact, when the task had one.
- The canonical rules: `AGENTS.md`, the applicable Skills, the references.

### Denied input

The auditor must not receive, and must not read when it is discoverable:

- the implementation conversation, transcript, or session history;
- the task plan, architecture proposal, file responsibility map, Folder
  Architecture Map, or pattern selection record;
- the implementer's final report, validation claims, or PR description;
- the implementer's explanation of why a choice was made.

If the context already contains that material, stop and request a fresh
session. An audit performed after reading the implementer's reasoning is not
an external audit.

## Phase 0 — Independence Gate

1. Confirm the session did not produce the deliverable.
2. Build the **Input Manifest**: what was received and what was withheld.
3. If the context contains implementation history, report
   `NOT INDEPENDENT — FRESH SESSION REQUIRED` and stop.
4. Record the client, the session, the model, and the date.

## Phase 1 — Artifact and Scope

- Resolve the **artifact root**. It is the target project root. It is never
  `AI_ENGINEERING_HOME`; that variable only locates the canonical rules.
- Resolve the **Git root** separately: it may differ from the artifact root.
  An artifact without Git is audited as a directory and declared as such.
- Resolve the scope: explicit paths, `--all`, or a commit range. Record the
  **Scope Manifest** before inspection.
- Record the branch, the commit, the file count, and the exclusions.
- Do not widen or narrow the scope after inspection starts.

## Phase 2 — Requirement Reconstruction

Rebuild the requirement from its original wording, without the
implementation plan:

1. Extract the acceptance criteria, constraints, non-goals, and explicit
   decisions the task stated.
2. Build the **Acceptance Matrix**: one row per criterion, with the evidence
   that would prove it and how it will be observed.
3. Mark a criterion `AMBIGUOUS` when the requirement does not settle it, and
   say so in the report instead of inventing an interpretation.
4. Do not derive criteria from the implementation. That would make the
   deliverable judge itself.

## Phase 3 — Stack and Architecture

Detect from evidence only: language, runtime, framework, version, package
manager, build system, architecture style, app type, and test layout. Record
confidence. Do not read the implementer's architecture proposal.

## Phase 4 — Deterministic Re-execution

Run the project's own gates before any semantic judgment. Use the detected
tooling and the project's own scripts:

- format check, lint, typecheck, unit and integration tests, build;
- structural compliance from `workflows/output-compliance.md`, in the form
  the detected ecosystem supports (`FUNC-001`, `TYPE-001`, `TYPE-002`,
  `TYPE-003`, `FILE-001`, `STRUCT-001`, `STRUCT-006`, `STRUCT-007`);
- the contract-consumer script when the project installs it;
- the documentation gate (`scripts/check-docs`) when the project installs it;
- secret scanning and dependency/license state when the project has tooling
  for it.

Record for every check: the command, the exit code, and the relevant output.
A non-zero exit is a finding with the command and the output. A check that
cannot run is `NOT_VERIFIED` with the missing tool named. **A skipped check is
never a pass.**

## Phase 5 — Claim Falsification

For every claim in the delivery report, produce one row:

| Claim | How it was re-checked | Result |
|---|---|---|
| "tests pass" | command and exit code | `CONFIRMED` / `FALSIFIED` / `NOT VERIFIED` |
| "lint clean" | command and exit code | … |
| "build succeeds" | command and exit code | … |
| "no regressions" | regression tests or explicit gap | … |
| "requirement met" | acceptance matrix | … |

A `FALSIFIED` claim is a P1 finding regardless of the code quality: the
delivery was misrepresented.

## Phase 6 — Semantic Audit

Apply the dimension definitions of `workflows/code-audit.md` Phase 3–6 by
reference. This workflow does not restate them.

Audit at the senior/staff bar, including the defects a deterministic check
cannot see:

- dead exported types, unused contracts, duplicated contracts for the same
  shape;
- logic that is syntactically valid but semantically wrong;
- acceptance criteria satisfied only on the happy path;
- error handling that swallows the failure;
- security assumptions that hold only in the demo;
- design choices that solve the demo and not the requirement.

These are the findings that justify the two-layer design: the machine catches
the objective minimum, the auditor catches the meaning.

## Phase 7 — Acceptance Verdict

Complete the Acceptance Matrix from Phase 2. Each criterion is `PASS`,
`FAIL`, or `NOT VERIFIED` with concrete evidence. A deliverable cannot be
accepted while a criterion is `FAIL` or silently `NOT VERIFIED`.

## Phase 8 — Report

Produce, in this order:

1. **Independence declaration**: client, session, model, date, and the Input
   Manifest (received / withheld).
2. **Scope**: artifact root, Git root, branch, commit, files, exclusions.
3. **Acceptance Matrix**: criterion → PASS/FAIL/NOT VERIFIED → evidence.
4. **Deterministic results**: command, exit code, output for every check.
5. **Claim falsification table**.
6. **Dimension matrix**: the applicable dimensions with PASS/WARN/FAIL/N/A
   and an explicit N/A reason.
7. **Findings**: P0–P3, `file:line`, evidence class (REAL / SIMULATED /
   STATIC), impact, recommended correction, confidence.
8. **Verdict**: one of
   - `ACCEPTED` — every criterion PASS, no P0/P1 finding;
   - `ACCEPTED WITH FINDINGS` — every criterion PASS, no P0, at most one P1;
   - `REJECTED` — any criterion FAIL, any P0, or more than one P1;
   - `NOT VERIFIED` — a critical check or criterion could not be evidenced.
9. **Limits**: what could not be verified and why, and the I-7 statement.

## Evidence Classes

- **REAL** — executed against the artifact (tests ran, linter output, build
  output).
- **SIMULATED** — fixture or mock based.
- **STATIC** — inspection without execution.

A static check does not prove runtime behavior. When a check cannot run,
record `NOT_VERIFIED` with the reason.

## Read-only Policy

Never:

- edit or fix source files;
- create commits, branches, or PRs;
- push or publish anything;
- modify the working tree;
- install dependencies or tools;
- claim a check passed without executed evidence.

The human user performs any correction in a separate authorized flow.

## Relationship to the Task-to-Code Flow

The external audit runs **after** the deliverable is complete, in a separate
session. It does not replace `workflows/task-to-code.md` Phase 12 (review);
it verifies that phase's output from outside. Corrections found here return
to a new Task-to-Code flow, never to this one.
