# System Validation Workflow

## Purpose

Audit the consistency of the AI Engineering System and its client
integration without modifying files or remote services.

This workflow audits the **whole system** as it stands. It is not
`workflows/system-audit.md`, which applies a change and then audits that
change. Use this one for a consistency sweep; use the other one when a
change is being applied.

## 1. Resolve the Canonical Root

Determine the system root using this preference:

1. AI_ENGINEERING_HOME when configured and valid.
2. The repository root of the current working directory.
3. The resolved target of the global OpenCode AGENTS.md link.
4. $HOME/Documents/ai-engineering, only if that directory exists.

Do not assume the current working directory is the system root.

If the root cannot be established, report the missing information.

## 2. Validate the Core

Check for the existence of:

- AGENTS.md
- CLAUDE.md
- README.md
- agents/orchestrator.md
- agents/reviewer.md
- agents/auditor.md
- commands/task.md
- commands/generate-pr.md
- commands/review-pr.md
- commands/commit.md
- commands/audit-e2e.md
- commands/create-project.md
- commands/system-check.md
- commands/validate.md
- commands/code-audit.md
- commands/checkpoint.md
- commands/resume-work.md
- commands/provider-check.md
- commands/delivery-check.md
- commands/estimate.md
- contracts/task-provider.md
- contracts/design-provider.md
- contracts/git-provider.md
- integrations/access-strategy.md
- integrations/provider-policy.md
- integrations/provider-matrix.md
- integrations/external-skill-registry.md
- integrations/skill-catalog.md
- references/official-documentation.md
- references/linter-rules.md
- references/system-workflow-regression-matrix.md
- references/token-efficiency.md
- references/mandatory-rules.md
- references/project-startup-decisions.md
- references/project-creation-order.md
- workflows/task-to-code.md
- workflows/pull-request-review.md
- workflows/code-audit.md
- workflows/commit.md
- workflows/system-validation.md
- workflows/project-validation.md
- workflows/session-resilience.md
- workflows/self-learning.md
- workflows/provider-integration-proof.md
- workflows/secure-delivery.md
- workflows/dependency-assessment.md
- workflows/task-planning.md
- workflows/estimation.md
- workflows/project-initialization.md
- workflows/system-audit.md
- workflows/output-compliance.md

Check that CLAUDE.md references AGENTS.md rather than duplicating
the complete shared instructions.

Do not hardcode a required-file total. The inventory above is the
maintained list; report the count as derived from the actual file
existence check, not from a fixed constant. A partial commit that removes
one of the required files must fail this check.

## 3. Validate Skills

Check that every referenced Skill exists at its canonical location.

At minimum inspect:

- discovery
- project-analysis
- stack-detection
- task-context
- design-analysis
- complexity-advisor
- architecture-advisor
- implementation-recipes
- api-contract-design
- database-engineering
- internationalization
- testing-strategy
- engineering-review
- git-workflow
- repository-strategy
- pull-request
- pull-request-review
- review-memory
- accessibility
- seo
- performance
- core-web-vitals
- best-practices

Do not require a dedicated Next.js Skill.

Next.js must be covered by stack detection.

## 3.1 Validate External Skills

Read:

- integrations/external-skill-registry.md

For every registered external Skill, verify:

- canonical local directory exists
- SKILL.md exists and contains valid metadata
- source repository and source path are recorded
- upstream commit is recorded as a full Git commit SHA
- license identifier is recorded
- required license file exists
- supporting files referenced by SKILL.md exist
- local modifications are documented
- no conflicting internal Skill or policy is introduced

For the initially selected Skills, verify:

### frontend-design

- Path: skills/frontend-design/
- Required files: SKILL.md and LICENSE.txt
- License: Apache-2.0

### test-driven-development

- Path: skills/test-driven-development/
- Required files: SKILL.md, writing-good-tests.md, and LICENSE-MIT.txt
- License: MIT
- Verify that the local language-neutral adaptation is documented.

### accessibility

- Path: skills/accessibility/
- Required files: SKILL.md, references/WCAG.md,
  references/A11Y-PATTERNS.md, and LICENSE.txt
- License: MIT
- Verify that the removed orchestration reference is documented.

### seo

- Path: skills/seo/
- Required files: SKILL.md, references/STRUCTURED-DATA.md, and LICENSE.txt
- License: MIT
- Verify the application-type gate and the removed orchestration reference
  are documented.

### performance

- Path: skills/performance/
- Required files: SKILL.md, references/MEASUREMENT.md, references/RUM.md,
  and LICENSE.txt
- License: MIT
- Verify the documented memory-profiling limitation.

### core-web-vitals

- Path: skills/core-web-vitals/
- Required files: SKILL.md, references/LCP.md, references/INP.md,
  references/CLS.md, and LICENSE.txt
- License: MIT
- Verify the application-type gate is documented.

### best-practices

- Path: skills/best-practices/
- Required files: SKILL.md, references/SECURITY.md, and LICENSE.txt
- License: MIT
- Verify that the removed orchestration reference is documented.

Possible statuses:

- VERIFIED
- NOT INSTALLED
- LICENSE MISSING
- SOURCE REVISION MISSING
- SUPPORTING FILE MISSING
- POLICY CONFLICT
- NOT VERIFIED

Never report an external Skill as VERIFIED based only on its registry entry.

Report Required External Skill readiness as BLOCKED while a required
external Skill is missing or its license and source cannot be verified.

## License Metadata Interpretation

The registry records the canonical SPDX license identifier.

The upstream Skill frontmatter may use a license-file reference, an SPDX
identifier, or omit a license field.

Do not require the frontmatter value to match the registry SPDX identifier
literally.

For frontend-design:

- Accept the upstream metadata "Complete terms in LICENSE.txt".
- Verify that LICENSE.txt exists.
- Verify that the registry records Apache-2.0.

For test-driven-development:

- A missing license field in SKILL.md is not automatically an error.
- Verify that LICENSE-MIT.txt exists and contains the MIT license.
- Verify that the registry records MIT.

Report LICENSE METADATA INCONSISTENCY only if the metadata contradicts
the actual license evidence or points to a missing license file.

Do not modify upstream Skill metadata solely to satisfy a simplistic
string-equality check.

## 4. Validate Client Adapters

For OpenCode, inspect:

- global AGENTS.md configuration
- global command discovery
- global agent discovery
- global Skill discovery
- command descriptions
- referenced agent names
- agent frontmatter
- relevant permissions

Inspect existing paths before suggesting any link changes.

Never overwrite an existing configuration.

For Codex CLI, inspect:

- the installed `codex --version`
- the canonical README's Codex CLI setup instructions
- the model-visible input from `codex debug prompt-input` when available
- whether instructions are auto-loaded from the current project's
  `AGENTS.md` and whether shared instructions and Skills are explicitly
  referenced when the canonical repository is outside the project
- whether the setup incorrectly claims that files in `commands/` become
  Codex slash commands

Classify `debug prompt-input` as STATIC evidence: it shows what Codex would
receive, not that an agent read the referenced files or completed a task. A
successful `codex exec` response is REAL behavioral evidence. If execution
fails before the agent responds, report behavior as NOT VERIFIED and include
the observed failure. Do not initiate billable execution without
authorization. Do not modify global Codex configuration during validation.

### External-Directory Permissions

Verify against the actual global client configuration, not only the policy
text, that external-directory access remains narrowly scoped:

- external-directory access does not default to an unrestricted wildcard
  `allow`
- the default action for unmatched external paths is `ask` or stricter,
  never a blanket `allow`
- the checkpoint handoff directory defined in
  `workflows/session-resilience.md` remains accessible under an explicit,
  narrow rule
- more-specific rules follow the wildcard so the intended rule wins
- no broad allowance is granted for the entire home directory, all projects,
  or the whole filesystem
- reviewer and auditor agent permission restrictions remain in effect and
  are not widened by an inherited wildcard

Report the actual configured values. Do not infer them from policy text, and
do not modify the client configuration during validation.

## 5. Validate Policies

Confirm consistency across the Core, Skills, commands, and workflows:

- Ask for the branch name before implementation when not supplied.
- Preserve uncommitted changes.
- Use the correct base branch.
- Do not create a PR or publish review comments remotely.
- Generate copy-ready PR and review content.
- Keep review workflows read-only.
- Use architecture patterns only when justified.
- Support direct URLs and local artifacts.
- Treat MCP as optional.
- Keep credentials out of version control.

## Policy Conflict Detection

Inspect active instructions, not only headings and file references.

Verify that:

- provider-policy.md contains exactly one Cost Classification section
- provider-policy.md and provider-matrix.md use the same cost taxonomy
- TDD does not authorize deleting existing work
- TDD does not require fixing unrelated test failures automatically
- TDD test requirements are proportional to task complexity and risk
- the external Skill registry accurately documents local modifications
- /system-check validates the Engineering System, not application code
- /validate validates the current software project, not the system configuration
- validation commands are discovered from actual project configuration
- no universal test runner or coverage threshold is assumed
- validation scope is proportionate to task complexity and risk
- missing dependencies are reported rather than installed automatically,
  except for the mandatory formatting and linting setup
- validation does not modify application behavior; the only permitted
  source mutation is automated formatting of files within the declared
  task scope per the mandatory formatting and linting policy
- potentially destructive commands require appropriate authorization
- formatting and linting are configured and treated as mandatory
- missing formatter or linter tooling is reported as SETUP_REQUIRED,
  never as NOT_APPLICABLE
- workflows authorize the mandatory formatter and linter setup instead of
  unconditionally forbidding dependency installation and configuration
- AGENTS.md, the testing Skill, and the validation workflows agree on the
  formatting and linting policy
- the formatting and linting policy is scoped to the application code of
  software projects the system works on, and does not require converting
  documentation-only knowledge repositories into software projects
- no active instruction requires creating a package.json or installing
  formatter or linter tooling in the canonical ai-engineering repository
  solely to satisfy the policy
- reproducible lint, format, and format-check commands are required, not
  optional
- no active instruction claims validation passed without evidence or
  treats a successful build as a substitute for formatting and linting
- the required-file inventory above matches the actual maintained set of
  canonical files
- `agents/orchestrator.md` delegates the detailed task and PR-review
  sequences to the canonical workflows and does not define a parallel,
  contradictory order
- `/task` continues from the first incomplete phase of the canonical
  Task-to-Code workflow and does not repeat analysis already completed
  before branch preparation

## Review Quality and Evidence

Verify the Pull Request review workflow maintains evidence quality and
remains read-only and copy-only:

- every finding requires supporting evidence and an actionable explanation
- file references and line ranges are grounded in the reviewed revision,
  not assigned without inspection
- severity is proportionate to demonstrated impact and is not inflated for
  sensitive domains alone
