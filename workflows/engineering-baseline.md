# Engineering Baseline Workflow

## Purpose

Give a software project an opt-in engineering baseline that removes one
specific failure: onboarding that only works when an existing team member
explains the project from memory, because the repository does not contain
what a new person needs to start.

The baseline also reduces the configuration a project repeats at the start:
the skeleton, the tooling, the environment, the decisions, the documentation
gate, and the delivery pipeline.

The baseline is **opt-in**. Some projects want it and some do not. The system
asks once, records the answer in the project, and respects it. It never
restructures an existing repository, never overwrites a working convention,
and never applies a module the user did not select.

This workflow is **not**:

- `workflows/project-initialization.md` — that creates a new project. This
  workflow can be offered during that creation, and can also run on an
  existing project.
- `workflows/project-validation.md` — that executes the project's gates.
  This workflow installs the documentation gate and verifies it once.
- `workflows/secure-delivery.md` — that authors and assesses the delivery
  pipeline. The `devops` module extends it and never duplicates it.
- `workflows/external-audit.md` — that audits a deliverable from a fresh
  session. This workflow changes the project and is not an audit.
- `workflows/code-audit.md` — that audits foreign code read-only.
- `workflows/system-audit.md` — that audits the AI Engineering System
  itself.

## Adoption Contract

| # | Rule | Consequence |
|---|---|---|
| B-1 | **Opt-in** | The baseline is never applied without an explicit answer from the user. A general request to implement a task is not that answer. |
| B-2 | **Asked once** | The system asks only when the record is absent. Acceptance and refusal write the record; "not now" writes nothing and asks again on a later task. |
| B-3 | **Additive and incremental** | Existing files, tooling, scripts, conventions, and pipelines are preserved. A module adds what is missing and adapts what differs. It never replaces what works and never restructures the repository. |
| B-4 | **One module at a time** | Each module is applied, verified, and recorded before the next one starts. Every module is a stopping point. |
| B-5 | **Existing convention wins** | When the project already has a way to do something, that way is preserved unless it is broken and the user approves the change. |
| B-6 | **Conflict is reported, not resolved silently** | A module that conflicts with an existing convention stops and asks. |
| B-7 | **Verified, not declared** | A module is `adopted` only when its verification passed. Artifacts without evidence are `partial`, with the gap named. |
| B-8 | **No package without the gate** | Any new formatter, linter, test runner, or tooling dependency goes through `workflows/dependency-assessment.md` and human authorization. |
| B-9 | **No remote write, no commit** | The workflow writes only inside the target project and never commits, pushes, publishes, or opens a PR without explicit approval. |

## The Single Question

This is the canonical wording. Other workflows and commands reference it and
do not restate it.

> This project has no engineering baseline. A baseline removes the need for
> an existing team member to explain the project from memory when someone new
> joins, and reduces the configuration a new project repeats at the start. It
> covers onboarding, a documentation gate, decisions (ADR), bootstrap, DevOps,
> and release. Adopt it?
>
> 1. Yes, all modules
> 2. Yes, choose modules
> 3. Not now (ask again on a later task)
> 4. No, never for this project

### When the question is asked

Ask **only** when every condition holds:

1. The target project is a software project with application source code.
   A documentation-only repository, an estimation-only artifact, or a
   reference example is not asked.
2. The task will create or change source code. A read-only intent — audit,
   review, estimation, investigation, architecture-only — never asks.
3. `docs/engineering-baseline.md` does not exist in the target project.
4. The question has not already been answered in the current session.

If any condition fails, do not ask.

### What each answer writes

| Answer | Record | Later behavior |
|---|---|---|
| 1. All modules | `status: partial`, every module `pending` until verified; `adopted` once all are verified | The adoption runs; unfinished modules are resumed with `/adopt-baseline` |
| 2. Choose modules | `status: partial`, selected modules `pending`, the rest `declined` | The adoption runs for the selected set only |
| 3. Not now | **No file** | The question is asked again on a later task |
| 4. Never | `status: declined`, no module entries | The system never asks again; only an explicit `/adopt-baseline` reopens it |

