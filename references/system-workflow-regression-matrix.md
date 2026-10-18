# System Workflow Regression Matrix

## Purpose

A lightweight reference for protecting the transversal behavior of the
Engineering System. It helps select targeted checks when a change touches
shared policies, cross-workflow boundaries, or orchestration.

## Status and Scope

- This is a reference, not a canonical workflow, and not a mandatory
  procedure for every task. It does not add a required acceptance phase.
- It does not define a closed architecture, a fixed process order, or a new
  layer of the system.
- Scenarios are selected according to the files, policies, and behaviors a
  change actually affects. Do not run the whole matrix by default.
- Broader integral validation is reserved for significant changes: the
  orchestrator, permission boundaries, or the interaction between several
  workflows.
- The canonical policies remain the source of truth. This reference never
  overrides or duplicates the decision policy of the task, dependency,
  validation, provider, delivery, review, or session-resilience workflows.

## When to Use

Use selectively when a change touches one of these areas:

- branch preparation or the branch policy
- the dependency approval gate or its refusal path
- architecture evaluation, the architecture decision gate, or
  code-organization conventions
- mandatory formatting and linting, or validation reporting
- provider or delivery read-only boundaries
- pipeline authoring, environment promotion, or deployment gates
- PR preparation or review character
- checkpoint identity, preservation, or recovery behavior
- importing, adapting, or recording external web quality Skills and their
  provenance or license
- planning rigor, specification depth, or the planning and clarification
  boundary
- interface or API contract design, or the pre-implementation contract step
- the implementation quality baseline or stack implementation guidance
- schema, migration, query, index, or transaction changes
- the architecture decision-record recording threshold

For an ordinary localized implementation that touches none of these, this
reference requires no action.

## Evidence Classes

Record each outcome with exactly one evidence class, and never conflate
them:

- **REAL**: execution against an available local project or an authorized
  external provider.
- **SIMULATED**: execution using a fixture or mock, including a simulated
  approval or provider response.
- **STATIC**: inspection of instructions, references, configuration,
  permissions, or repository state without runtime execution.

A static check does not prove runtime behavior, a simulated provider does
not prove authentication, and a passing build does not prove the whole
workflow. When a check cannot be executed, record BLOCKED or NOT_VERIFIED
with the concrete reason.

## Critical Transversal Scenarios

