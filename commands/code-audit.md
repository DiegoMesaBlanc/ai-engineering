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
- If omitted, audit the current change set (Git status + diff); if the working tree is clean, audit the whole project.
- **For architecture/folder structure and type/contract ownership, consider passing the containing module or `src/` to include declaration sites outside the diff.**

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