An answer given during `/create-project` is recorded in the startup decision
set and materialized in `docs/engineering-baseline.md` as soon as the
repository exists.

When the answer is **2. Yes, choose modules**, present the six modules with
their one-line purpose and ask which to adopt. Do not preselect, and do not
treat silence as acceptance. The selected set becomes the ceiling for this
adoption.

## The Record

The record is `docs/engineering-baseline.md` in the **target project**. It is
durable, travels with the repository, and is the only answer to "has this
project adopted the baseline?".

```markdown
---
status: partial
version: 1
date: 2026-10-24
modules:
  onboarding: adopted
  decisions: adopted
  bootstrap: pending
  devops: declined
  docs-gate: adopted
  release: pending
---

# Engineering Baseline

## Current state

## Gap report

## Module log

## Conflicts

## Verification evidence

## Out of scope
```

### Status values

| Value | Meaning |
|---|---|
| `not-assessed` | The record exists but no module has been assessed yet. It is transient and is updated in the same run. |
| `adopted` | Every selected module is verified. |
| `partial` | Some selected modules are verified; at least one is `pending` or `partial`. |
| `declined` | The user answered "never". The system does not ask again. |

### Module values

`adopted`, `partial`, `pending`, `declined`, `n/a`.

A module is `adopted` only with verification evidence. `partial` names the
missing evidence. `n/a` names the reason.

### Rules

- The frontmatter is written by this workflow, not guessed.
- `version` is the **baseline definition version**, not the record revision.
  It changes when the workflow's module set or adoption contract changes in a
  way that requires re-assessment. `date` records the last update.
- The body is human-readable and holds the gap report, the module log with
  dates and evidence, the conflicts, and the out-of-scope declaration.
- Never store credentials, tokens, or secret values in the record.
- A human may edit the file; the next run re-validates it instead of
  trusting it.

## The Modules

