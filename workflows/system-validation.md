# System Validation Workflow

## Purpose

Audit the consistency of the AI Engineering System and its client
integration without modifying files or remote services.

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
- commands/pr.md
- commands/review-pr.md
- commands/system-check.md
- commands/validate.md
- commands/checkpoint.md
- commands/resume-work.md
- commands/provider-check.md
- commands/delivery-check.md
- contracts/task-provider.md
- contracts/design-provider.md
- contracts/git-provider.md
- integrations/access-strategy.md
- integrations/provider-policy.md
- integrations/provider-matrix.md
- integrations/external-skill-registry.md
- integrations/skill-catalog.md
- references/official-documentation.md
- workflows/task-to-code.md
- workflows/pull-request-review.md
- workflows/system-validation.md
- workflows/project-validation.md
- workflows/session-resilience.md
- workflows/provider-integration-proof.md
- workflows/secure-delivery.md
- workflows/dependency-assessment.md

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
- testing-strategy
- engineering-review
- git-workflow
- repository-strategy
- pull-request
- pull-request-review
- review-memory

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

Possible statuses:

- VERIFIED
- NOT INSTALLED
- LICENSE MISSING
- SOURCE REVISION MISSING
- SUPPORTING FILE MISSING
- POLICY CONFLICT
- NOT VERIFIED

Never report an external Skill as VERIFIED based only on its registry entry.

Report E2E readiness as BLOCKED while a required external Skill is missing
or its license and source cannot be verified.

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
- checkpoints are written to a user-level handoff directory outside the
  target project's source tree
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
- no file in the core execution path defines a competing ordering of the
  lifecycle

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
- Provider configuration status:
- Delivery and CI/CD state:
- Dependency decision state:
- License status:
- External Skill registry:
- External Skills installed:
- External Skill licenses:
- Upstream commit verification:
- External Skill policy conflicts:
- E2E readiness:

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
