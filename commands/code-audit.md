---
description: Read-only senior/staff whole-cycle audit of generated code for over-engineering, architecture, patterns, quality, security, data, tests, performance, a11y, i18n, SEO, UI, and mobile — without modifying code or remote state.
agent: reviewer
---

# /code-audit

Run the Code Audit Workflow:

`workflows/code-audit.md`

## Input

Optional arguments in `$ARGUMENTS`:

- File or directory paths to audit (relative to project root).
- `--all` to audit the entire project (explicit opt-in).
- If omitted, audit only uncommitted changes:
  - Staged + unstaged tracked changes (`git diff HEAD`)
  - Untracked files (`git ls-files --others --exclude-standard`)
  - If the working tree is clean: `NOTHING TO AUDIT` (no fallback to whole project).
- **Default scope is the change set only; declaration sites outside the diff are not audited.**

## Behavior

1. Resolve the project root and classify as software project or documentation-only.
2. Detect stack, app type(s), and existing conventions.
3. Build the applicability matrix for all code dimensions.
4. Load only applicable Skills and audit each dimension.
5. Produce a dimension matrix + prioritized findings + verdict.
6. Never edit, commit, push, or write remote state.

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