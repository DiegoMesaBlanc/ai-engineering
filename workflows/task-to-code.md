# Task to Code Workflow

## Goal

Transform an engineering task and available design context into a validated
implementation on a user-provided task branch.

## Phase 1 — Detect

Determine:

- project state
- repository
- task provider
- design provider
- Git provider
- current branch
- working-tree status

## Phase 2 — Understand the Task

Accept the task in any available form: provider reference, URL, local
artifact, or a short ordinary user description. A brief description is a
complete valid task; do not require a rewritten prompt, a Markdown task
specification, or manual implementation instructions.

When a task exists:

- retrieve the task when access is authorized and available; never claim
  to have read inaccessible content
- preserve the original wording and explicit acceptance criteria
- extract business intent
- extract functional requirements
- identify acceptance criteria
- inspect dependencies and comments
- identify design references
- infer requirements and acceptance criteria supported by repository,
  design, and policy evidence, and mark them as inferred
- identify only the material ambiguities that cannot safely be inferred

Maintain the result as a concise working interpretation inside the
session, as defined in `skills/task-context/SKILL.md`. Do not create a
task specification file by default.

Load Task Context only when relevant.

## Phase 3 — Analyze the Design

When design context exists:

- identify the relevant frame, page, component, or region
- retrieve the smallest useful design context
- map design requirements to existing components
- identify missing information

Do not assume Figma.

## Phase 4 — Analyze the Repository

For an existing project:

- inspect repository structure
- detect language and framework
- detect application architecture
- detect monorepo or multi-repository organization
- inspect dependency manifests, lockfiles, and installed direct dependencies
- detect microfrontend composition when present
- identify conventions and reusable abstractions

For a new project:

- determine the minimum viable architecture
- establish repository structure
- determine whether a workspace is justified
- evaluate whether microfrontends provide a concrete benefit
- establish a formatter and linter with reproducible commands, following
  the Formatting and Linting Policy in AGENTS.md

Do not introduce a monorepo or microfrontends without evidence.

## Phase 5 — Intent and Alignment

Identify the engineering intent from the request:
- Architecture or solution design.
- Time and cost estimation.
- Requirements analysis or technical investigation.
- Software implementation.
- Validation, debugging, or code review.

Apply intent-specific behavior:
- Architecture-only or estimation-only requests: do not start implementation or modify application source code unless separately authorized. Do not request a branch name or prepare a Git branch. Produce proportionate analysis (options, trade-offs, assumptions, uncertainty, risks, unresolved questions). When moving to implementation later, reuse this work. An estimation request follows `workflows/estimation.md`, which produces a durable estimation document from the supplied artifact and never implements.
- Other modes (e.g., requirements analysis/technical investigation, validation/debugging/code review that do not require application code changes): do not request or prepare a branch unless the user explicitly needs to make code changes. If read-only, perform the requested analysis without Git branch preparation.
- Software implementation: proceed to alignment and, once agreed, branch preparation will occur before modifying code.

If software implementation:

1. Understand business objective, task/design/repo context.
2. Identify acceptance criteria, constraints, non-goals, material unknowns.
3. Infer routine technical details from codebase, conventions, official docs, policies.
4. Ask concise questions only for important business requirements missing/conflicting/cannot be inferred. Do not ask about choices inferrable from evidence or already available.
5. Present concise summary of agreed behavior and proposed approach.
6. Confirm alignment before implementation.

A preliminary complexity estimate may be recorded here as tentative. It
does not fix the rigor level. The definitive complexity classification is
produced in Phase 8 and the rigor level is derived in Phase 9.

The technical complexity and architecture evaluation is performed once in
Phase 8. Do not repeat the affected-modules, repository-strategy,
frontend-composition, architecture-impact, design-pattern, and risk
checklist here; reuse the Phase 8 result when it has already been obtained.

## Phase 6 — Ask for the Branch (implementation only)

Only when a software implementation task has been aligned and code changes are required, ask for the exact branch name if it was not provided.

Do not generate the name automatically.

Do not request or prepare a branch for architecture-only, estimation-only, read-only investigation, validation/debugging reviews, or other non-implementation requests.

## Phase 7 — Prepare the Branch (implementation only)

Only when a software implementation task has been aligned and code changes are required:

- validate the name
- detect branch collisions
- identify the correct base
- preserve uncommitted work
- create the branch or isolated worktree
- verify the active working location

Do not implement until the requested branch is ready.

## Phase 8 — Complexity and Architecture

Evaluate:

- task complexity
- affected modules
- repository strategy
- frontend composition
- architecture impact
- relevant design patterns
- risks and testing needs

Perform the architecture-first evaluation defined in
`skills/architecture-advisor/SKILL.md` before implementation. It must cover
the real project architecture and conventions, placement and cohesion,
reuse and module conventions, design patterns only when justified, and
existing dependencies, tools, and styling conventions.

Base the diagnosis on implementation evidence and classify it as detected
with evidence, inferred with uncertainty, or no identifiable architecture.
Separate the architectural style, the organization strategy, and the
language/framework conventions, and address reusable types, interfaces, and
contracts separately from their consumers.

Use only the analyses relevant to this task.

When the task changes a boundary, interface, or integration, load
`skills/api-contract-design/SKILL.md` and include the contract in the
proposal. When the task touches the database — schema, migration, query,
index, or transaction — load `skills/database-engineering/SKILL.md` and
include the database plan in the proposal. Use
`skills/implementation-recipes/SKILL.md` for stack-appropriate
implementation guidance, and record a significant, hard-to-reverse decision
following the Decision Records guidance in
`skills/architecture-advisor/SKILL.md` only when it meets the recording
threshold.

This is the single phase that performs the technical complexity and
architecture evaluation. When it has already been completed for the current
task and repository state, reuse the result instead of repeating it. Extend
the analysis only when new evidence, a changed repository state, or a
material unresolved question justifies it.

Mandatory decision gate. Before implementing:

- If the project has a coherent architecture, continue it unless there is
  an important incompatibility.
- If no clear architecture is detected, conventions conflict, or the task
  requires choosing a new architecture, present the diagnosis and a
  recommendation with concrete alternatives and trade-offs, then ask the
  user whether to preserve the existing organization or adopt the
  recommended alternative.
- Wait for the answer before implementing. Do not silently turn a
  recommendation into an approved decision.
- Do not ask about routine decisions that can be inferred from the code.
- If the user specifies an architecture, respect it and explain any
  significant incompatibility before implementing it.

The proposal must include a sufficiently concrete target structure, the
location of relevant files, the contracts and types, the direction of
dependencies, and the pending decisions; stating "architecture evaluated"
is not sufficient.

Require human approval for significant architectural changes per the Human Decision Gates in AGENTS.md. Do not treat routine component choices as significant architectural decisions. Consult the user when a styling-system decision significantly affects architecture, dependencies, or the visual experience.

## Phase 9 — Plan

Determine the proportional rigor level from the Phase 8 complexity
classification and produce the plan by following
`workflows/task-planning.md`. Do not reclassify complexity here.

Before the plan is finalized, complete the dependency assessment described
in `workflows/dependency-assessment.md` and record its outcome.

Provide a concise implementation plan including:

- files and modules affected
- existing abstractions to reuse
- the dependency outcome (`NO_NEW_DEPENDENCY`, `REUSE_EXISTING`,
  `NEW_DEPENDENCY_PROPOSED`, or `BLOCKED`) with its evidence
- proposed new components only when justified
- formatting and linting setup when the project lacks it
- the Secure Delivery Workflow selected when the task involves CI/CD,
  security, or delivery controls, including authoring a delivery pipeline
  with its environments, gates, and deployment stages
- validation
- risks

The depth of the plan follows its rigor level: R1 Brief for trivial and low
tasks, R2 Moderate for medium tasks, R3 Complex for high tasks, and R4
High-risk for critical tasks. A High task requires approval only when a
Human Decision Gate applies; a Critical task activates reinforced controls
and requires approval before high-impact operations. The plan is session
state by default; a durable planning artifact is created only when
`workflows/task-planning.md` justifies it.

A new dependency must be authorized through the approval gate in
`workflows/dependency-assessment.md` before it is added or installed. Do not
treat approval to implement the task as authorization to introduce
libraries.

## Phase 10 — Implement

Implement on the requested branch.

Preserve existing conventions.

Follow `skills/implementation-recipes/SKILL.md` for the detected stack and
apply its non-negotiable quality baseline: quality is never proportional to
task size. A trivial change is implemented carefully, not quickly at the
expense of conventions, error handling, typing, security, or tests.

