# Context: review mode

Load this bundle for subagents reviewing an implemented change set, a design,
or a pull request. Review is read-only: the subagent returns evidence-based
findings; it does not edit code, publish comments, approve, or merge.

## Always active (kernel pointers)

- `AGENTS.md` — mandatory guardrails, including workspace safety and the
  commit and phase approval gates.
- `references/audit-pass-criteria.md` — the bar each audit dimension must
  reach before code the task creates is delivered.
- `workflows/design-review.md` — for pre-code reviews; `workflows/code-audit.md`
  — for independent implementation audits.

## On demand by capability

- Engineering quality: `skills/engineering-review/SKILL.md`,
  `skills/best-practices/SKILL.md`.
- Security: `skills/security-review/SKILL.md`,
  `skills/vulnerability-catalog/SKILL.md`.
- Testing: `skills/testing-strategy/SKILL.md`,
  `skills/test-driven-development/SKILL.md`.
- Compliance sweep: `workflows/output-compliance.md` via `compliance-runner`.
- PR content: `skills/pull-request/SKILL.md` via `writer-pr`.

## Independence

A review in Independent Mode must have a fresh context that does not contain
the author's conversation; otherwise independence is `NOT ESTABLISHED` and the
reviewer stops. Reusing a reviewer's own session to request its full report is
allowed; reusing it to re-review after a design change is not.

## Boundaries

- Reviewers are read-only; no edits, commits, comments, approvals, merges, or
  remote writes.
- A finding carries `file:line` evidence and a severity; an open P1 blocks.
- Budgets are report-only; a time-controlled non-response is `NOT VERIFIED`,
  not a pass and not a silent skip.
