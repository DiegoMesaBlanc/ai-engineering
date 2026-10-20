# Task to Code Workflow

## Goal

Transform an engineering task and available design context into a validated
implementation on a user-provided task branch.

## Phase 1 — Detect

Determine:

- project state
- repository
- target project root and writable scope
- evidence-only paths supplied as reference, comparison, or context
- task provider
- design provider
- Git provider
- current branch
- working-tree status
- task type and provisional rigor
- provisional Context Manifest: required Skills and references, likely
  files, exploration budget, and stop condition

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
- distinguish the target project from evidence-only paths and preserve
  that boundary throughout the task
- infer requirements and acceptance criteria supported by repository,
  design, and policy evidence, and mark them as inferred
- identify only the material ambiguities that cannot safely be inferred
- refine the Context Manifest and do not add unrelated paths, Skills, or
  documentation to it

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

Treat design artifacts as evidence. Never edit the source design artifact or
an external reference project.

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

**Official documentation consultation (mandatory after stack detection):**

Once the stack is detected (language, framework, runtime, versions), **consult `references/official-documentation.md`** for the detected technologies to retrieve authoritative guidance on:

- Framework-specific architecture conventions (e.g., Next.js App Router patterns, React Server Components)
- Official style guides and conventions (e.g., React Rules, Angular Style Guide)
- Version-specific APIs, deprecations, and migration paths
- Security best practices for the detected stack
- Testing strategies recommended by the framework

**When to consult:** After Phase 4 completes stack detection, before Phase 8 (architecture evaluation).
**Scope:** Only the sections relevant to the detected stack and the current task. Do not load entire documentation sites.

## Phase 5 — Intent and Alignment

Identify the engineering intent from the request:
- Architecture or solution design.
- Time and cost estimation.
- Requirements analysis or technical investigation.
- Software implementation.
- Validation, debugging, or code review.

Apply intent-specific behavior:
- Architecture-only or estimation-only requests: do not start implementation or modify application source code unless separately authorized. Do not request a branch name or prepare a Git branch. Produce proportionate analysis (options, trade-offs, assumptions, uncertainty, risks, unresolved questions). When moving to implementation later, reuse this work. An estimation request follows `workflows/estimation.md`, which produces a durable estimation document for a complete project from start to finish, from the supplied project artifact, and never implements; ticket-, story-, or task-level estimates are out of scope.
- Other modes (e.g., requirements analysis/technical investigation, validation/debugging/code review that do not require application code changes): do not request or prepare a branch unless the user explicitly needs to make code changes. If read-only, perform the requested analysis without Git branch preparation.
- Software implementation: proceed to alignment and, once agreed, branch preparation will occur before modifying code.

If software implementation:

1. Understand business objective, task/design/repo context.
2. Identify acceptance criteria, constraints, non-goals, material unknowns.
3. Infer routine technical details from codebase, conventions, official docs, policies.
4. Ask concise questions only for important business requirements missing/conflicting/cannot be inferred. Do not ask about choices inferrable from evidence or already available.
5. Present concise summary of agreed behavior and proposed approach.
6. Confirm alignment before implementation.

For projects or features with user-facing text, decide localization early:
whether more than one language is required, which locales, whether RTL is
needed, and whether locale-aware date, number, or currency formatting applies.
Load `skills/internationalization/SKILL.md` when it does, and include it in the
agreed behavior. Do not impose internationalization on work without
user-facing text.

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

**Official documentation consultation (mandatory for architecture evaluation):**

Before proposing architecture, **consult `references/official-documentation.md`** for the detected stack to retrieve:

- Official architecture recommendations for the framework (e.g., Next.js App Router architecture, React component patterns)
- Official guidance on architectural styles (Clean Architecture, Hexagonal, Feature-Sliced, Modular Monolith)
- Framework-specific conventions for folder structure, module boundaries, and dependency direction
- Version-specific constraints and capabilities

**When to consult:** At the start of Phase 8, before evaluating architecture alternatives.
**Scope:** Architecture-relevant sections only. Cross-reference with `skills/architecture-advisor/SKILL.md` catalog.

Base the diagnosis on implementation evidence and classify it as detected
with evidence, inferred with uncertainty, or no identifiable architecture.
Separate the architectural style, the organization strategy, and the
language/framework conventions, and address reusable types, interfaces, and
contracts separately from their consumers.

**The proposal must include a Folder Architecture Decision and Map** (layers/modules,
responsibility→folder, allowed dependency directions, contract/type location
per module, co-location rules, public module API) and the **contract/type
placement** per folder, following the ownership principle from
`architecture-advisor`. If no coherent architecture exists, apply the
complexity-based decision (small→layer-first, medium→feature-first,
large→feature modules with boundaries) and document the rationale.

**The proposal must also include a File Responsibility Map for each new
or significantly changed module**: which file owns contracts/types
(`types.ts`/`contracts.ts`/`schemas.ts`), which owns static data/catalogs
(`messages/`, `catalogs/`), which owns configuration (`config.ts`),
which owns behavior/formatters (`format.ts`, `utils.ts`), and which file,
if any, exposes the module's public API. **Rule: one interface/type per file in its owning layer's folder.** Exception: multiple types with the **same exclusive consumer** are grouped in one file. A barrel (`index.ts`) is optional
and justified only by multiple public exports or consumers, a stable module
boundary, a substantially populated folder, or an existing project
convention; never add one merely to re-export a single file. For i18n modules, this means separating
types, locale config, per-locale catalogs, and formatters. For API modules,
separate schemas + inferred types (`contracts.ts`) from validators,
converters, and clients (`client.ts`). A type is extracted only when **actually reused from
another file**; mere export without reuse is a WARN/observation.