Proceed autonomously for routine work after alignment. Do not pause for user approval for each file edit, test, formatting change, local refactoring, or ordinary corrective adjustment within the approved scope. Do not weaken quality, security, testing, or validation standards.

Provide concise status updates at meaningful milestones (alignment complete, milestone completed, important new finding or decision required, validation results, final diff/summary). Each update must explain what changed, the evidence available, and the next step. An update is not a request for approval unless a decision gate applies.

If new evidence changes a material requirement or approved design, pause and explain the reason before continuing beyond the approved scope. Otherwise, proceed autonomously.

Use checkpoints according to `workflows/session-resilience.md`. Do not require restarting the terminal.

Do not expand the scope silently.

## Phase 11 — Validate

Load `skills/testing-strategy/SKILL.md`.

1. Inspect the current changes and project tooling.
2. Determine validation risk.
3. Select the smallest sufficient validation plan.
4. Run relevant checks using existing project commands.
5. Broaden validation when risk, failure evidence, or project policy
   requires it.
6. Record commands, outcomes, and limitations.
7. Report failures and checks that were not executed.
8. Never claim that validation passed without evidence.

When the change set touches CI/CD, security, or delivery configuration,
follow `workflows/secure-delivery.md` for scoped delivery checks and
respect its read-only and authorization boundaries. When a task authors a
delivery pipeline, follow the same workflow's authoring mode: build once
and promote across environments, run the required validation gates before
every deployment, keep environment differences out of the artifact, and
require explicit authorization for production and remote operations.

Formatting and linting are mandatory for the application code of the
software project, for every detected language in scope. If the project
lacks a configured formatter or linter, configure them as part of the
task, as described in the Formatting and Linting Policy in AGENTS.md and
in `workflows/project-validation.md`. Report missing tooling in scope as
`SETUP_REQUIRED`, never as `NOT_APPLICABLE`. A documentation-only
knowledge repository is out of scope and must not be converted into a
software project to comply.

Prefer focused tests for changed behavior before running broad suites.

Before an expensive validation, and at logical phase boundaries, record a
checkpoint as described in `workflows/session-resilience.md` so the task can
continue after `/compact` or `/new`. Keep compaction enabled and limit
context growth with focused reads and bounded output.

Use existing test frameworks, type checkers, linters, formatters, build
commands, and configured security tools.

For monorepos, target the affected project or packages when reliable
workspace tooling is available.

Never install dependencies, add frameworks, change configuration, or
execute potentially destructive validation commands without appropriate
authorization, except for the mandatory formatting and linting setup.

Preserve unrelated and uncommitted user changes.

A passing build alone does not establish completion.

## Phase 12 — Review

Load `skills/engineering-review/SKILL.md`.

Review the resulting implementation for:

- correctness
- requirements
- architecture
- security
- performance
- maintainability
- testing
- scope

Perform the post-implementation architectural validation defined in
`skills/architecture-advisor/SKILL.md`: compare the real architecture with
the proposal and the approved decision, in addition to the functional
tests. Each important conclusion must cite concrete evidence (files,
modules, imports, configuration, or executed commands), and any property
that cannot be verified automatically must be reported as such. Report
deviations and their impact, and do not claim the agent's behavior is proven
by verifying instructions alone.

## Phase 13 — Git Delivery

Inspect:

- current branch
- status
- diff
- changed files
- unrelated changes

Propose a commit message.

Create a commit only under the established human-approval policy.

Do not push automatically.

## Phase 14 — PR Preparation

When requested, generate:

- copy-ready PR title
- copy-ready PR description
- testing summary
- risks and limitations
- reviewer notes

The user creates the PR/MR manually.

Never create or publish the PR/MR.

## Phase 15 — Engineering Insights

After implementation and validation, report meaningful engineering
observations related to the task.

Include only observations with practical value, such as:

- relevant improvement opportunities
- technical debt discovered during the task
- reusable abstractions worth considering
- architectural risks
- testing gaps
- security or performance concerns

Separate these observations from the task deliverable.

Do not implement unrelated improvements automatically.

Explain the expected benefit and relevant trade-offs.

Keep the report concise and proportional to the task complexity.