- confirmed defects are separated from observations, questions, and
  design-review items
- findings are scoped to the requested change unless a broader audit is
  explicitly requested
- test or command execution is not claimed without an actual result
- /review-pr is read-only and copy-only
- no remote comments, approvals, merges, commits, or pushes are authorized
  by the review workflow
- the review workflow respects provider access and authorization rules

Distinguish active instructions from quoted examples or explanatory text.

If an active instruction conflicts with AGENTS.md, report POLICY CONFLICT.

A successful file-existence check alone does not establish policy
consistency.

## Session Resilience and Checkpoint Safety

Verify that checkpoint and recovery behavior is defined once and remains
safe:

- `workflows/session-resilience.md` is the single canonical definition
- `/checkpoint` and `/resume-work` delegate to that workflow instead of
  duplicating its logic
- checkpoints are written to a provider-neutral user-level handoff directory
  outside the target project's source tree, using the same state root as
  self-learning and no single-client path
- checkpointing does not modify source files, commits, branches, or remote
  state
- a checkpoint never overwrites a different task's checkpoint
- checkpoints never store credentials, tokens, or secrets
- recovery verifies the actual repository, branch, HEAD, and working-tree
  state instead of trusting the checkpoint as proof
- recovery treats the checkpoint as a guide, detects divergence, preserves
  current work, and continues from the first incomplete action instead of
  repeating completed phases
- uncommitted changes are preserved during both checkpointing and recovery
- context continuity across `/compact` and `/new` is preserved without
  requiring the terminal to be closed and reopened
- no instruction claims a prior validation passed without recorded evidence
  or re-execution
- automatic compaction remains enabled and no undocumented context-threshold
  configuration keys are introduced
- no active instruction requires closing and reopening the terminal as the
  normal recovery path
- no experimental compaction plugin is required by the policy
- command references in AGENTS.md and README.md match the actual command
  files

### Checkpoint Identity and Duplicate Detection

Verify checkpoint slug and overwrite rules statically:

- the literal handoff directory path is defined in exactly one place,
  `workflows/session-resilience.md`
- the provider-neutral state root is used by both the handoff directory and
  the self-learning store, with no client-specific path such as a fixed
  OpenCode, Claude, Codex, or Aider directory
- AGENTS.md delegates to that workflow instead of repeating the literal
  path, and `/checkpoint` and `/resume-work` reference the workflow's path
  without duplicating it
- checkpoint identity uses the task identifier or a generated checkpoint
  identifier and never relies on the branch name alone
- task and checkpoint identifiers are normalized into safe file names:
  path separators, `.`/`..` components, and other unsafe characters are
  rejected before any path is derived
- `/checkpoint` resolves the final checkpoint path and verifies it stays
  inside the authorized handoff directory, rejecting identifiers, paths,
  or symlinks that would escape it
- `/resume-work` applies the same normalization when resolving a supplied
  task identifier or checkpoint path
- no two checkpoint-writing implementations exist; `/checkpoint` and
  `/resume-work` delegate to the workflow and do not duplicate its logic
- `/checkpoint` rejects checkpoint paths outside the authorized handoff
  directory
- `/resume-work` lists multiple matching checkpoints and asks for a
  selection instead of picking one by branch name
- no instruction authorizes creating or updating another task's checkpoint
- checkpoint contents require a task identifier or checkpoint identifier,
  so two tasks on the same branch remain distinguishable

## 6. Reference Integrity

Inspect references to Skills, workflows, contracts, agents, and commands.

Identify:

- referenced but missing files
- inconsistent filenames
- obsolete command names
- contradictory lifecycle instructions
- duplicate canonical instructions
- official-documentation.md contains documentation references for all
  supported technology families
- skill-catalog.md contains a decision for every supported technology
- every installed external Skill appears in external-skill-registry.md
- candidate Skills are not reported as installed
- installed external Skills have verified licenses and recorded source
  revisions
- `commands/delivery-check.md` and `commands/provider-check.md` reference
  existing workflows and contain no deployment, publish, or remote-write
  instruction for their read-only mode
- `workflows/secure-delivery.md` references only existing commands,
  workflows, and policies
- `references/system-workflow-regression-matrix.md` references only
  existing workflows, commands, and policies and does not restate a
  duplicated literal handoff path
- `skills/implementation-recipes/SKILL.md`,
  `skills/api-contract-design/SKILL.md`, and
  `skills/database-engineering/SKILL.md` exist and are referenced
  consistently by `agents/orchestrator.md`, `workflows/task-to-code.md`, and
  `README.md`; they do not duplicate the architecture decision policy owned
  by `skills/architecture-advisor/SKILL.md`
- `workflows/self-learning.md` exists and is referenced consistently by
  `AGENTS.md`, `agents/orchestrator.md`, and `README.md`; it defines the
  neutral state root once and never duplicates the handoff path or a
  client-specific directory
- `workflows/estimation.md` and `commands/estimate.md` exist and are
  referenced consistently by `AGENTS.md`, `agents/orchestrator.md`,
  `workflows/task-to-code.md`, and `README.md`; estimation is described as a
  standalone activity scoped to complete projects from start to finish (never
  to tickets, stories, or tasks) that never implements, and
  `commands/estimate.md` delegates to the workflow instead of restating its
  procedure
- `skills/internationalization/SKILL.md` exists and is referenced
  consistently by `agents/orchestrator.md`, `workflows/task-to-code.md`,
  `commands/task.md`, `skills/implementation-recipes/SKILL.md`, and
  `README.md`; localization is settled early for work with user-facing text
  and the Skill is not loaded for work without it

Do not flag a reference as broken merely because a file is intentionally
provided by the client rather than by the canonical repository.

## 7. Provider State

Distinguish:

- supported by the architecture
- documented as a candidate
- locally configured
- successfully tested

Do not claim a provider works merely because its name appears in a table.

Do not print credentials or secret environment-variable values.

### Provider Integration Proof

Verify against `workflows/provider-integration-proof.md`,
`integrations/provider-policy.md`, `integrations/access-strategy.md`,
`integrations/provider-matrix.md`, and the provider contracts that:

- provider statuses are supported by observed evidence
- simulation is never reported as real verification
- authentication and read verification are distinguished
- read and write capabilities are tracked independently
- no remote write operation is performed by the verification workflow
- credentials and secrets are not exposed
- absent optional providers do not incorrectly fail the provider-agnostic
  core
- `/provider-check` references the canonical workflow
  (`workflows/provider-integration-proof.md`) and respects read-only agent
  permissions
- provider policies remain consistent with the contracts and the access
  strategy
- provider verification never triggers unapproved paid usage or installs
  unnecessary integrations
- the documented commands and the required-file inventory remain correct
- the required-file count is derived from the actual inventory, never a
  hardcoded total

## 7.1 Secure Delivery and CI/CD Safety

Verify against `workflows/secure-delivery.md`, `commands/delivery-check.md`,
`workflows/project-validation.md`, and the existing command boundaries that:

- `/delivery-check` references `workflows/secure-delivery.md` and respects
  read-only agent permissions
- `secure-delivery.md` distinguishes read-only assessment, authorized
  implementation, execution of tests and scans, and publication or
  deployment that requires explicit authorization
- `/system-check`, `/validate`, `/provider-check`, and `/delivery-check`
  keep distinct responsibilities with no duplicate canonical policy
- tooling and CI decisions are project-specific and evidence-based rather
  than universal defaults or familiar-platform bias
- the mandatory software-project formatter/linter policy in AGENTS.md is
  unchanged and remains intact in the delivery workflow
- security checks never print, reproduce, or store discovered secret
  values; sensitive evidence is redacted to location and type
- configured checks and actually executed or passed checks are clearly
  distinguished
- a clean dependency audit is reported only when the audit command
  executed successfully and its vulnerability database was available
- simulated scans are never reported as real security results
- absent optional CI/CD platforms or providers are reported as
  NOT_CONFIGURED and do not incorrectly fail the provider-agnostic core
- no unapproved installations, billable integrations, remote writes,
  deployments, or publisher actions are authorized by the delivery
  workflow
- the delivery workflow never weakens required checks or security
  controls to obtain a green pipeline
- declarations of pipeline security or production readiness require
  supporting evidence, not static review alone
- NOT_APPLICABLE is used only when a control genuinely does not apply;
  missing in-scope mandatory tooling remains SETUP_REQUIRED
- a read-only assessment is not silently turned into implementation work
- pipeline authoring is an authorized mode distinct from the read-only
  `/delivery-check`; it promotes one artifact across environments, keeps
  environment differences out of the artifact, runs the required validation
  gates before every deployment, requires an explicit approval gate for
  production, and never weakens gates to obtain a green pipeline
- a failing pipeline is handled by proposing a bounded fix or a rollback,
  never by silently bypassing or disabling gates
- the required-file count is derived from the actual inventory, never a
  hardcoded total

## 7.2 Dependency Decision and Approval

Verify against `workflows/dependency-assessment.md`, AGENTS.md, the task
workflows, `commands/task.md`, and `skills/stack-detection/SKILL.md` that:

- every software implementation task performs the dependency assessment
  before its implementation plan is finalized
- the dependency assessment is defined once, canonically, in
  `workflows/dependency-assessment.md`, and other files reference it
  instead of duplicating its decision policy
- exactly one dependency outcome is recorded: `NO_NEW_DEPENDENCY`,
  `REUSE_EXISTING`, `NEW_DEPENDENCY_PROPOSED`, or `BLOCKED`
- existing dependencies are checked before a new one is recommended, and a
  dependency-free option is considered where reasonable
- a new-dependency recommendation includes evidence-based tradeoffs and
  does not invent versions, security status, or maintenance claims
- installation of a new runtime or development dependency requires explicit
  authorization, except for the established mandatory formatter/linter setup
- a general task approval is not treated as blanket authorization to
  introduce libraries
- a user refusal prevents installation, and the no-approval path does not
  silently change the task scope
- approved installation uses the project's package manager, updates the
  manifest and lockfile together without upgrading unrelated dependencies,
  and reports actual results
- an installation failure is reported rather than silently substituting a
  different dependency
- package manifests and lockfiles remain consistent
- no credentials, registry tokens, paid usage, or billable integrations are
  introduced by the assessment or installation
- the documentation-only ai-engineering repository is not converted into a
  software package to implement this workflow
- no new slash command was added unless a demonstrated use case required one

## 7.3 Orchestrator and Workflow Sequence Consistency

Verify against `agents/orchestrator.md`, `workflows/task-to-code.md`,
`workflows/pull-request-review.md`, and `commands/task.md` that:

- `agents/orchestrator.md` delegates the detailed, ordered task sequence to
  `workflows/task-to-code.md` instead of maintaining a parallel sequence
