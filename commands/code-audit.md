---
description: Read-only senior/staff audit of the last commit by default, or of the whole project with --all, reporting improvement suggestions for architecture, patterns, quality, security, data, tests, performance, a11y, i18n, SEO, UI, and mobile.
agent: reviewer
---

# /code-audit

Run the Code Audit Workflow:

`workflows/code-audit.md`

## Input

Two modes only:

- Without arguments, audit the **last commit** (`HEAD`).
- `--all` to audit the **whole project** in its current state.

Rules:

- If the repository has no commit, report `NOTHING TO AUDIT`.
- The working tree is not audited in commit mode; `--all` audits the current
  project, including uncommitted changes.
- Findings are **improvement suggestions** ordered by impact. They are never
  applied automatically and do not block a commit by themselves.
- No other scope mode exists. Do not add a third mode or a fallback between
  them.
- Auditing a deliverable **we** produced is not a scope of this command. It
  is the Independent Mode of the same workflow, invoked through the
  `external-auditor` agent in a fresh context (see `agents/external-auditor.md`).

## Behavior

1. Resolve the project root and classify as software project or documentation-only.
2. Detect stack, app type(s), and existing conventions.
3. Build the applicability matrix for all code dimensions.
4. Load only applicable Skills and audit each dimension.
5. Produce a dimension matrix + prioritized suggestions + overall assessment.
6. Never edit, commit, push, or write remote state. Never check out a commit or
   modify the working tree to inspect one.

## Output

- Dimension matrix: PASS / WARN / FAIL / N/A with N/A reason.
- Findings grouped by severity (P0–P3) with file:line, evidence, impact, recommended correction, confidence.
- Verification gaps (checks not run).
- Verdict: PASS / PASS WITH FINDINGS / FAIL / NOT VERIFIED.

## Read-only Policy

Do not:
- edit or fix source files
- create commits or branches
- push or publish anything
- modify the working tree
- install dependencies