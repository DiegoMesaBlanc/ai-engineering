---
name: engineering-baseline
description: Adopt an opt-in engineering baseline that makes onboarding work from the repository itself instead of depending on an existing team member explaining the project from memory, and reduces the configuration a project repeats at the start. Use when asked to "adopt a baseline", "set up onboarding", "make a project easy to join", "bootstrap a project", "add a docs gate", "record architecture decisions", or "onboarding depends on someone explaining the project".
---

# Engineering Baseline

The canonical procedure is `workflows/engineering-baseline.md`. Follow it; do
not restate it.

## When this applies

A software project, new or existing, wants the baseline that closes one
specific failure: onboarding that only works when an existing team member
explains the project from memory. It also reduces the configuration repeated
at the start.

It applies in three situations:

- a new project, during the startup decision set of
  `workflows/project-initialization.md`;
- an existing project, through `/adopt-baseline`;
- an existing project whose record has `pending` or `partial` modules, to
  resume.

## When this does not apply

- A documentation-only repository, an estimation artifact, or a reference
  example: not applicable, with the reason stated.
- A read-only intent — audit, review, estimation, investigation,
  architecture-only: never ask and never write.
- Foreign code: `workflows/code-audit.md`.
- Our own deliverable under audit: `workflows/code-audit.md` (Independent Mode).
- The delivery pipeline itself: `workflows/secure-delivery.md`.

## Non-negotiable

- **Opt-in.** Never apply the baseline without the user's explicit answer.
- **Asked once.** Acceptance and refusal write `docs/engineering-baseline.md`;
  "not now" writes nothing and asks again on a later task.
- **Additive.** Preserve existing files, tooling, scripts, and conventions.
  Never restructure the repository.
- **One module at a time**, verified before the next one starts.
- **Verified, not declared.** A module is `adopted` only with evidence;
  otherwise it is `partial` with the gap named.
- **Read-only until consent.** The diagnosis writes nothing.
- **No package** without `workflows/dependency-assessment.md` and human
  authorization.
- **No commit, push, or publication** without explicit approval.

## Modules

`onboarding`, `docs-gate`, `decisions`, `bootstrap`, `devops`, `release`.

Recommended order: `onboarding`, `docs-gate`, `decisions`, `bootstrap`,
`devops`, `release`. Each module is a stopping point.

## Output

The Baseline Gap Report, the modules applied with verification evidence, the
modules remaining, the conflicts, and `docs/engineering-baseline.md` with its
new status. The `onboarding` module is accepted only when the newcomer test
passes with zero questions.