- the orchestrator project lifecycles do not contradict the canonical
  workflow's branch-preparation, analysis, planning, and implementation
  order
- the orchestrator delegates PR-review sequencing to
  `workflows/pull-request-review.md`
- `/task` follows the canonical Task-to-Code sequence, continues from the
  first incomplete phase, and reuses completed analysis instead of repeating
  task, design, repository, or stack inspection
- `workflows/task-to-code.md` defines the technical complexity and
  architecture evaluation in exactly one phase, reuses a completed result
  instead of repeating the affected-modules, architecture, pattern, and risk
  checklist, and extends it only when new evidence justifies it; the
  post-implementation architectural validation in the review phase is
  distinct from that evaluation and does not duplicate it
- architecture-only and estimation-only requests remain free of branch
  preparation, and software implementation still requires branch preparation
  before modifying application code
- no file in the core execution path defines a competing ordering of the
  lifecycle

## 7.4 System Workflow Regression Matrix

Verify that `references/system-workflow-regression-matrix.md` remains a
lightweight reference and not a competing procedure:

- it exists at its canonical path and references only existing workflows,
  commands, and policies
- it is explicitly described as a reference, not a mandatory procedure for
  every task, and it adds no required acceptance phase
- it states that scenarios are selected according to the files, policies, and
  behaviors a change affects, rather than run wholesale by default
- it states that it does not define a closed architecture, a fixed process
  order, or a new layer
- it reserves broader integral validation for significant changes: the
  orchestrator, permission boundaries, or interaction between several
  workflows
- it protects the critical transversal behaviors: the branch policy,
  preservation of uncommitted work, the dependency approval gate and refusal
  path, the mandatory formatter and linter scope together with its tool
  lifecycle states, honest validation reporting, provider and delivery
  read-only boundaries, PR and review copy-only character, checkpoint and
  recovery safety, contract-before-implementation, the quality-not-proportional
  baseline, database safety and destructive-operation gates, delivery pipeline
  authoring with environments and pre-deploy gates, provider-neutral
  self-learning with promotion gates, bounded context spend, standalone,
  project-scoped, and ranged estimation, early localization with correct implementation, mobile
  analysis and implementation, and proportional decision records
- REAL, SIMULATED, and STATIC evidence are defined and never conflated
- it does not duplicate the decision policy of the task, dependency,
  validation, provider, delivery, review, or session-resilience workflows,
  and references them instead
- it does not require creating a package.json, formatter or linter tooling,
  or a testing framework in the canonical repository
- no new command was introduced solely to host the regression matrix
- no active instruction in the regression matrix contradicts AGENTS.md or a
  canonical workflow

## 8. Output

Return:

### SYSTEM VALIDATION

- Canonical root:
- Client:
- Required files:
- Missing files:
- Invalid references:
- Policy contradictions:
- OpenCode discovery:
- Codex CLI instruction loading:
- Codex CLI behavioral execution:
- Provider configuration status:
- Delivery and CI/CD state:
- Dependency decision state:
- License status:
- External Skill registry:
- External Skills installed:
- External Skill licenses:
- Upstream commit verification:
- External Skill policy conflicts:
- System workflow regression matrix:
- V2.5 interactive autonomy and automatic task interpretation checks:
- V2.6 architecture-first checks:
- V2.7 architecture decision gate checks:
- V2.8 architecture catalog and evidence boundary checks:
- V2.12 web quality checks:
- V2.13 proportional planning checks:
- V2.14 formatter/linter setup checks:
- V2.15 contract and recipe checks:
- V2.16 database engineering checks:
- V2.17 delivery pipeline authoring checks:
- V2.18 neutral state root and single orchestrator checks:
- V2.19 estimation checks:
- V2.20 internationalization and mobile UI checks:
- V2.21 refactoring and abstraction justification checks:
- V2.22 code audit checks:
- V2.23 folder architecture and type ownership checks:
- V2.24 commit generation checks:
- V2.26 folder architecture by complexity, dumping ground prohibition, and reuse-based type ownership checks:
- V2.27 commit approval checks:
- V2.28 primary unit per file, unit folder, and test co-location checks:
- V2.29 target project and evidence-only path checks:
- V2.30 minimum sufficient context and token-efficiency checks:
- V2.31 structural compliance enforcement checks:
- V2.32 mandatory rule reachability checks:
- V2.33 new project initialization checks:
- V2.34 end-to-end audit checks:
- V2.35 progressive project creation checks:
- V2.36 target version and official documentation checks:
- V2.37 documentation freshness checks:
- V2.38 self-learning execution checks:
- V2.39 portability checks:
- V2.40 feature-driven structure checks:
- V2.41 folder architecture checks:
- V2.42 topology decision checks:
- V2.43 source grounding checks:
- V2.44 architecture matrix checks:
- External Skill readiness:

The numbering skips V2.9 to V2.11: those identifiers were never used, and
no rule, section, or reference depends on them. V2.25 was removed when its
checks were folded into V2.26 and V2.28; its index entry was left behind
and has been removed.

### Findings

Classify:

- P0: unsafe configuration or destructive risk
- P1: core workflow broken
- P2: important inconsistency
- P3: optional improvement

### Recommended Actions

Provide the smallest changes that resolve actual findings.

## Safety

This workflow is read-only.

Never create, delete, overwrite, commit, push, or publish anything.
Never expose secrets.

Report checks as NOT VERIFIED when the available tools cannot establish
their result.

## Documentation and Skill Selection

Verify that:

- technology and version detection precedes documentation selection
- only relevant documentation sources are retrieved
- existing project tooling is preferred
- candidates are not automatically installed
- repository-specific Skills are not treated as universal engineering rules
- external Skills cannot override AGENTS.md
- the absence of a dedicated framework Skill is not reported as a failure
  when official documentation provides adequate guidance

Verify that every technology family in
references/official-documentation.md has a corresponding decision in
integrations/skill-catalog.md.

A decision may be DOCS_ONLY, CANDIDATE, TASK_SPECIFIC, INSTALLED,
REJECTED, or REASSESS.

Do not require a dedicated external Skill when official documentation
is sufficient.

Report a missing catalog entry as P3 unless it causes an actual workflow
failure.

Also verify that Markdown code fences are balanced in the Skills,
workflows, and references. An unbalanced fence changes which instructions
are interpreted as text or as code, so treat it as a defect even when the
content remains readable.

Report an unbalanced fence in a Skill that is actively loaded as P2, and
in other documentation as P3.

## V2.5 Regression Coverage

Verify the following controls exist and are not contradictory:

- The user is not asked to make routine implementation decisions.
- The agent resolves discoverable technical choices independently.
- Material business ambiguities trigger concise clarification.
- The implementation approach is aligned before code changes.
- Routine code adjustments and validation proceed without repeated approval requests.
- Significant architectural decisions and new dependencies retain their existing authorization gates.
- Architecture-only and estimation-only requests do not silently generate application code.
- Status updates report progress but do not create artificial approval gates.
- The agent reuses previously completed analysis after alignment.
- Client-specific interaction features are used only where supported.
- Architecture-only and estimation-only requests do not require branch preparation or Git state changes unrelated to the requested analysis.
- Software implementation requests still require branch preparation before modifying application code per the branch policy.
- No conflicting rules have been introduced in AGENTS.md, the orchestrator, the task workflow, or commands.

## V2.5 Refinement — Automatic Task Interpretation

Verify the following controls exist and are not contradictory across
`AGENTS.md`, `agents/orchestrator.md`, `commands/task.md`,
`workflows/task-to-code.md`, and `skills/task-context/SKILL.md`:

- a short, ordinary task description is accepted as a complete task input,
  with no requirement to rewrite it into an elaborate prompt
- short tasks trigger repository-aware interpretation: the repository,
  code, conventions, configuration, tests, and designs are inspected
  before technical questions are asked of the user
- explicit task requirements and inferred requirements are distinguishable,
  and inferred requirements are never presented as explicit task text
- acceptance criteria are derived from task and repository evidence and
  are validated with the user only when a material ambiguity exists
- missing business or product decisions trigger selective, one-at-a-time
  questions instead of guesses
- the user is not asked to author or maintain a separate Markdown task
  specification, and no Markdown artifact is produced for the internal
  working interpretation by default
- the working interpretation is persisted only through the existing
  checkpoint mechanism when continuity requires it, or when the user
  requests a durable artifact
- routine implementation details remain autonomous after alignment
- branch preparation, dependency approval, architecture approval,
  security, and validation gates remain intact and unchanged
- repeated repository analysis is avoided by reusing completed analysis
- architecture-only and estimation-only requests do not automatically
  modify application code
- interpretation extends the existing task-context Skill and Task-to-Code
  workflow; no duplicate canonical workflow, task-transcription subsystem,
  or task-interpreter command was introduced
- task retrieval that fails never results in a claim that inaccessible
  task content was read; only the minimum missing information is requested

## V2.6 Regression — Architecture-First Engineering

Verify that architecture is evaluated before implementation and validated
after it, across frameworks, languages, and application types:

- `workflows/task-to-code.md` performs the architecture evaluation before
  the implementation phase and does not implement first
- the architecture evaluation covers code organization and placement,
  cohesion, reuse, module conventions (public exports, barrels, import
  aliases), design patterns only when justified, and styling decisions
- the architecture knowledge is defined once in
  `skills/architecture-advisor/SKILL.md`; no duplicate architecture Skill
  was created and no competing order was introduced
- `skills/engineering-review/SKILL.md` and the review phase require a
  post-implementation architectural validation that contrasts the result
  with the proposed design and the project's conventions, in addition to
  functional tests
- the rules are framework- and language-neutral: no universal folder
  structure, fixed file count, or mandatory pattern is imposed
- a styling-system decision that significantly affects architecture,
  dependencies, or the visual experience triggers a human decision gate
- routine technical details remain autonomous and are not turned into
  approval gates
- the architecture evaluation remains in exactly one phase, and the
  post-implementation validation is distinct from it
- `skills/project-analysis/SKILL.md` detects module and import conventions
  without introducing new ones during analysis

## V2.7 Regression — Architecture Decision Gate

Verify that the architecture diagnosis is evidence-based and that the
mandatory decision gate exists and is not contradictory across
`skills/architecture-advisor/SKILL.md`, `skills/project-analysis/SKILL.md`,
`workflows/task-to-code.md`, `commands/task.md`, `agents/orchestrator.md`,
and `references/system-workflow-regression-matrix.md`:

- the architecture knowledge remains defined once in
  `skills/architecture-advisor/SKILL.md`; no technology-specific Skill and no
  new document directory was created
- the canonical Skill requires inspecting real structure, entry points,
  imports, boundaries, stack-relevant elements, configuration, tests, and
  existing documentation before proposing a solution