**File cohesion:** where the detected ecosystem supports it, keep one
primary component, class, module, or other unit per file. **Rule: one interface/type per file in its owning layer's folder.** Exception: multiple types with the **same exclusive consumer** are grouped in one file. Follow the
applicability matrix in `skills/implementation-recipes/SKILL.md` and the
evidence in `references/official-documentation.md`; never impose a rule an
ecosystem does not support. A component file that exports several
independent components with their own behavior is a dumping ground, not a
cohesive unit. Small private non-exported helpers may stay with their unit.
Angular follows its official "one concept per file" rule, including its
documented exception for small related components or directives.

**For every unit with multiple related artifacts — implementation, test,
styles, stories, fixtures, or subcomponents — use a unit folder and
co-locate the test with the unit, as the ecosystem supports.** Do not leave
`X`, `X.test`, and `X/` as siblings. Apply the rule according to the
detected ecosystem's official convention (component folders for
React/Vue/Svelte/Angular/Solid; mirrored test packages for Java/Kotlin;
`test_<module>.py` for Python; `_test.go` for Go; `tests/` for Rust;
mirrored test projects for C#/.NET; and so on).

A single trivial file with no test, styles, or subcomponents may remain a
single file. Consult `references/official-documentation.md` ("Project
Structure and Test Co-location") and the applicability matrix in
`skills/implementation-recipes/SKILL.md` before deciding the layout.

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

- If the project has a **coherent** architecture (detected structure with
  clear boundaries and consistent import directions), continue it unless
  there is an important incompatibility.
- If no clear architecture is detected, the detected structure is
  incoherent (e.g., flat `lib/` mixing domains), or conventions conflict,
  present the diagnosis and a recommendation with concrete alternatives and
  trade-offs, **including the complexity-based decision** from the table
  in `architecture-advisor`, then ask the
  user whether to preserve the existing organization or adopt the
  recommended alternative.
- Wait for the answer before implementing. Do not silently turn a
  recommendation into an approved decision.
- Do not ask about routine decisions that can be inferred from the code.
- If the user specifies an architecture, respect it and explain any
  significant incompatibility before implementing it.

The proposal must include a sufficiently concrete target structure, the
location of relevant files, the **Folder Architecture Map**, the contracts
and types, the direction of dependencies, and the pending decisions; stating
"architecture evaluated" is not sufficient.

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
- proposed new components only when justified per the Refactoring and Abstraction Justification gate in `skills/complexity-advisor/SKILL.md`
- formatting and linting setup when the project lacks it
- the final Context Manifest and exploration budget for the selected rigor
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

Implement on the requested branch and only within the target project.

**Mandatory compliance check:** run `workflows/output-compliance.md` over the
change set before the task is considered implemented. It mechanically detects
`function` declarations where an arrow function applies, files that mix type
declarations with runtime behavior, contract files whose types have different
consumers, and unit-folder/test co-location violations. A `FAIL` result is
the task's own defect: fix it and re-run. Do not report completion with an
unresolved `FAIL`.

Apply the minimum sufficient context policy and stop rules from
`workflows/session-resilience.md`. Do not keep exploring after two
consecutive searches add no new evidence, and record any budget expansion
with its reason.

Do not modify evidence-only paths, reference projects, design artifacts, or
external repositories. If a required change appears to fall outside the
target project, stop and request explicit authorization.

Preserve existing conventions.

Follow `skills/implementation-recipes/SKILL.md` for the detected stack and
apply its non-negotiable quality baseline: quality is never proportional to
task size. A trivial change is implemented carefully, not quickly at the
expense of conventions, error handling, typing, security, or tests.

**Official documentation consultation (on-demand during implementation):**

During implementation, **consult `references/official-documentation.md` on-demand** when encountering:

- Unfamiliar APIs or framework-specific behavior (e.g., Next.js Server Actions, React Server Components, Zod refinements)
- Framework-specific implementation decisions (e.g., Next.js App Router data fetching, React Server Components patterns)
- Deprecated or experimental features
- Version compatibility issues
- Security-sensitive behavior (auth, encryption, headers)
- Migration and upgrade guidance
- Uncertain performance or rendering behavior

**When to consult:** On-demand, only when the implementation hits a technical question not resolved by existing code, conventions, or internal rules.
**Do NOT consult:** For routine implementation (boilerplate, wiring, standard patterns already established in the codebase).
**Scope:** Only the specific section needed to resolve the question. Do not browse entire documentation sites.

Proceed autonomously for routine work after alignment. Do not pause for user approval for each file edit, test, formatting change, local refactoring, or ordinary corrective adjustment within the approved scope. Do not weaken quality, security, testing, or validation standards.

Provide concise status updates at meaningful milestones (alignment complete, milestone completed, important new finding or decision required, validation results, final diff/summary). Each update must explain what changed, the evidence available, and the next step. An update is not a request for approval unless a decision gate applies.

If new evidence changes a material requirement or approved design, pause and explain the reason before continuing beyond the approved scope. Otherwise, proceed autonomously.

Use checkpoints according to `workflows/session-resilience.md`. Do not require restarting the terminal.

Do not expand the scope silently. Any new component, module, or abstraction must satisfy the Refactoring and Abstraction Justification gate in `skills/complexity-advisor/SKILL.md`; an over-engineering finding from review must be resolved (revert or justify with evidence), not waived.

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

Propose the commit groups, messages, and files.

Present the proposal and ask for explicit approval in the current
conversation. Invoking `/task` or a general request to implement the task is
not commit approval.

Create the commit only after the user replies affirmatively. If the user
declines or requests changes, do not commit; adjust the proposal and ask
again. If the approved scope or message changes, re-present the proposal.

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
