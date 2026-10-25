---
name: external-audit
description: Independently audit a deliverable we produced, in a fresh session that did not implement it, re-executing every claim and judging the acceptance criteria. Use when asked to "audit our own work", "external audit", "independent audit", "verify what we delivered", or "did we actually build what was asked".
---

# External Audit

The canonical procedure is `workflows/external-audit.md`. Follow it; do not
restate it.

## When this applies

The deliverable was produced by us or by an agent in our session, and it
must be judged from outside. The goal is to detect what the implementing
context cannot see: overclaiming, dead code, duplicated contracts, a happy
path that hides a failure, and acceptance criteria that were assumed instead
of verified.

## When this does not apply

- Code written by someone else → `workflows/code-audit.md`.
- A colleague's pull request → `workflows/pull-request-review.md`.
- Running the project's gates only → `workflows/project-validation.md`.
- The AI Engineering System itself → `workflows/system-audit.md`.

## Non-negotiable

- The audit runs in a session that did not produce the deliverable.
- The auditor receives the requirement and the artifact, never the
  implementation plan, the rationale, or the delivery claims.
- The auditor is read-only.
- Every claim is re-executed before it is believed.
- A skipped check is `NOT_VERIFIED`, never a pass.

## Output

The acceptance matrix, the deterministic results, the claim falsification
table, the dimension matrix, severity-ranked findings with `file:line`, and
one verdict: `ACCEPTED`, `ACCEPTED WITH FINDINGS`, `REJECTED`, or
`NOT VERIFIED`.
