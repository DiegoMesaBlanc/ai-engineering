# Dependency Assessment Workflow

## Purpose

Ensure every software implementation task makes an evidence-based dependency
decision before implementation.

Determine whether the task can be completed with the existing project
dependencies, abstractions, and native APIs, whether a compatible dependency
is already installed, or whether a new library is genuinely justified and
authorized.

This workflow is the single canonical dependency procedure. It is referenced
by `workflows/task-to-code.md`, the task commands, the stack-detection Skill,
and AGENTS.md, which must not duplicate its decision policy.

## Scope

This workflow applies to the target software project.

It does not apply to the documentation-only ai-engineering repository.
Never create a package.json, install packages, or add dependency manifests to
ai-engineering to satisfy this workflow.

## Boundaries

- The assessment is part of task analysis and planning. It is not a license
  to install anything.
- A recommendation is not an installation.
- Inspecting manifests, lockfiles, and installed packages is read-only.
- Adding or installing a new runtime or development dependency requires
  explicit authorization, except for the mandatory formatter and linter
  setup already authorized in AGENTS.md.
- Never install paid services or dependencies, initiate billable usage, or
  add paid dependencies without explicit authorization.
- Never expose credentials, registry tokens, or private registry URLs.

## Phase 1 — Inspect the Task and Project

Before proposing anything, inspect:

- the task description and acceptance criteria
- the existing language, framework, runtime, and versions
- dependency manifests and lockfiles
- installed direct dependencies and relevant transitive dependencies
- the package manager and workspace or monorepo boundaries
- existing project abstractions, utilities, native APIs, and reusable
  components
- project conventions and applicable organizational restrictions

Do not install anything merely because a library is mentioned in the task
description.

Do not assume a dependency is missing until the actual project configuration
has been inspected.

## Phase 2 — Choose One Explicit Dependency Outcome

Record exactly one outcome for the task plan:

- `NO_NEW_DEPENDENCY`: the existing stack, native APIs, and project
  abstractions are sufficient.
- `REUSE_EXISTING`: an installed, compatible dependency already covers the
  requirement.
- `NEW_DEPENDENCY_PROPOSED`: a new dependency has a demonstrated benefit and
  needs user authorization.
- `BLOCKED`: a material compatibility, access, licensing, or technical
  question prevents a reliable decision.

Explain the evidence behind the selected outcome.

Do not ask the user to approve a dependency when none is needed. State
clearly that no new dependency is recommended and proceed with the normal
task flow.

## Phase 3 — Compare Options When a New Dependency Is Justified

Present a concise comparison of feasible approaches:

1. Existing implementation or native API, where viable.
2. Reuse or extension of an installed library, where viable.
3. One or two new libraries that genuinely fit the requirement.

Do not force three options when fewer are technically viable.

For each new library, evaluate when relevant:

- compatibility with the actual framework and runtime versions
- maintenance status and stable release information
- documentation quality and ecosystem fit
- security advisories and dependency health
- license suitability
- direct and transitive dependency impact
- bundle or runtime cost, when relevant
- developer experience and testability
- migration and long-term maintenance costs
- strengths, weaknesses, and reasons to prefer it over the alternatives

Use authoritative current sources when compatibility or maintenance claims
materially affect the choice. Do not invent current versions, download
counts, security status, or maintenance claims.

Recommend the smallest reasonable dependency footprint. Avoid adding a
library when existing code or native functionality solves the task
adequately.

## Phase 4 — Approval Gate

Before adding or installing a new runtime or development dependency,
explain:

- why it is needed
- which option is recommended and why
- what files and lockfiles will change
- any material security, license, compatibility, maintenance, or
  bundle-size concerns
- whether the dependency is runtime or development-only

Ask the user for approval when the required authorization has not already
been explicitly provided.

Treat approval of a specific option as authorization only for that option
and scope. If a material change in package, version, license, cost, or
purpose becomes necessary, request renewed approval.

Do not treat general approval to implement a task as blanket authorization
to introduce arbitrary libraries.

An explicit user instruction naming a dependency can count as approval for
that dependency, but compatibility and security checks still apply.

The existing policy exception for mandatory formatter and linter setup in
AGENTS.md is preserved. Report those required tooling additions clearly and
do not extend that exception to unrelated runtime libraries or frameworks.

Never install paid services, initiate billable usage, or add paid
dependencies without explicit authorization.

## Phase 5 — After Approval

Use the project's existing package manager and workspace conventions.

- Check for an already installed compatible version before adding a package.
- Avoid duplicate packages serving the same purpose.
- Select a compatible version using current evidence.
- Update the appropriate manifest and lockfile together.
- Do not upgrade unrelated dependencies.
- Do not install globally when the project requires a local dependency, or
  the reverse.
- Do not bypass peer-dependency or security errors using force flags without
  explicit investigation and authorization.
- Record the reason for the dependency and its intended usage.
- Add or update tests where justified.
- Validate install reproducibility, formatting, linting, tests, type
  checking, build, and relevant security checks according to the project's
  stack and risk.

If installation fails, do not silently substitute a different dependency.
Report the actual blocker and offer alternatives.

## Phase 6 — No-Approval Path

If the user declines a proposed dependency:

- do not install it
- evaluate whether the task can be completed with the existing stack or a
  native API
- explain any resulting limitations
- if the acceptance criteria cannot reasonably be met without the dependency,
  describe the remaining decision instead of silently changing the scope

## Reporting

Record in the task plan:

- the selected outcome
- the supporting evidence
- the manifests, lockfiles, and abstractions inspected
- the options considered, when a new dependency is proposed
- the approval status: `NOT_REQUIRED`, `REQUESTED`, `GRANTED`, `DECLINED`, or
  `BLOCKED`
- for an approved dependency: name, selected version, runtime or
  development-only scope, purpose, and the files changed

Reuse the existing reporting statuses. Do not claim a dependency decision
was validated without evidence, and do not report an install as successful
without an observed result.

## Safety

- Read-only inspection by default.
- Never install anything without the required authorization.
- Never install paid services or dependencies without explicit
  authorization.
- Never expose credentials, registry tokens, or private registry URLs.
- Never silently substitute a dependency after an installation failure.
- Never change the task scope silently when a dependency is declined.
- Never convert the documentation-only ai-engineering repository into a
  software package to satisfy this workflow.
