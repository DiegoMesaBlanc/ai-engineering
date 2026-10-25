---
description: Adopt an opt-in engineering baseline that makes onboarding work from the repository itself instead of depending on an existing team member explaining the project from memory, and reduces the configuration a project repeats at the start, additively and incrementally, in a new or existing project.
agent: orchestrator
---

# /adopt-baseline

Run the Engineering Baseline Workflow:

`workflows/engineering-baseline.md`

## What it is for

This command closes two gaps, and says so explicitly:

- onboarding that only works when an existing team member explains the
  project from memory;
- the configuration a project repeats at the start: skeleton, tooling,
  environment, decisions, documentation gate, and delivery pipeline.

It is **opt-in**. It never runs on its own and never applies a module the
user did not select.

## Input

Optional arguments in `$ARGUMENTS`:

- `--modules onboarding,docs-gate` — adopt only the named modules.
- `--dry-run` — run the diagnosis and the plan, write nothing.
- `--reassess` — re-diagnose a project that already has a record.
- `--decline` — record `status: declined` and stop asking.
- `--status` — report the record and the gaps without writing.

Without arguments, resume from the first incomplete module in
`docs/engineering-baseline.md`; when the record is absent, ask the single
question and proceed from the answer; when the record is `adopted`, report
its state and change nothing.

## Mandatory behavior

1. Ask the canonical question **once** when the record is absent, and never
   when it is `declined`.
2. Reject an unknown module name and list the valid ones. `--decline` stops
   and asks when a module is already `adopted`, because it would erase a
   verified state.
3. Diagnose read-only before writing anything: the Baseline Gap Report with
   the class `Adopt`, `Adapt`, `Conflict`, or `Skip` per module.
4. Preserve every existing convention, tool, script, and pipeline. Apply
   additively; never restructure the repository.
5. Apply one module at a time, verify it, update the record, and stop.
6. Send every new package through `workflows/dependency-assessment.md` and
   human authorization.
7. Verify each module with its own check. The `onboarding` module is
   `adopted` only after the newcomer test passes with zero questions.
8. Keep the record truthful: `adopted` requires evidence; otherwise
   `partial` with the gap named.
9. Never commit, push, publish, or open a pull request. Creating a commit
   requires explicit approval in the current conversation.
10. Write only inside the target project. An evidence-only path is read-only.

## Modules

`onboarding`, `docs-gate`, `decisions`, `bootstrap`, `devops`, `release`.

The definitions, the recommended order, and the verification of each module
are in the workflow. This command does not restate them.

## Output

- The answer to the single question and the mode.
- The Baseline Gap Report.
- The modules applied with their verification evidence.
- The modules remaining and the next module's content.
- The conflicts and their resolution.
- `docs/engineering-baseline.md` with its new status.
- The out-of-scope declaration.

## Relationship to other commands

- `/create-project` offers the baseline during the startup decision set.
- `/task` asks the single question for an existing project, only when the
  task changes source code and the record is absent.
- `/validate` and `/external-audit` execute `scripts/check-docs` once the
  `docs-gate` module exists.
- `/delivery-check` and `workflows/secure-delivery.md` govern the pipeline
  that the `devops` module extends.

This command is not an audit and does not review foreign code. Those are
`/code-audit`, `/external-audit`, and `/review-pr`.