- the diagnosis explicitly distinguishes detected with evidence, inferred
  with uncertainty, and no identifiable architecture, and states that
  finding `components`, `services`, or `lib` directories does not by itself
  establish an architecture
- the Skill separates architectural style, organization strategy, and
  language/framework conventions, and does not impose hexagonal, Clean,
  feature, microservice, or any other option universally
- the Skill defines the default rule to separate reusable types,
  interfaces, and contracts from their consumers, organized by
  feature/domain/module, with justified exceptions and respect for language
  conventions
- the Skill covers responsibility separation per stack without requiring
  empty folders or a fixed directory set
- the Skill and `workflows/task-to-code.md` define the mandatory decision
  gate: continue a coherent existing architecture, or present the diagnosis
  with a recommendation, concrete alternatives, and trade-offs, and wait for
  the user's decision before implementing; a recommendation is never
  silently treated as approved
- `commands/task.md` and `agents/orchestrator.md` delegate to that gate
  instead of defining a competing order
- the post-implementation validation requires concrete evidence and
  distinguishes documentation consistency of `ai-engineering`, compliance
  with the flow before implementing, and architectural quality of the
  resulting code; it does not claim the agent's behavior is proven by
  verifying instructions alone
- the Skill carries compact references to recognized architectural sources
  without creating a Skill per technology
- the V2.7 checks do not weaken the existing branch, dependency,
  validation, provider, delivery, review, or session-resilience gates
- `skills/project-analysis/SKILL.md` reports the confidence of the
  architecture classification explicitly

## V2.8 Regression — Architecture Catalog and Evidence Boundaries

Verify that the architecture catalog offers alternatives for the ecosystems
actually supported by `skills/stack-detection/SKILL.md`, without turning
them into a universal folder structure or architecture requirement:

- the canonical Skill distinguishes application architectural style, code
  organization, and language/framework conventions
- alternatives are documented for supported languages and frameworks and
  are selected only after identifying the actual stack
- architecture is detected from implementation evidence; when no coherent
  architecture is identified or evidence conflicts, the flow presents
  appropriate alternatives and trade-offs and waits for approval on an
  important unresolved decision
- guidance integrates reusable types, interfaces, contracts, components,
  services, hooks/composables, contexts/providers, and adapters where
  relevant, without requiring universal directories or empty folders
- the catalog remains canonical in the existing architecture-advisor Skill;
  no duplicate architecture document or per-framework architecture Skill is
  introduced
- `/system-check` validates consistency and completeness of the written
  catalog and references only; it does not claim to execute `/task`, prove
  agent behavior, inspect an arbitrary application's architecture, or
  validate application code behavior
- documentation consistency, flow compliance, implemented architecture
  quality, and functional behavior remain separate findings

## V2.12 Regression — Web Quality Skills

Verify that the imported web quality Skills remain faithfully vendored,
honest about their evidence, and correctly gated:

- `skills/accessibility/`, `skills/seo/`, `skills/performance/`,
  `skills/core-web-vitals/`, and `skills/best-practices/` exist and each
  contains SKILL.md, its referenced supporting files, and LICENSE.txt
- every imported Skill is recorded in
  `integrations/external-skill-registry.md` with the pinned upstream commit
  `afa8da942115f2961fdbfa80807ea0b232ff6c00`, an MIT license, and documented
  local modifications, and each is listed as INSTALLED in
  `integrations/skill-catalog.md`
- the upstream `web-quality-audit` orchestration Skill and its
  `scripts/analyze.sh` are neither present nor referenced by any imported
  Skill; no second orchestrator was introduced
- field, lab, and hypothesis evidence remain distinct: a Lighthouse or lab
  score is never reported as WCAG conformance or as a prediction of
  rankings, and static findings are labeled as hypotheses
- SEO and Core Web Vitals checks are gated by application type and are
  reported as NOT_APPLICABLE for non-web deliverables rather than scored
- the absence of a heap-profiling workflow is documented and memory leaks
  are not claimed to be detected automatically
- Chrome DevTools MCP integration is described as optional, with documented
  non-MCP fallbacks; no imported Skill requires an MCP connection or an
  automatic installation
- the imported Skills do not override AGENTS.md or the branch, dependency,
  architecture, validation, provider, delivery, review, or
  session-resilience policies

## V2.13 Regression — Proportional Planning and SDD

Verify that planning is proportional, evidence-based, and integrated without
duplicating rules across files:

- `workflows/task-planning.md` exists at its canonical path, appears in the
  required-file inventory, and is invoked from Phase 9 of
  `workflows/task-to-code.md` without introducing a competing lifecycle
  order
- `skills/complexity-advisor/SKILL.md` classifies complexity and recommends
  a rigor level; it does not implement the planning procedure
- the planning procedure — levels and content per level — is defined once in
  `workflows/task-planning.md`, and the retained rigor level and artifact
  decision belong to `skills/task-context/SKILL.md`; no rule is duplicated
  across these files
- `agents/orchestrator.md` and `commands/task.md` delegate to the planning
  workflow instead of restating the levels or the approval gates
- a preliminary complexity estimate may be recorded as tentative during
  alignment; it does not fix the rigor level
- the definitive complexity classification (Phase 8) and the derived rigor
  level (Phase 9) are not fixed before sufficient evidence is available
- trivial and low tasks are not forced to produce a heavy specification, and
  a Markdown specification is not required by default
- medium and higher tasks define verifiable acceptance criteria, and complex
  or high-risk tasks define their expected behavior before implementing
- approval is triggered by the Human Decision Gates (risk, irreversible or
  high-impact operations, security or data, cost, scope, shared or
  production resources, project rules) and not by the classification label;
  a Critical task activates reinforced controls and requires approval before
  high-impact operations
- a durable planning artifact is created only for justified traceability,
  approval, or an explicit request; the default is session state, and no
  second specification of the code is produced
- `workflows/project-validation.md` and `skills/discovery/SKILL.md` remain
  unchanged and their existing behavior is preserved
- no duplicate canonical workflow, competing order, or second orchestrator
  was introduced

## V2.14 Regression — Formatter and Linter Setup and Execution

Verify that formatter and linter configuration and execution are concrete,
conflict-free, scoped, and reported honestly, with the procedure defined
once:

- `workflows/project-validation.md` defines the mandatory setup procedure
  once, in Phase 5, and remains the canonical procedure; no separate
  `workflows/tooling-setup.md` or second setup flow was created
- Phase 2 defines the tool lifecycle states — detected, configured,
  installed, executed, approved — and states explicitly that they are
  independent of the check-result statuses (`PASS`, `FAIL`,
  `SETUP_REQUIRED`, `BLOCKED`)
- Phase 5 detects before configuring, selects tools per the AGENTS.md
  defaults, configures for the detected framework and versions, prevents
  formatter/linter conflicts, scopes monorepo changes to the affected
  packages, and produces an explicit proposal when tooling is missing
- existing working configuration is preserved and a redundant second tool is
  not added
- the standard mandatory setup is authorized by policy without a redundant
  approval, while a paid tool, a non-standard dependency, or replacing a
  working configuration still requires a decision, and a real restriction is
  reported as BLOCKED rather than success
- `skills/stack-detection/SKILL.md` adds only detection and selection rules
  and delegates the procedure to `workflows/project-validation.md` instead of
  restating it
- `commands/validate.md` reports the per-tool lifecycle states separately
  from check statuses and reports proposals, blocks, and formatter or linter
  errors
- `AGENTS.md` remains the single source of the default tools per ecosystem
  and was not duplicated by the procedure
- a documentation-only repository such as this one is not given an
  application formatter, linter, `package.json`, or dependencies

## V2.15 Regression — Contracts and Implementation Recipes

Verify that implementation guidance and interface contracts are concrete,
non-duplicative, and proportional:

- `skills/implementation-recipes/SKILL.md` and
  `skills/api-contract-design/SKILL.md` exist at their canonical paths and
  are listed in the Skill inventory of this workflow
- `agents/orchestrator.md`, `workflows/task-to-code.md`, and `README.md`
  reference both Skills consistently, and none restates a competing
  lifecycle order or a duplicate decision policy
- both Skills defer architecture decisions to
  `skills/architecture-advisor/SKILL.md` instead of duplicating them
- `skills/implementation-recipes/SKILL.md` states the non-negotiable
  quality baseline: quality is never proportional to task size, and only
  planning rigor and test depth are proportional
- `AGENTS.md` carries the quality-not-proportional principle, and
  `skills/complexity-advisor/SKILL.md` clarifies that complexity and rigor
  govern planning and testing depth only, never code quality
- `skills/api-contract-design/SKILL.md` requires defining the contract
  before implementation when a boundary, interface, or integration changes,
  reuses the existing contract style, treats breaking changes as approval
  gates, and never exposes secrets in the contract
- no separate architecture-decisions Skill exists; the Decision Records
  guidance lives in `skills/architecture-advisor/SKILL.md` and records only
  significant, hard-to-reverse decisions without fabricated rationale
- `workflows/task-to-code.md` loads the contract Skill in the architecture
  phase when boundaries change and loads the implementation recipes in the
  implement phase
- `references/system-workflow-regression-matrix.md` includes the
  contract-before-implementation and quality-not-proportional scenarios
  (R-33, R-34)
- `integrations/skill-catalog.md` lists the internal authored Skills and
  does not present them as external import candidates
- no new command, package.json, dependency, or competing orchestrator was
  introduced

## V2.16 Regression — Database Engineering

Verify that database work is gated, safe, and non-duplicative:

- `skills/database-engineering/SKILL.md` exists at its canonical path and is
  listed in the Skill inventory of this workflow
- `agents/orchestrator.md`, `workflows/task-to-code.md`, and `README.md`
  reference the Skill consistently and state that it loads only when a task
  touches the schema, migrations, queries, indexes, or transactions
- `skills/database-engineering/SKILL.md` requires backward-compatible
  expand/contract migrations applied through the project's tooling, forbids
  editing applied migrations, and requires parameterized queries and
  N+1 awareness
- it requires explicit human approval for destructive or production data
  operations and never permits bypassing the migration tooling
- `skills/implementation-recipes/SKILL.md` defers schema, migration, query,
  and transaction changes to the database Skill instead of duplicating them
- the Skill does not duplicate the architecture decision policy owned by
  `skills/architecture-advisor/SKILL.md` or the contract policy owned by
  `skills/api-contract-design/SKILL.md`
- `references/system-workflow-regression-matrix.md` includes the database
  safety scenario (R-35)
- no new command, package.json, dependency, or competing orchestrator was
  introduced

## V2.17 Regression — Delivery Pipeline Authoring

Verify that pipeline authoring is an authorized mode, environment-aware, and
gate-enforced, without weakening the read-only assessment:

- `workflows/secure-delivery.md` describes an authoring mode distinct from
  the read-only assessment, and `commands/delivery-check.md` remains
  read-only and states that authoring happens inside a task
- the authoring mode builds one artifact and promotes it across environments
  with environment differences kept out of the artifact and no secret values
  in source
- it requires the same validation gates before every deployment and states
  that gates are never weakened to obtain a green pipeline
- production deployment requires an explicit approval gate, and each
  deployable environment has a stated rollback or recovery path
- a failing pipeline is met with a proposed fix or rollback, never a silent
  bypass, and no remote write, push, merge, or deploy occurs without explicit
  authorization
- `workflows/task-to-code.md`, `agents/orchestrator.md`, and `README.md`
  reference the authoring mode consistently and do not restate a competing
  delivery policy
- `workflows/secure-delivery.md` still does not recommend or introduce a CI
  platform, container base, or deployment dependency without justification
  and authorization
- `references/system-workflow-regression-matrix.md` includes the pipeline
  authoring scenario (R-36)
- no new command, skill, package.json, dependency, or competing orchestrator
  was introduced

## V2.18 Regression — Neutral State Root and Single Orchestrator

Verify the properties that are not covered by a later, more specific
check:

- the learning store uses the provider-neutral state root
  (`${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}`) with no
  client-specific path, and never writes inside a target project's source
  tree or the canonical repository's tracked files
- learnings never store credentials, tokens, secrets, or personal data, and
  identifiers that become path components are normalized against traversal
- promotion to the canonical layer is always a proposal that requires human
  approval; no instruction allows the system to edit the canonical
  repository automatically
- `workflows/session-resilience.md` writes checkpoints to the same
  provider-neutral state root and defines the handoff path in exactly one
  place; `AGENTS.md` and the commands reference it without duplicating the
  literal path
- no new command, skill, package.json, dependency, or competing orchestrator
  was introduced

The learning execution, the maturity ladder, and the context-efficiency
policy are verified in V2.38 and V2.30.

## V2.19 Regression — Standalone Estimation

Verify that estimation is standalone, honest, and non-invasive:

- `workflows/estimation.md` and `commands/estimate.md` exist at their
  canonical paths and are listed in the required-file inventory of this
  workflow
- estimation is described as a standalone activity that does not start the
  task lifecycle and does not modify application source, configuration,
  branches, or remote state
- estimation is scoped to a complete project from start to finish and states
  that individual tickets, stories, and tasks are out of scope; `AGENTS.md`,
  `agents/orchestrator.md`, `commands/estimate.md`, and `README.md` reflect
  that project scope consistently
- `commands/estimate.md` is a thin adapter that delegates to
  `workflows/estimation.md`; the accepted artifacts, the procedure, and the
  report are defined only in the workflow and are not restated in the command
- the document is written to the current working directory as
  `<slug>-estimation.<ext>` with a relative, cross-platform-safe path, and the
  command reports the saved location (relative and absolute)
- before writing, the workflow asks the user for the document language and
  format, offering a short list of suggested options, and uses exactly what
  the user states; the language is not inferred from the request; a choice
  the current client or model cannot produce is flagged and a supported
  alternative is offered instead of being produced silently (Markdown
  recommended, HTML dependency-free, PDF/DOCX only with an existing converter
  and no dependency added); the chosen language and format are reported
- the slug normalization rejects path separators, `.`/`..`, characters
  illegal on Windows, reserved device names, and case-collision risk, and
  never uses an absolute or machine-specific path
- effort is reported as a low/likely/high range per discipline with stated
  assumptions and a confidence level; monetary cost appears only when a rate
  is supplied, and is otherwise reported as not determined
- the workflow states that it never fabricates effort, duration, cost, or
  rates and never presents an unread artifact as a low estimate
- Phase 1 accepts an image supplied as a local path or as an attachment to the
  session, interpreted with `skills/design-analysis/SKILL.md`; when the client
  or model cannot read the image, it is reported as not estimated, never
  fabricated
- `AGENTS.md`, `agents/orchestrator.md`, `workflows/task-to-code.md`, and
  `README.md` reference the estimation workflow consistently and keep
  estimation-only requests free of branch preparation
- `references/system-workflow-regression-matrix.md` includes the estimation
  scenario (R-39)
- no package.json, dependency, or competing orchestrator was introduced

## V2.20 Regression — Internationalization and Mobile UI

Verify that localization is decided early and implemented correctly, and that
mobile interfaces are analyzed and built for mobile:

- `skills/internationalization/SKILL.md` exists at its canonical path and is
  listed in the required-file inventory of this workflow
- the Skill is scoped to user-facing work and is explicitly not loaded for
  work without user-facing text
- localization is settled early (required locales, RTL, and locale-aware
  formatting) in `workflows/task-to-code.md` (Phase 5), `agents/orchestrator.md`
  (§11), and `commands/task.md`, without duplicating a competing sequence
- the Skill requires user-facing strings to be externalized, forbids
  concatenating translated fragments, requires pluralization and locale-aware
  formatting, and covers direction/RTL, translation workflow, testing, and
  accessibility/SEO (`lang`, `dir`, `hreflang`)
- a new i18n library is subject to `workflows/dependency-assessment.md` and
  human authorization
- `skills/design-analysis/SKILL.md` provides a Mobile Analysis section and
  `skills/implementation-recipes/SKILL.md` provides concrete mobile
  implementation guidance, without introducing a separate mobile Skill
- `agents/orchestrator.md`, `integrations/skill-catalog.md`, `README.md`, and
  `references/official-documentation.md` reference internationalization and
  mobile consistently
- `references/system-workflow-regression-matrix.md` includes the
  internationalization and mobile scenarios (R-40, R-41)
- no new command, package.json, dependency, or competing orchestrator was
  introduced

## V2.21 Regression — Refactoring and Abstraction Justification

Verify that unsolicited refactoring and unnecessary fragmentation are prohibited:

- `skills/complexity-advisor/SKILL.md` defines the Refactoring and Abstraction
  Justification gate with thresholds (reuse, project size/complexity, required
  change, testability) and negative triggers (single use, no threshold,
  aesthetic, barrel without utility)
- `skills/architecture-advisor/SKILL.md` defines the Fragmentation Gate
  requiring the same justification for component/module extraction, barrels,
  and reorganization
- `AGENTS.md` scopes localized refactoring to what the task requires; unsolicited
  restructuring is out of scope and becomes an Engineering Insight
- `workflows/task-to-code.md` Phase 9 references the justification gate for
  proposed new components; Phase 10 enforces it and requires resolution of
  over-engineering findings (revert or justify with evidence), not waiver
- `skills/engineering-review/SKILL.md` classifies unnecessary abstraction/
  fragmentation as a finding that must be resolved, not a conditional PASS
- `references/system-workflow-regression-matrix.md` includes the scenario
  (R-42)
- no new command, package.json, dependency, or competing orchestrator was
  introduced

## V2.22 Regression — Code Audit

Verify that the code-creation audit is a standalone, read-only, non-duplicative workflow:

- `commands/code-audit.md` exists and delegates to `workflows/code-audit.md`
  without restating the procedure; it specifies the agent (`reviewer`) and
  read-only behavior
- `workflows/code-audit.md` defines the 14 applicability-gated dimensions,
  the senior/staff bar, the dimension matrix (PASS/WARN/FAIL/N/A), findings
  format (P0–P3, evidence, severity, confidence), and the verdict
- the workflow loads only applicable Skills and does not duplicate their
  checklists; it references `architecture-advisor`, `complexity-advisor`,
  `implementation-recipes`, `engineering-review`, `api-contract-design`,
  `database-engineering`, `best-practices`, `testing-strategy`,
  `test-driven-development`, `performance`, `core-web-vitals`,
  `accessibility`, `seo`, `internationalization`, `frontend-design`,
  `design-analysis`, `repository-strategy`
- it audits code only (no process compliance: branch policy, dependency
  gate, evidence honesty, git hygiene are out of scope)
- it is read-only: no edits, commits, pushes, or remote writes
- `agents/reviewer.md` and `agents/orchestrator.md` reference the code-audit
  workflow and intent
- `README.md` distinguishes `/code-audit` from `/system-check`, `/validate`,
  and `/review-pr`
- `references/system-workflow-regression-matrix.md` includes the scenario
  (R-43)
- no new Skill, agent, package.json, dependency, or competing orchestrator
  was introduced
- **default scope is the uncommitted change set; clean tree = NOTHING TO AUDIT; no fallback to whole project**

## V2.23 Regression — Folder Architecture, Dependency Direction, and Type/Contract Ownership

Verify that folder architecture and type/contract ownership are defined once, enforced in generation/review/audit, and reconciled with R-42:

- `skills/architecture-advisor/SKILL.md` defines the **Folder Architecture Map** (layers/modules, responsibility→folder, allowed dependency directions, contract/type location per module, co-location rules) and the **ownership principle** (type lives in the folder of the owning layer); includes the per-ecosystem matrix (TS/JS, Java/Kotlin, Python, C#/.NET, Go, Rust, PHP, Ruby, SQL) with typical folders, contract location, and import direction; explicitly states "not mandatory; preserve detected conventions."
- `skills/complexity-advisor/SKILL.md` reconciles R-42: shared contract extraction meets reuse threshold; single-consumer props/local types do not; language idiom overrides file-count aesthetics.
- `skills/implementation-recipes/SKILL.md` includes the **Folder Architecture and Contract Placement** section with the cross-ecosystem table and per-stack examples (React/Next.js feature `types.ts`, Java `domain`, Python `domain/contracts.py`, Go consumer-side interfaces).
- `skills/engineering-review/SKILL.md` checks folder conformance, dependency direction, and contract ownership in Architecture.
- `workflows/code-audit.md` Dimension 1 renamed to "Architecture & folder structure"; adds explicit **Folder Architecture check** (build map, map declarations, verify imports, flag layer-skipping/mixed-responsibility/global buckets) and **Type/Contract Ownership check** (language-aware declaration patterns, consumer mapping, follow declaration sites, flag shared-in-impl/import-from-impl/duplicates/cycles/idiom violations; exempt single-consumer props, Go consumer interfaces, private types, RBS); severity P1/P2/P3 with tags `TYPE_OWNERSHIP`/`FOLDER_ARCHITECTURE`; pre-existing labeling for declarations outside scope; no universal template imposed.
- `commands/code-audit.md` notes scope expansion for architecture/type ownership.
- `references/system-workflow-regression-matrix.md` includes R-44 with documented cases:
  - TS: shared type in `lib/payment-service.ts` imported by UI/hook → P2; `MethodOptionProps`/`Product` inline → OK.
  - Java: contract in `service/impl` imported by controller → defect; DTO in `domain` → OK.
  - Python: `TypedDict` in `services/stripe.py` imported by router → defect; `Protocol` in `domain/contracts.py` → OK.
  - Go: interface in consumer package → OK; forced in provider → defect anti-idiomatic.
  - Folder: UI → concrete repo; cross-feature private import; global `types.ts` dump.
- No new Skill, agent, command, package.json, dependency, or competing orchestrator introduced.
- No blanket "all inline types are defects" rule; no universal folder template imposed.
- **default scope is the uncommitted change set; checks operate only on modified files; declaration sites outside the diff are context only; pre-existing issues outside the diff are not reported; clean tree = NOTHING TO AUDIT**

## V2.24 Regression — Commit Generation with Project-Native Gates

Verify that commit generation is a standalone workflow with project-native
quality gates and atomic commits:

- `commands/commit.md` exists and delegates to `workflows/commit.md`
  without restating the procedure; it specifies the agent (`orchestrator`)
  and allows `--dry-run`, `--staged`, `--full`, and path arguments.
- `workflows/commit.md` defines the gate sequence: secrets scan → format
  check → lint → typecheck → unit tests → integration/e2e (`--full`) →
  build → security audit → AI sanity review.
- The workflow detects actual tooling from the project (package manager,
  scripts, lockfiles); it does not invent commands or install tools.
- Missing mandatory tooling (formatter, linter for software projects)
  reports `SETUP_REQUIRED` and blocks commits; no silent omission.
- Secrets scan blocks on common patterns; no external dependency required.
- AI sanity review is lightweight: obvious correctness/security defects
  only; no architecture, patterns, or over-engineering evaluation.
- Commits are atomic, English, Conventional Commits unless project
  convention differs; no AI mentions; no unverified test claims.
- Same gates for AI and human changes; no authorship-based bypass.
- Never pushes, deploys, or bypasses gates (`--no-verify` not supported).
- Clean tree = `NOTHING TO COMMIT`.
- Documentation-only repositories skip application gates.
- `skills/git-workflow/SKILL.md` references the workflow without
  duplicating rules.
- `agents/orchestrator.md` references the workflow in the Delivery phase.
- `README.md` distinguishes `/commit` from `/validate`, `/code-audit`,
  and `/generate-pr`.
- `references/system-workflow-regression-matrix.md` includes R-45.
- No new Skill, agent, package.json, dependency, or competing orchestrator
  introduced.

- No new Skill, agent, package.json, dependency, or competing orchestrator
  introduced.

## V2.26 Regression — Folder Architecture by Complexity, Dumping Ground Prohibition, and Reuse-Based Type Ownership

Verify that folder architecture is decided by complexity, `lib/`/`utils/` dumping grounds are prohibited, and type extraction is reuse-based:

- `skills/architecture-advisor/SKILL.md` replaces "detected and preserved" with **detect first, decide by complexity if absent**: small→layer-first (`src/api/<domain>`, `src/services/<domain>`, `src/server/<domain>`, `src/shared/`, `src/components/`, `src/hooks/`); medium→feature-first (`src/features/<feature>/{api,domain,server,components,hooks,contracts}` + `src/shared/`); large→feature modules with explicit public APIs and boundaries. **Universal responsibility rules**: `lib/`/`utils/` are generic utilities only; API has its own folder (`contracts.ts` + `client.ts`); services in `services/` or `domain/`; storage in `server/`/`infrastructure/`; i18n in `shared/` or top-level. A mixed `lib/` is an anti-pattern, not a convention to preserve. New code uses correct folders; existing moves only when touched (R-42).
- `skills/implementation-recipes/SKILL.md` adds the **complexity-based structure decision table** and an **API module recipe** (`contracts.ts` for schemas + inferred types, `client.ts` for validators/converters/fetch, `service.ts` for domain logic, `storage.ts` for persistence); i18n example moved to `src/shared/i18n/`.
- `skills/project-analysis/SKILL.md` adds **Dumping Ground Detection**: classifies `lib/` files; flags domain-specific code in `lib/` as "legacy; do not replicate".
- `skills/engineering-review/SKILL.md` adds dumping ground check, reuse-based type extraction (only when imported from another file), schema + inferred type pair stays in `contracts.ts`/`schemas.ts`, API file cohesion (contracts separate from client).
- `workflows/code-audit.md` Dimension 1: **Folder Architecture check** adds dumping ground detection; **Type/Contract Ownership check** is reuse-based (P2 when imported from another file, WARN when only exported); **File Cohesion check** adds API-specific pattern (schemas + validators + client in one file → P2).
- `references/system-workflow-regression-matrix.md` includes R-47 with documented cases:
  - Counterexample: a `lib/` containing i18n, payments service, storage, API contracts, and API client mixed → P2 `FOLDER_ARCHITECTURE` + `TYPE_OWNERSHIP`.
  - Correct for a small app: layer-first `src/api/payments/{contracts,client}`, `src/services/payments`, `src/server/payments/storage`, `src/shared/i18n/`.
- No new Skill, agent, package.json, dependency, or competing orchestrator introduced.
- `/commit` does **not** block on architecture/type placement; behaves like Husky (objective gates only). Architecture is in code-audit/review/task-to-code.

## V2.27 Regression — Commit Approval Gate

Verify that no commit is created without explicit human approval:

- `AGENTS.md` includes "Creating a commit" in Human Decision Gates and a **Commit Approval Policy**: never run `git commit`/`--amend` without explicit approval of the proposed scope and message; invoking `/commit` or `/task` authorizes preparing a proposal, not creating commits; a general request to implement is not approval; a changed scope or message requires re-approval.
- `workflows/commit.md`: Phase 13 is a **Proposal and Approval Gate**; Phase 14 creates only approved commits; Phase 15 reports approval status; a dry run stops at the proposal.
- `commands/commit.md`: states that the command prepares and validates a proposal and does not create commits by itself; includes the approval gate.
- `skills/git-workflow/SKILL.md`: "Before commit" asks for explicit approval and creates only after an affirmative answer.
- `agents/orchestrator.md` and `workflows/task-to-code.md`: delivery requires explicit approval and rejects command invocation or implementation request as commit approval.
- `references/system-workflow-regression-matrix.md` includes R-48.
- No commit is created during validation; the validation itself is read-only.

## V2.28 Regression — Primary Unit per File, Unit Folders, and Co-located Tests

Verify that file/unit organization is applied according to ecosystem support, not as a universal rule:

- `skills/architecture-advisor/SKILL.md`: **Primary Unit per File and Stack Applicability** defines the rule, the dumping-ground anti-pattern, the Angular exception, and optional barrels; **Unit Folders and Co-located Tests** defines the sibling anti-pattern, ecosystem-appropriate layouts, R-42 scope, and the official documentation reference.
- `skills/implementation-recipes/SKILL.md`: the canonical applicability matrix covers React/Next.js, Angular, Vue, Svelte, Solid, Java, Kotlin, Python, Go, Rust, C#/.NET, PHP, and Ruby, with the primary-unit rule and test location per ecosystem. Barrels are optional and not part of the default layout.
- `skills/project-analysis/SKILL.md`: Folder Architecture Map and Internal Module Layout Detection report primary-unit violations, unit grouping, test co-location, and barrel justification where supported.
- `skills/engineering-review/SKILL.md`: review flags multi-unit files and `X`, `X.test`, and `X/` siblings only where the ecosystem supports the rule; it does not impose React/Angular conventions on Solid, Kotlin, Python, Go, Rust, C#/.NET, or Ruby.
- `workflows/code-audit.md`: Dimension 1 includes the **Primary unit per file check**, **Unit folder and test co-location check**, and **Barrel check**, with `FILE_COHESION`/`FOLDER_ARCHITECTURE` tagging and P2/P3 severity.
- `workflows/task-to-code.md` Phase 8: the proposal must include a File Responsibility Map, respect the applicability matrix, and use a unit folder with a co-located test where the ecosystem supports it.
- `references/official-documentation.md`: **Project Structure and Test Co-location** records what each source supports: React/Next.js do not mandate one component per file; Airbnb/ESLint do; Angular supports “one concept per file” with an exception; Vue/Svelte are inherently one component per file; Solid and Kotlin allow multiple related declarations; Java public API types are conventionally one per file, and PHP autoloaded classes are one per file; the remaining ecosystems have no one-unit-per-file mandate.
- `references/system-workflow-regression-matrix.md` includes R-46 and R-49.
- No demo-specific assertion is part of this validation; the demo and other reference paths are read-only.

## V2.29 Regression — Target Project and Evidence-Only Paths

Verify that scope is explicit and reference paths are never modified:

- `AGENTS.md`: **Scope and Evidence Boundaries** states that the target project is the only writable scope, evidence-only paths are read-only without explicit authorization, reference examples are not implicit tasks, and out-of-target changes require a stop-and-ask decision.
- `workflows/task-to-code.md`: Phase 1 records the target project root and evidence-only paths; Phase 2 preserves the boundary; Phase 3 treats design artifacts as evidence; Phase 10 forbids modifying evidence-only paths, reference projects, design artifacts, or external repositories.
- `agents/orchestrator.md`: **Scope and Evidence Boundaries** requires identifying the target and evidence-only paths before delegating and passing the boundary to every delegated agent.
- `skills/discovery/SKILL.md`: discovery identifies the target project and evidence-only paths and reports them in its output.
- `references/system-workflow-regression-matrix.md` includes R-50.
- A path supplied only as evidence, comparison, or context is never modified without explicit authorization, even when it demonstrates a real defect.

## V2.30 Regression — Minimum Sufficient Context and Token Efficiency

Verify that context efficiency is bounded, measured, and subordinate to
correctness:

- `AGENTS.md`: **Minimum Sufficient Context** requires a Context Manifest,
  soft exploration budgets, stop rules, measurement, and no correctness
  tradeoff.
- `workflows/session-resilience.md`: **Token and Context Efficiency** defines
  the Context Manifest, soft exploration budgets for R1–R4, retrieval rules,
  stop rules, and measurement; budgets exclude implementation and validation.
- `workflows/task-to-code.md`: Phase 1 records the manifest and provisional
  rigor; Phase 2 refines it without unrelated paths; Phase 9 includes the
  final budget; Phase 10 enforces the stop rules and records expansions.
- `skills/task-context/SKILL.md`: the working interpretation contains the
  Context Manifest and the rule not to broaden it without evidence.
- `references/token-efficiency.md`: defines metrics, baseline and
  calibration, optional read-only queries, and the no-hard-cap principle.
- `references/system-workflow-regression-matrix.md` includes R-51.
- The policy applies to exploration only; implementation, tests, formatting,
  build, validation, and security checks remain mandatory.
- No hard token cap, compaction plugin, or reduction of required evidence is
  introduced.

## V2.31 Regression — Structural Compliance Enforcement

Verify that the structural rules are enforced mechanically and not only
documented in prose:

- `workflows/output-compliance.md` exists at its canonical path and is listed
  in the required-file inventory of this workflow.
- It defines deterministic checks with executable detection patterns and
  `file:line` evidence: `FUNC-001`, `TYPE-001`, `TYPE-002`, `FILE-001`.
- It is read-only and never modifies code.
- `AGENTS.md` **Implementation and Validation** makes the check mandatory
  before a task is reported as implemented and states that a `FAIL` is a
  defect of the task, not a finding to defer.
- `workflows/task-to-code.md` Phase 10 runs it and forbids reporting
  completion with an unresolved `FAIL`.
- `workflows/project-validation.md` includes it as a mandatory check, and
  `commands/validate.md` runs it.
- `workflows/code-audit.md` requires the executed detection as evidence for
  `TS_FUNCTION_STYLE` and `TYPE_OWNERSHIP` findings.
- `references/linter-rules.md` exists at its canonical path and is listed in
  the required-file inventory of this workflow. It provides the copy-ready
  ESLint flat configuration, the local `structural/no-mixed-types-and-runtime`,
  `structural/no-pass-through-re-export`, and `structural/no-unit-folder`
  rules, and the contract consumer script.
- The local rules are version-agnostic: they resolve the file name and the
  source through the API available in ESLint 8, 9, and 10, so the canonical
  artifact installs on the major version the target project uses.
- Every structural rule of the feature-driven architecture is **installed**,
  not only checked: the unit-folder rule in particular is an ESLint rule, so
  it fails the lint the project already runs instead of depending on a
  procedure the agent may skip.
- `AGENTS.md` **Formatting and Linting Policy** requires the structural
  compliance rules for JavaScript and TypeScript, requires the type-per-file
  rule in every ecosystem whose linter can express it, and states that a
  linter without them is not a complete setup.
- `workflows/project-validation.md` Phase 5 installs the structural rules
  when configuring the linter, and Phase 7 verifies that the resolved
  configuration actually contains them, reporting a missing rule as
  `SETUP_REQUIRED` even when the lint command exits successfully. The
  verification covers every supported ecosystem, and states explicitly when
  a rule is covered by verification rather than by an installed rule.
- `references/system-workflow-regression-matrix.md` includes R-52.
- The checks cover TypeScript/JavaScript, Python, Java/Kotlin, Go, Rust,
  C#/.NET, PHP, and Ruby.

## V2.32 Regression — Mandatory Rule Reachability

Verify that every mandatory rule is reachable from every surface an agent
may read, and that the canonical wording lives in exactly one place.

`ai-engineering` is the single source of truth. A rule that exists only in
an optional Skill is invisible to an agent that never loads it. A rule
restated in many files drifts apart. The registry in
`references/mandatory-rules.md` records the canonical wording, the anchor,
and the required surfaces of every mandatory rule.

### Reachability check

For each row of the registry, verify the anchor is present on every required
surface. The comparison is normalized — lowercase, `-` and space treated as
equivalent, and whitespace collapsed — so that a line wrap or a hyphen does
not produce a false failure.

The check uses only `grep`, `sed`, and `tr`, so it runs on any client and in
any environment without an interpreter dependency:

```sh
normalize() {
  tr '\n' ' ' | tr 'A-Z' 'a-z' | tr '-' ' ' | tr -s ' ' ' '
}

trim() {
  sed 's/`//g; s/^[[:space:]]*//; s/[[:space:]]*$//'
}