| ID | Protected behavior | Select when the change touches | Expected invariant | Evidence |
|---|---|---|---|---|
| R-01 | Branch name requested before code changes | Branch policy, `/task`, `workflows/task-to-code.md` | The exact branch name is requested before implementation | Interaction transcript or documented invocation |
| R-02 | Branch collision and correct base | Branch preparation | An existing branch stops the flow; the base is taken from repository state | `git branch`, `git status` |
| R-03 | Preservation of uncommitted work | Branch preparation, checkpoint, recovery | Staged, unstaged, and untracked changes are not lost | Before/after `git status --short` |
| R-04 | Dependency approval gate | `workflows/dependency-assessment.md`, `AGENTS.md` | No install before explicit authorization; manifests and lockfiles unchanged by the proposal | Manifest and lockfile diff or hashes before approval |
| R-05 | Dependency refusal path | `workflows/dependency-assessment.md` | No install; scope is not changed silently | Recorded outcome |
| R-06 | Mandatory formatter and linter scope | `AGENTS.md`, `workflows/project-validation.md` | Missing in-scope tooling is `SETUP_REQUIRED`, never `NOT_APPLICABLE`; a documentation-only repository is out of scope | Recorded status |
| R-07 | Honest validation reporting | `workflows/project-validation.md` | Reported statuses match observed command output | Command output and exit codes |
| R-08 | Provider capability separation | `workflows/provider-integration-proof.md` | Configuration, authentication, read evidence, and untested write are distinguished; simulation is never real | Provider report |
| R-09 | Optional providers absent | `workflows/provider-integration-proof.md` | An absent optional provider is not a core failure | Status `NOT_CONFIGURED` |
| R-10 | Delivery read-only boundary | `workflows/secure-delivery.md` | Assessment changes no configuration or remote state | Delivery report |
| R-11 | PR and review are copy-only | `workflows/pull-request-review.md`, `commands/pr.md`, `commands/review-pr.md` | Nothing is published, approved, merged, or pushed | Absence of remote actions |
| R-12 | Review grounded in the revision | `workflows/pull-request-review.md` | Findings correspond to the reviewed revision; no invented lines or results | Report plus reviewed diff |
| R-13 | Checkpoint identity and preservation | `workflows/session-resilience.md` | Distinct tasks have distinct checkpoints; mixed working-tree state is preserved; no source file changes | Checkpoint metadata plus `git status` |
| R-14 | Recovery divergence and next action | `workflows/session-resilience.md` | Divergence is reported, work is preserved, and the next action derives from actual repository state | Recovery report |
| R-15 | Architecture evaluated before implementation | `workflows/task-to-code.md`, `skills/architecture-advisor/SKILL.md` | Placement, reuse, module conventions, and styling decisions are evaluated before code changes | Recorded architecture evaluation |
| R-16 | Post-implementation architectural validation | `workflows/task-to-code.md`, `skills/engineering-review/SKILL.md` | The result is contrasted with the proposed design and project conventions | Review report |
| R-17 | Evidence-based architecture diagnosis | `skills/architecture-advisor/SKILL.md`, `skills/project-analysis/SKILL.md` | Detected, inferred, and absent architecture are distinguished; directory names alone do not establish an architecture | Recorded diagnosis with evidence |
| R-18 | Mandatory architecture decision gate | `workflows/task-to-code.md`, `commands/task.md`, `skills/architecture-advisor/SKILL.md` | When architecture is unclear or conflicting or a new one is required, options and trade-offs are presented and the user's decision is awaited before implementing | Recorded decision |
| R-19 | Post-implementation validation with evidence | `workflows/task-to-code.md`, `skills/engineering-review/SKILL.md`, `skills/architecture-advisor/SKILL.md` | The result is compared to the approved design with concrete evidence; documentation consistency is not conflated with code quality or with proven behavior | Review report |
| R-20 | Supported-stack architecture catalog and honest system check | `skills/architecture-advisor/SKILL.md`, `skills/stack-detection/SKILL.md`, `workflows/system-validation.md` | Alternatives fit the detected supported ecosystem; `/system-check` validates written consistency without claiming behavioral proof | System consistency report plus separate task-flow and application evidence |
| R-21 | Web quality Skill provenance and license | `integrations/external-skill-registry.md`, `integrations/skill-catalog.md`, `skills/accessibility/`, `skills/seo/`, `skills/performance/`, `skills/core-web-vitals/`, `skills/best-practices/` | Imported web quality Skills carry the pinned upstream commit, an MIT license file, and documented local modifications; the upstream orchestration Skill is not imported | Registry and file-hash checks |
| R-22 | Web quality evidence integrity | `skills/performance/SKILL.md`, `skills/core-web-vitals/SKILL.md`, `skills/accessibility/SKILL.md`, `workflows/system-validation.md` | Field, lab, and hypothesis evidence stay distinct; a lab or Lighthouse score is never presented as WCAG conformance or as a ranking prediction | Recorded findings |
| R-23 | Web quality application-type gate | `skills/seo/SKILL.md`, `skills/core-web-vitals/SKILL.md`, `integrations/skill-catalog.md` | SEO and Core Web Vitals checks apply only to web-facing surfaces; non-web deliverables are reported as NOT_APPLICABLE rather than scored | Recorded applicability decision |
| R-24 | Memory-profiling gap disclosed | `skills/performance/SKILL.md`, `integrations/external-skill-registry.md`, `workflows/system-validation.md` | The absence of a heap-profiling workflow is documented and memory leaks are not claimed to be detected automatically | Recorded limitation |
| R-25 | Proportional planning rigor | `skills/complexity-advisor/SKILL.md`, `workflows/task-planning.md`, `workflows/task-to-code.md` | Simple changes are not forced into a heavy specification; complex changes define their expected behavior before implementing | Recorded plan with its rigor level |
| R-26 | Expected behavior before implementation and risk-based gates | `workflows/task-planning.md`, `workflows/task-to-code.md`, `AGENTS.md` | A complex or high-risk change defines its expected behavior first; approval is triggered by a Human Decision Gate, not by the classification label; a Critical task requires reinforced controls | Recorded plan and approval status |
| R-27 | No redundant planning artifacts | `workflows/task-planning.md`, `skills/task-context/SKILL.md` | Plans are session state by default; a durable artifact is created only when justified; no duplicate specification of the code is produced | Recorded artifact decision |
| R-28 | Portable proportional planning | `workflows/task-planning.md`, `commands/task.md`, `agents/orchestrator.md` | Planning lives in the common core and does not depend on OpenCode; commands and the orchestrator delegate instead of restating the logic | Static inspection |
| R-29 | Formatter/linter tool lifecycle states | `workflows/project-validation.md`, `commands/validate.md`, `skills/stack-detection/SKILL.md` | Detected, configured, installed, executed, and approved are reported per tool and kept independent of the check-result statuses; a declared dependency is not reported as installed and an installed tool is not reported as executed until its check ran | Recorded tool states |
| R-30 | Conflict-aware tooling selection and configuration | `workflows/project-validation.md`, `skills/stack-detection/SKILL.md`, `AGENTS.md` | Missing tooling yields an explicit proposal aligned with the AGENTS.md defaults; existing working configuration is preserved; overlapping formatter/linter rules are disabled instead of left to disagree; no redundant second tool is added | Recorded proposal and configuration diff |
| R-31 | Monorepo-scoped tooling setup | `workflows/project-validation.md` | Configuration, scripts, and dependencies are limited to the affected packages; unrelated packages are not reconfigured or reformatted | Diff scope |
| R-32 | Tooling setup authorization without redundant approval | `workflows/project-validation.md`, `AGENTS.md`, `workflows/dependency-assessment.md` | The standard mandatory formatter/linter setup is authorized by policy without a redundant prompt; a paid tool, a non-standard dependency, or replacing a working configuration still hits a gate; a real restriction is reported as BLOCKED, never as success | Recorded outcome |
| R-33 | Contract defined before implementation | `skills/api-contract-design/SKILL.md`, `workflows/task-to-code.md` (Phase 8), `skills/architecture-advisor/SKILL.md` | When a task changes a boundary, interface, or integration, the contract (schemas, error model, authorization, versioning, compatibility) is defined before implementation; the existing contract style is reused, and a new style or breaking change requires approval | Recorded contract |
| R-34 | Quality is not proportional to task size | `AGENTS.md`, `skills/complexity-advisor/SKILL.md`, `skills/implementation-recipes/SKILL.md`, `workflows/task-to-code.md` (Phase 10) | Only planning rigor and test depth are proportional; a trivial change still follows conventions, typing, error handling, security, and tests, and is never implemented in a junior way | Review report |
| R-35 | Database changes are safe and gated | `skills/database-engineering/SKILL.md`, `workflows/task-to-code.md` (Phase 8), `agents/orchestrator.md` | The Skill loads only when a task touches the schema, migrations, queries, indexes, or transactions; migrations are backward-compatible and applied through project tooling; queries are parameterized and avoid N+1; destructive or production data operations require explicit human approval | Database plan |
| R-36 | Delivery pipeline is authored with environments and pre-deploy gates | `workflows/secure-delivery.md`, `workflows/task-to-code.md`, `agents/orchestrator.md`, `commands/delivery-check.md` | Authoring is an authorized mode distinct from the read-only `/delivery-check`; one artifact is promoted across environments with differences kept out of the artifact; the same validation gates run before every deployment and are never weakened; production requires an explicit approval gate; a failing pipeline is met with a proposed fix or rollback, never a bypass | Proposed pipeline configuration and environment/gate summary |

## Broader Integral Validation

Reserve a fuller sweep for significant or cross-cutting changes:

- orchestrator sequencing and lifecycle delegation
- agent and command permission boundaries
- interaction between several workflows

In those cases run the relevant scenarios above together with a complete
`workflows/system-validation.md` pass. This is a selection decision, not a
mandatory step for every task.

## Harness

An optional external, disposable harness may be used for scenarios that need
a project to run against. Keep provider fixtures separate from real provider
checks, and never create a `package.json`, formatter or linter tooling, or a
testing framework in the canonical documentation repository. Destructive or
divergent scenarios use a separate disposable repository and clean up only
the assets they create.

## Safety

- Read-only with respect to the canonical repository.
- Never install dependencies in the canonical repository.
- Never commit, push, merge, publish, approve, deploy, or modify remote
  state.
- Never expose credentials, tokens, or secret values.
- Never overwrite evidence from previous runs or discard user work.
- Report checks as NOT VERIFIED when the available tools cannot establish
  their result.