| Module | What it produces | Closes |
|---|---|---|
| `onboarding` | `README` with a working quickstart, `CONTRIBUTING.md`, `docs/onboarding.md`, `docs/architecture.md` with the module map, `CODEOWNERS` when more than one owner exists | Someone new cannot start without an explanation from memory |
| `docs-gate` | An executable `scripts/check-docs`, wired into CI and `/validate`, that fails on missing, stale, or contradictory documentation | Documentation that rots after the first month |
| `decisions` | `docs/adr/` with a template, an index, states, and supersession; seed records for the decisions already made | Decisions that only exist in someone's head |
| `bootstrap` | One-command setup and run (`make setup`, `make dev`, `make test`, `make check`, or the project's existing runner), `compose.yaml` for local dependencies when needed, `.env.example` synchronized with the configuration schema, an optional devcontainer | Manual configuration repeated on every machine |
| `devops` | Multi-stage `Dockerfile`, CI with the same gates as local, health checks, structured logging with a correlation id, graceful shutdown, dependency and secret scanning | A different, weaker pipeline per project, and configuration remembered instead of generated |
| `release` | Semantic versioning policy, `CHANGELOG.md`, release-notes template, deprecation policy with a migration path, a tag and release procedure | Not knowing what is in production or what changed |

### Recommended order

For an existing project, adopt in this order:

1. `onboarding`
2. `docs-gate`
3. `decisions`
4. `bootstrap`
5. `devops`
6. `release`

`onboarding` comes first because it attacks the main failure directly.
`docs-gate` comes second because it keeps onboarding from rotting. `decisions`
comes before `bootstrap` and `devops` because those steps consume the
decisions. `release` comes last because it depends on the pipeline and on the
versioning policy.

When `onboarding` and `bootstrap` are both selected, the command surface that
onboarding documents is the one bootstrap provides: apply the bootstrap
commands first inside that pair, or write onboarding after them, so the
documentation never references a command that does not exist.

### New-project scheduling

A new project interleaves the modules with
`references/project-creation-order.md`. The recommended order above is a
priority order, not a schedule: a module is applied at the earliest step
where its **verification is possible**, never before.

| Module | Earliest creation step | Why |
|---|---|---|
| `decisions` | after `A1` and `A2` | The style, topology, and target versions are already decided; the seed ADRs can be written |
| `bootstrap` | after `A8` | It may be scaffolded after `A3`, but it is not `adopted` until a clean copy reaches a running application and a green suite |
| `onboarding` | after `A8` | The newcomer test needs a running vertical slice and a green suite |
| `docs-gate` | after `onboarding`, or after `A8` when onboarding is not selected | It checks the documents the selected modules produced |
| `devops` | after `A9` | It extends the pipeline that `A9` creates |
| `release` | after `A9` and `A10` | It needs the pipeline and the documentation |

Applying a module earlier than its verification allows produces artifacts
without evidence, which this workflow forbids.

### Module: onboarding

Produce:

- `README.md` with the stack, the architecture style, the prerequisites with
  versions, the commands, the local setup, and where to go next.
- `docs/onboarding.md` with a numbered path from a clean machine to a running
  application and a green suite. Every command is written out; every
  environment variable is named and linked to `.env.example`; no step says
  "ask someone".
- `CONTRIBUTING.md` with the branch policy, the commit policy, the review
  policy, and the quality gates.
- `docs/architecture.md` with the folder map, the module boundaries, and the
  ownership.
- `CODEOWNERS` when the project has more than one owner.

Verify with the **newcomer test** defined below.

### Module: docs-gate

Produce a real executable, not prose. `scripts/check-docs` fails when:

- a required document is missing (`README.md`, `CONTRIBUTING.md`,
  `docs/onboarding.md`, `docs/architecture.md`, the ADR index, the
  changelog when release is adopted);
- `.env.example` and the configuration schema disagree on a variable name,
  a required flag, or a default, when the project has a schema;
- an ADR lacks its required sections or an index entry points to a missing
  file;
- onboarding references a script or command that does not exist in the
  project;
- the README quickstart references a script that does not exist.

Wire it into the project's CI and into `/validate`. The script must exit
non-zero on failure and print the finding with `file:line` or the missing
path.

Verify by introducing a deliberate break, confirming the gate fails, then
restoring it and confirming it passes.

The required set is derived from the record: a document or a check is
required only when its module is `adopted` or `partial`. A module that is
`pending` is not applied yet, and a module that is `declined` or `n/a` was
not selected, so neither is checked. A project that adopts `docs-gate`
without `onboarding` or `release` is not failed for documents it never
selected, and a module still waiting for its creation step is not failed
before it starts.

### Module: decisions

Produce `docs/adr/` with:

- `0000-template.md` containing context, decision, alternatives,
  consequences, status, and supersession;
- `README.md` as the index, with number, title, status, and date;
- seed records for the decisions already made and already recorded elsewhere:
  architecture style, repository topology, target versions, persistence
  choice, identity and access, and any decision the source analysis settled.

Never invent rationale. A seed record cites the source of the decision.

### Module: bootstrap

Produce a single entry point and make it the documented path:

- `make setup` — verify tool versions, install dependencies, create the local
  environment from `.env.example`, start local dependencies, run migrations,
  and run the suite;
- `make dev`, `make test`, `make check`, and `make build` — the commands a
  contributor and CI both use;
- `compose.yaml` for local dependencies when the stack needs a database,
  a broker, or a cache;
- `.env.example` with every variable, a safe default where one exists, and no
  secret;
- an optional devcontainer when the team uses one.

Use the project's existing task runner when it has one. Otherwise, `make` is
the portable default. Never shadow existing scripts with a second entry
point.

Verify from a clean copy: `make setup` reaches a running application and a
green suite with no manual step.

### Module: devops

Extend `workflows/secure-delivery.md`; do not duplicate its phases.

Produce:

- a multi-stage `Dockerfile` with a non-root user and no development
  dependencies in the runtime image, when the project deploys as a container;
- a CI pipeline that runs the same gates as the local check: format check,
  lint, typecheck, tests, build, structural compliance, and `docs-check`
  when the `docs-gate` module is adopted. Use `make check` when `bootstrap`
  is adopted, and the project's existing scripts otherwise;
- a health endpoint or health check;
- structured logging with a correlation id;
- graceful shutdown;
- dependency and secret scanning using the project's approved tooling.

Verify by running the pipeline on a branch and confirming every gate, and by
building and starting the image and observing the health check.

### Module: release

Produce:

- the semantic-versioning policy and what counts as major, minor, and patch;
- `CHANGELOG.md` in Keep a Changelog form;
- a release-notes template;
- a deprecation policy with a migration path and a removal window;
- the tag and release procedure, with production publication behind explicit
  approval.

Verify with a dry run: from the change set, produce the changelog entry and
the release notes without publishing anything.

## Phase 0 — Consent and Mode

Determine the mode from the record and the invocation:

| Mode | When | Writes |
|---|---|---|
| `assess` | No record, `--dry-run`, `--status`, or a record whose selected modules are all verified and no module was requested | Nothing |
| `adopt` | The user accepted the question, or invoked `/adopt-baseline` with `--modules` to select or add modules | The selected modules and the record |
| `resume` | The record has `pending` or `partial` modules and the invocation does not change the set | The incomplete modules |
| `reassess` | `--reassess`, or the record is suspected stale | The diagnosis and the record, after the user confirms reopening a `declined` record |
| `decline` | The user answered "never" | Only `status: declined`, and only when no module is `adopted`; otherwise stop and ask |

Record the mode, the selected modules, and the user's answer before writing
anything.

A `declined` record is reopened only by an explicit `/adopt-baseline`,
`--reassess`, or `--modules`, never by a routine task. A `decline` on a
record that already has an `adopted` module stops and asks, because it would
erase a verified state.

## Phase 1 — Scope and Diagnosis

Read-only.

1. Resolve the **artifact root** and the **Git root** separately. The target
   project is the only writable scope.
2. Detect the stack, the package manager, the existing scripts, the CI, the
   documentation layout, the ADR convention, the environment handling, and
   the container setup.
3. Inventory what already exists per module.
4. Produce the **Baseline Gap Report**:

| Module | Current state | Evidence | Class | Risk |
|---|---|---|---|---|
| `onboarding` | `README` exists, no quickstart, no `CONTRIBUTING` | `README.md:1`, absent paths | `Adapt` | Medium |

Classes:

- **Adopt** — absent and additive.
- **Adapt** — present but incomplete or inconsistent; align without
  replacing what works.
- **Conflict** — collides with a working convention; stop and ask.
- **Skip** — not selected, or not applicable with a stated reason.

Do not write in this phase.

## Phase 2 — Plan

Produce a plan with:

- the modules in the recommended order for an existing project, or in the
  new-project schedule when the project is being created, restricted to the
  selected set;
- the files each module creates and the files it touches;
- the conflicts and the decision each one needs;
- the verification for each module and its pass criterion;
- the installation package, if any tooling is required, presented through
  `workflows/dependency-assessment.md`;
- the stopping points.

Do not start implementation before the plan is accepted for the module in
progress.

## Phase 3 — Apply One Module at a Time

For each module:

1. Show the files it will create and the exact diff of every existing file it
   will touch.
2. Apply additively. Never overwrite a working file without the approved
   diff.
3. Never restructure the repository, rename existing folders, or replace the
   existing tooling.
4. Never install a package outside the approved installation package.
5. Never commit, push, or publish.
6. Stop at the end of the module, update the record, and report what the next
   module would add.

A module that cannot be applied without a conflict stops with the conflict
described and the options presented.

## Phase 4 — Module Verification

| Module | Verification | Pass criterion |
|---|---|---|
| `onboarding` | Newcomer test on a clean copy | Zero questions and a green suite |
| `docs-gate` | Deliberate break, then restore | The gate fails on the break and passes on the restore |
| `decisions` | Index and template check | Every recorded decision has a complete ADR; the index resolves |
| `bootstrap` | Clean copy, one command | The application runs and the suite is green with no manual step |
| `devops` | Pipeline on a branch, image build and start | Every gate runs; the health check responds |
| `release` | Dry run | A changelog entry and release notes are produced without publishing |

### The newcomer test

This is the acceptance criterion that converts "a contributor can run the
project" from a wish into evidence.

1. Copy the repository to a clean directory with no local state, no
   `node_modules`, no `.env`, and no build output. A fresh `git clone` is
   preferred.
2. Start a **fresh session** with no prior context.
3. Follow only `docs/onboarding.md`, in order.
4. Reach: prerequisites installed, dependencies installed, configuration
   created from `.env.example`, application running, suite green, and one
   trivial change prepared with a PR.
5. Record every step where the documentation was insufficient and a question
   had to be asked.

Pass requires **zero questions**. A question means the onboarding is not
adopted; it is `partial` with the missing documentation named.

Evidence class:

- **SIMULATED** when the test is executed by an agent in a fresh session on a
  clean copy. This is the minimum required.
- **REAL** when an actual new person performs it.

A static reading of the document is not a newcomer test.

## Phase 5 — Update the Record

After each module:

- set the module value to `adopted`, `partial`, `pending`, `declined`, or
  `n/a`;
- append a module-log entry with the date, the files, the verification
  command, and the result;
- record every conflict and its resolution;
- set `status` from the module values: `adopted` when every selected module
  is verified, otherwise `partial`;
- set `date` to the current date;
- bump `version` only when the baseline definition changed, never for a
  module state change.

Never mark a module `adopted` because its files exist. The verification
decides.

## Phase 6 — Report

Report, in this order:

1. The answer to the single question and the mode.
2. The Baseline Gap Report.
3. The modules applied, with their verification evidence.
4. The modules remaining and what the next one would add.
5. The conflicts and their resolution.
6. The record path and its new status.
7. The out-of-scope declaration.

Stopping after any module is a normal outcome, not a defect.

## Relationship to Other Workflows

| Workflow | Relationship |
|---|---|
| `workflows/project-initialization.md` | Offers the baseline in Phase 3 and instantiates the selected modules in Phase 4, after the skeleton and the tooling are verified |
| `workflows/task-to-code.md` | Phase 1 asks the single question for an existing project, under the stated conditions; an accepted adoption is a separate change set |
| `workflows/dependency-assessment.md` | Governs every package the baseline needs |
| `workflows/secure-delivery.md` | The `devops` module extends it and does not duplicate it |
| `workflows/project-validation.md` | Runs `scripts/check-docs` once the `docs-gate` module exists |
| `workflows/external-audit.md` | Re-executes `scripts/check-docs` and judges the baseline's claims |
| `skills/architecture-advisor/SKILL.md` | Decides when a decision is significant; the `decisions` module provides the storage and the template |
| `references/project-creation-order.md` | The steps the baseline makes executable: A10, B21, D6, and E14 |

## Out of Scope

The baseline does not cover, and says so instead of implying coverage:

- incident response, on-call, postmortems, and disaster-recovery drills;
- infrastructure provisioning and infrastructure-as-code authoring;
- capacity planning and cost engineering;
- privacy, data retention, consent, and regulatory compliance;
- product analytics, feature flags, and go-to-market material.

These remain separate gaps. A project that needs them runs a separate flow;
the baseline does not pretend to close them.

## Anti-Patterns

- Generating documentation that describes a project nobody verified.
- Writing ADRs in bulk with invented rationale.
- Adding a `Makefile` that shadows the project's real scripts.
- Adding CI that duplicates or weakens the existing pipeline.
- Marking a module `adopted` because the files were created.
- Asking again after the user answered "never".
- Applying the baseline inside a feature task and mixing both change sets.
- Treating this workflow as an audit: it changes the project and is not
  independent.