grep -E '^\| M-[0-9]+ \|' references/mandatory-rules.md |
while IFS='|' read -r _ id _ _ anchor surfaces; do
  id="$(printf '%s' "$id" | trim)"
  anchor="$(printf '%s' "$anchor" | trim)"
  normalized_anchor="$(printf '%s' "$anchor" | normalize)"
  missing=""


A rule whose canonical wording is missing from `AGENTS.md` is a `FAIL`: it
would not reach every client.

### Single-source check

Verify the canonical wording is not copied into other files. The anchor of a
rule whose canonical wording is in `AGENTS.md` must appear in `AGENTS.md`,
and the surfaces listed in the registry must reference the rule rather than
restate its normative wording:

```sh
# The canonical wording of an always-active rule lives in AGENTS.md.
# Count the files that carry the anchor: more than the canonical file plus
# the registry means a copy that can drift.
grep -rlF "arrow functions" --include="*.md" . | grep -v node_modules
```

Review the result. A surface that restates the normative wording instead of
referencing `AGENTS.md` is a drift risk: convert it into a reference.

### Trigger check

Verify that every Skill description names the concrete triggers that make an
agent load it. A description that only states an abstract purpose does not
match a concrete task and the Skill is never loaded:

```sh
for skill in skills/*/SKILL.md; do
  printf '%s -> ' "$skill"
  sed -n '3p' "$skill" | cut -c14-
done
```

A description that omits the nouns of the task it supports is a `FAIL`.

### Consistency checks

- `references/mandatory-rules.md` exists at its canonical path and is listed
  in the required-file inventory of this workflow.
- Every registry row has an ID, a canonical wording, an anchor, and at least
  one required surface.
- Every required surface exists in the repository.
- The canonical wording of every always-active rule is in `AGENTS.md`.
- `references/system-workflow-regression-matrix.md` includes R-53.

## V2.33 Regression — New Project Initialization

Verify that a new project follows one path and asks for the architectural
style before any structure is created:

- `workflows/project-initialization.md` exists at its canonical path and is
  listed in the required-file inventory of this workflow.
- It recovers the source analysis the task points to and reuses it instead
  of re-deriving requirements; a requirement the analysis does not cover is
  treated as unknown, not implied.
- It confirms the detected stack against `references/official-documentation.md`.
- Phase 3 presents the applicable architectural styles with their trade-offs
  and asks the user which one to use. A recommendation is not a decision, and
  the repository structure is not created before the answer.
- Phase 5 installs the formatter, the linter, and the structural compliance
  rules from `references/linter-rules.md`, with reproducible commands, plus a
  test runner and a first test.
- `AGENTS.md` **Human Decision Gates** lists the architectural style of a new
  project as a decision that requires the user.
- `workflows/task-to-code.md` Phase 4 routes a new project to the
  initialization workflow instead of creating it inline, and its Phase 8 gate
  covers the new-project case.
- `skills/architecture-advisor/SKILL.md` covers the new-project case in its
  decision gate.
- `agents/orchestrator.md` and `commands/task.md` route a new project to the
  initialization workflow.
- `workflows/estimation.md` defines how the estimation document is reused as
  the input of implementation.
- `references/project-startup-decisions.md` exists at its canonical path and
  is listed in the required-file inventory of this workflow. It covers the
  universal decisions and the decisions of a backend, a frontend, a library
  or CLI, and a mobile application, each with a recommended default, the
  condition that requires the user, and where the answer comes from.
- Each decision is resolved in order: answered by the source analysis,
  governed by a detected convention or policy, has a defensible default, or
  materially changes the outcome with no default. The user is asked once,
  grouped by theme, only for the last category, with its recommendation and
  trade-off. Nothing that the source analysis already answers is re-asked,
  and no decision is left implicit in a default that was never stated.
- `references/project-creation-order.md` exists at its canonical path and is
  listed in the required-file inventory of this workflow. It defines the
  universal creation steps, the backend creation steps, the frontend,
  library, and mobile creation steps, and the complete creation checklist.
- `workflows/project-initialization.md` Phase 4 creates the project step by
  step, verifies each step before the next one starts, applies the growth
  rule, and copies the creation checklist into the report.
- Each creation step states what it produces, how to verify it, and where
  the project grows next, so the structure has an extension point instead of
  a parallel structure.
- `references/system-workflow-regression-matrix.md` includes R-54.

## V2.34 Regression — End-to-End Audit of an Adjustment

Verify that an adjustment can be applied and proved correct in one command,
driven only by its text:

- `commands/audit-e2e.md` exists at its canonical path and is listed in the
  required-file inventory of this workflow. It delegates to
  `workflows/system-audit.md` and copies no block from it.
- Every command in `commands/` has its own description in the README, and
  the structural audit verifies it. A command without a README description
  is a `FAIL`: the catalog a reader consults is incomplete.
- `workflows/system-audit.md` exists at its canonical path and is listed in
  the required-file inventory of this workflow.
- The command accepts the adjustment text as its only input. It defines no
  mode, no flag, and no implicit scope, and it never derives the scope from
  the Git change set.
- The adjustment text determines the area, the rules in scope, the required
  checks, and the artifacts to build, through the area mapping in the
  workflow.
- The command implements the adjustment following `workflows/task-to-code.md`,
  then audits it in the same run.
- The structural audit always runs: reference integrity, index and
  inventory, numbering, rule reachability, cross-client reach in
  `AGENTS.md`, duplication and contradiction, and coverage.
- The behavioral audit builds a real artifact per affected ecosystem or
  project type in a temporary directory outside the repository, installs the
  canonical artifacts, verifies the compliant artifact passes, injects the
  violation the adjustment was meant to prevent, verifies it is detected
  with `file:line` evidence, fixes it, re-verifies, and cleans the
  directory.
- A check that could not be executed is reported as `NOT VERIFIED` with the
  reason, never as passed.
- A defect found in the system's own artifact is reported as a defect with
  the evidence, and the cycle is repeated after the correction.
- The command presents the commit text and does not create the commit.
- `references/system-workflow-regression-matrix.md` includes R-55.

## V2.35 Regression — Progressive Project Creation

Verify that a new project is created progressively, within an agreed scope,
with user-controlled stopping points:

- `commands/create-project.md` exists at its canonical path and is listed in
  the required-file inventory of this workflow.
- `AGENTS.md` states that a new project is created progressively, that the
  scope catalog is agreed first and is a ceiling, that each step ends in a
  checkpoint that reports the next step and waits, and that stopping is a
  normal outcome.
- `AGENTS.md` states that creating a project from zero requires the
  explicit `create-project` command, and that without it the agent asks
  instead of creating one.
- `references/project-creation-order.md` defines the scope catalog, the
  dependency budget, the one-slice-at-a-time rule, the checkpoint
  semantics, and the progress tracker with its states.
- `workflows/project-initialization.md` Phase 3 agrees the scope catalog,
  and Phase 4 executes one step at a time, verifies it, updates the
  tracker, reports the next suggested step, and waits.
- The remaining steps are never generated to finish the project, and the
  scope is never exceeded without asking.
- `references/system-workflow-regression-matrix.md` includes R-56.

## V2.36 Regression — Target Version and Official Documentation

Verify that the target version is settled before the structure is created,
and that the official documentation mandate is satisfiable:

- `AGENTS.md` states that a new project settles its target version before its
  structure is created, that the source of truth is the source analysis,
  then the official documentation, then a stated default, that a version is
  never chosen because it is installed, and that the official documentation
  is consulted for that version with the consultation recorded.
- `workflows/project-initialization.md` Phase 2 confirms the target version,
  consults the official documentation for it, compares it with the
  environment, and reports a mismatch instead of silently resolving it.
- `references/project-startup-decisions.md` lists the target version as a
  universal decision, and states the version and documentation rules.
- `references/project-creation-order.md` states that the project is created
  for the confirmed target version, and every skeleton step of every project
  type produces an artifact pinned to it and verifies the reported version.
- `skills/stack-detection/SKILL.md` separates the version found from the
  target version, and records the source of truth for it.
- `references/official-documentation.md` contains a section for every
  supported ecosystem, so the consultation mandate is satisfiable for all of
  them.
- `workflows/system-audit.md` requires the behavioral artifact to be built
  for the confirmed target version, and to report as `NOT VERIFIED` any
  check that could not run against it.
- `references/system-workflow-regression-matrix.md` includes R-57.

## V2.37 Regression — Documentation Freshness

Verify that consulted documentation is current and that a stale source is
visible:

- `AGENTS.md` states that the version guide is consulted first, that a
  version-agnostic source is preferred, that a pinned source is registered
  with its version and a review rule, and that the consulted version and the
  consultation date are recorded.
- `references/official-documentation.md` contains a **Freshness** policy that
  classifies each source as version-agnostic or version-pinned, and a
  **Version-Pinned Sources** table with the version and the review rule of
  every pinned source.
- Every URL in the official documentation index that contains a version
  number is registered in the pinned table. An unregistered pinned source is
  a `FAIL`, because it can age without anyone noticing.
- `workflows/system-audit.md` runs that check, and resolves every external
  URL.
- `skills/stack-detection/SKILL.md` consults the version guide before the
  ecosystem pages, prefers a version-agnostic source, and records the
  version and the date.
- `workflows/project-initialization.md` Phase 2 reports a source that does
  not cover the target version instead of applying it silently.
- `references/system-workflow-regression-matrix.md` includes R-58.

## V2.38 Regression — Self-Learning Execution

Verify that the system learns from its mistakes and does not repeat them:

- `workflows/self-learning.md` Phase 1 is an **Apply** phase: it reads the
  store for the project scope and the global scope before acting, applies
  every relevant entry, and reports the applied ones.
- Repeating a lesson the store already records is defined as a defect of
  the task, not bad luck, and the task is not complete until it is
  corrected.
- An entry the evidence contradicts is not obeyed: the evidence wins and the
  entry is reported as superseded.
- An empty or missing store is reported and never blocks a task.
- `AGENTS.md` states the consume rule, and `workflows/task-to-code.md`
  Phase 1 reads the learnings and Phase 12 captures a lesson when there is
  a reusable signal.
- `commands/task.md` requires reading the learnings before acting.
- The maturity ladder is Junior, Semi Senior, Senior, Staff, is derived from
  evidence, is stored with its date, and is reported in the task report.
- The execution floor is Staff for every task; the level changes the
  direction and review the work needs, never the quality it gets.
- The store holds no credentials, tokens, secrets, or personal data, and
  never writes inside a target project or the canonical repository.
- `references/system-workflow-regression-matrix.md` includes R-59 and R-60.

### Store structure check

```sh
root="${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}"
[ -d "$root" ] && echo "store present: $root" || echo "store absent (allowed, never blocks)"
[ -d "$root/learnings" ] && echo "learnings scope present" || echo "no learnings yet (allowed)"
[ -d "$root/maturity" ] && echo "maturity scope present" || echo "no maturity record yet (allowed)"
```

A missing store or scope is reported, never treated as a failure. A learning
entry that contains a credential, a token, or personal data is a `FAIL`.

## V2.39 Regression — Portability

Verify that the system works on every client and operating system:

- `references/portability.md` exists at its canonical path and is listed
  in the required-file inventory of this workflow. It defines the contract:
  the repository is the source of truth, the knowledge is client-neutral, the
  procedures assume a POSIX shell, and no operating-system assumption is
  baked into the rules.
- `AGENTS.md` states the portability rule: a client without Skills still
  receives every mandatory rule, the procedures degrade to `NOT VERIFIED`
  with the reason when a tool is missing, and no operating-system path or
  client directory is hardcoded.
- Every workflow is reachable **without** the `commands/` adapters, so a
  client without slash commands is never blocked.
- No shared procedure hardcodes an operating-system path or a client
  configuration directory, and none depends on a macOS-only utility.
- A check that could not run because a tool is missing is reported as
  `NOT VERIFIED` with the tool named, never as passed.
- The setup instructions cover macOS, Linux, and Windows, and state that
  linking is a convenience rather than a requirement.
- `references/system-workflow-regression-matrix.md` includes R-61.

## V2.40 Regression — Feature-Driven Structure

Verify that a feature-driven project is organized by capability, with one
folder per unit and a public API per feature:

- `AGENTS.md` states the rule: the code is grouped by business capability,
  each unit has its own folder with its implementation, its test, and its
  types, each feature exposes a public API, a feature never imports the
  internals of another feature, and a unit with a second consumer is
  promoted to `shared/` rather than copied.
- `skills/architecture-advisor/SKILL.md` defines **Feature-Driven Folder
  Architecture**: the target tree, the folder per unit, the public API of a
  feature, the golden dependency rule, the promotion rule, and the decision
  that the owner, the folder, and the public surface are recorded before
  the files are created.
- The convenience barrel and the public API of a feature are distinguished,
  so the optionality of one does not weaken the requirement of the other.
- `skills/implementation-recipes/SKILL.md` gives the feature-first structure
  with one folder per unit and a public API per feature.
- `references/project-creation-order.md` requires the folder decision before
  the files are created, and states that a flat feature folder is the
  symptom of skipping it.
- `workflows/output-compliance.md` checks STRUCT-001 (one folder per unit),
  STRUCT-002 (no import that reaches past a feature's public API), and
  STRUCT-003 (every feature exposes a public API).
- `workflows/code-audit.md` includes the feature-driven structure check with
  the golden dependency rule.
- `references/system-workflow-regression-matrix.md` includes R-62.

## V2.41 Regression — Folder Architecture as a Decision

Verify that the folder structure is decided, and that every style has a
concrete tree for the detected ecosystem:

- `references/folder-architectures.md` exists at its canonical path and is
  listed in the required-file inventory of this workflow. It defines the
  concrete tree of every style, the rules that hold in every tree, the
  per-ecosystem translation of the unit of organization, and the concrete
  variants of the most distinct combinations.
- Every style listed in `skills/architecture-advisor/SKILL.md` has a tree in
  that reference, and the reference is where the trees live, so they do not
  diverge.
- `AGENTS.md` states that the folder structure is a decision, asked with the
  architectural style and settled before any file is created, and that a
  style presented only as a one-line hint is not a choice the user can make.
- `references/project-startup-decisions.md` lists the folder architecture as
  a decision of the startup decision set.
- `workflows/project-initialization.md` presents the concrete trees with the
  question, asks about the architecture **and** the folders together, and
  never filters Ports and Adapters out of the options.
- `references/project-creation-order.md` takes the tree from the same
  reference.
- `workflows/output-compliance.md` checks STRUCT-004: the folders that exist
  are the folders that were agreed.
- The per-ecosystem translation covers every ecosystem of the applicability
  matrix: TypeScript and JavaScript, React and Next.js, Angular, Vue, Svelte,
   Solid, Java and Kotlin, Python, Go, Rust, C# and .NET, PHP, and Ruby.
- `references/system-workflow-regression-matrix.md` includes R-63.

## V2.42 Regression — Topology Decision and the Monorepo Counter-Case

Verify that the repository topology is decided with a reason, and that an
unjustified monorepo is declined:

- `AGENTS.md` states that a monorepo is justified by a concrete benefit —
  shared code or types, atomic changes across projects, common tooling, or
  consistent validation — and that one team and a shared release train are
  not that benefit.
- `skills/repository-strategy/SKILL.md` states the counter-case: when the
  applications share no code and no change spans them, the benefit a
  monorepo exists for is absent, a coordinated release is a pipeline
  concern, and the simpler topology is separate repositories.
- `workflows/project-initialization.md` records the topology decision and
  its reason as part of the startup decision set.
- `workflows/output-compliance.md` checks STRUCT-005: a repository with
  several packages and no recorded concrete benefit is a `FAIL`.
- `references/system-workflow-regression-matrix.md` includes R-64.

## V2.43 Regression — Source Grounding

Verify that every structural rule and folder tree names its source:

- `AGENTS.md` requires every structural rule and every folder tree to name
  its source and the kind of source, and forbids presenting a community
  methodology as a framework standard or extrapolating one ecosystem's rule
  to another.
- `references/folder-architectures.md` states the authority of each tree:
  Fowler's Presentation-Domain-Data Layering, Feature-Sliced Design,
  Cockburn's original ports and adapters description, the bounded context for
  DDD, and Spring Modulith for a feature-driven Java module.
- `references/folder-architectures.md` records the source of the rule for
  every ecosystem in the per-ecosystem table.
- `references/official-documentation.md` distinguishes an ecosystem official
  rule from a recognized community methodology, and records the verified
  source for each: the Angular Style Guide for "one concept per file", the
  Vue single-file component specification, Solid for multiple components per
  file, the Go specification for a package per directory, PSR-4 for one
  class per file, Spring Modulith for the provided interface, and the
  Feature-Sliced Design layers and public API references.
- Every URL in that section resolves.
- `references/system-workflow-regression-matrix.md` includes R-65.

## V2.44 Regression — Architecture Matrix

Verify that every architectural style works in every supported ecosystem:

- `references/folder-architectures.md` contains the **Canonical Matrix** with
  the top-level folders, the public API, and the core dependency direction of
  each style, and states that the matrix is machine-checkable and how it is
  verified.
- The matrix covers the six styles — layered, feature-driven, hexagonal, DDD,
  modular monolith, and event-driven — and the fifteen supported ecosystems.
- A checker derives the expected layout from the matrix, so the trees and the
  checks cannot drift apart.
- For every combination it verifies: the folders of the style exist, a unit
  owns its implementation and keeps its test where the ecosystem places it,
  the module exposes its public API, and the dependency direction of the style
  holds.
- Where the ecosystem has a toolchain available, the generated project is
  compiled, built, imported, or linted. An ecosystem without a local toolchain
  is verified structurally and reported as not verified for execution, never
  as executed.
- `references/system-workflow-regression-matrix.md` includes R-66.
