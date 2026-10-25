---
description: Independent read-only audit of a deliverable we produced, from a fresh session that did not implement it, with re-executed evidence, claim falsification, and an acceptance-criteria verdict.
agent: external-auditor
subtask: true
---

# /external-audit

Run the External Audit Workflow:

`workflows/external-audit.md`

## Input

Optional arguments in `$ARGUMENTS`:

- File or directory paths to audit (relative to the artifact root).
- `--all` to audit the whole artifact.
- `--range <a>..<b>` to audit a commit range.
- `--task <reference>` to locate the original requirement.
- If omitted, audit the artifact's uncommitted change set; a clean tree is
  `NOTHING TO AUDIT` unless the requirement points to a commit range.

The original task or requirement wording is a required input. Supply it with
the invocation or with `--task`. The auditor must not receive the
implementation plan, the pattern selection record, the validation report, or
the implementation conversation.

## Behavior

1. Enforce the independence gate and build the Input Manifest.
2. Resolve the artifact root separately from `AI_ENGINEERING_HOME`.
3. Rebuild the acceptance criteria from the original requirement.
4. Re-execute the project's gates and the structural compliance checks.
5. Falsify every claim in the delivery report.
6. Audit the applicable dimensions by reference to `workflows/code-audit.md`.
7. Produce the acceptance matrix, the findings, and the verdict.
8. Never edit, commit, push, or write remote state.

## Modes

`agent: external-auditor` plus `subtask: true` forces a fresh child context,
so this command is safe to invoke from the session that implemented the
deliverable.

The `external-auditor` agent is also `mode: all`, so it can be selected as
the **Auditor** mode with Tab. That path does not reset the session history:
use it in a **new session**, never in the one that implemented the
deliverable.

## Output

- Independence declaration and Input Manifest.
- Scope: artifact root, Git root, branch, commit, files, exclusions.
- Acceptance Matrix: PASS / FAIL / NOT VERIFIED with evidence.
- Deterministic results: command, exit code, output.
- Claim falsification table.
- Dimension matrix with N/A reasons.
- Findings P0–P3 with `file:line`, evidence class, impact, correction, and
  confidence.
- Verdict: `ACCEPTED` / `ACCEPTED WITH FINDINGS` / `REJECTED` /
  `NOT VERIFIED`.
- Limits, including the I-7 statement.

## Read-only Policy

Do not:

- edit or fix source files;
- create commits or branches;
- push or publish anything;
- modify the working tree;
- install dependencies or tools.
