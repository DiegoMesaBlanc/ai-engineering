# AI Engineering System — Shared Instructions

This is the canonical shared instruction entry point for the AI Engineering
System.

## Component Responsibilities

- AGENTS.md: shared engineering principles and mandatory guardrails.
- CLAUDE.md: Claude Code compatibility entry point.
- skills/: reusable engineering knowledge, loaded on demand.
- workflows/: provider-neutral engineering procedures.
- contracts/: provider-neutral integration definitions.
- commands/ and agents/: client-specific adapters; currently configured
  primarily for OpenCode.
- workflows/: provider-neutral engineering procedures. A workflow is the
  single canonical path of a procedure; a command is only its adapter.

Do not duplicate Skills, workflows, or architecture definitions.

## Engineering Principles

- Inspect before assuming.
- Understand the task before implementation.
- Detect the actual stack, repository topology, and architecture.
- Reuse existing project abstractions when appropriate.
- Prefer the simplest solution that correctly solves the task.
- Treat code and engineering quality as non-negotiable: every task,
  however small, follows professional best practices, clean code, typing,
  error handling, security, and appropriate tests. Only process rigor and
  test depth are proportional to the task; implementation quality never is.
- Evaluate monorepos, microfrontends, Atomic Design, and design patterns
  only when evidence justifies them. A monorepo is justified by a concrete
  benefit — shared code or types, atomic changes across projects, common
  tooling, or consistent validation. One team and a shared release train are
  not that benefit: when the projects share no code and no change spans
  them, separate repositories are the simpler topology, and the decision is
  recorded with the reason.
- Require human approval for significant architectural changes.
- Load only relevant Skills and supporting references.
- Write TypeScript and JavaScript with **arrow functions**. Use the
  `function` keyword only when it is genuinely required (a generator, a
  `this` binding that arrow syntax cannot provide, the `arguments` object,
  or `new.target`) and state the reason in a comment.
- Keep interfaces, types, enums, and contracts in a **dedicated file** in
  the folder of the layer or module that owns them, never in the same file
  as runtime behavior. One type per file is the default; group several
  types in one file only when they are consumed by the same component,
  service, or logical unit.
- These structural rules take **precedence over a detected project
  convention that violates them**. A convention that produces `function`
  declarations, or files that mix type declarations with runtime behavior,
  is a defect to correct in the code you write — not a convention to
  preserve and copy.
- A module that consumes a contract does not re-export it. The owning file
  is the single public path of the contract. A **pass-through re-export** —
  a component re-exporting the props it imports from `./types`, a service
  re-exporting the contracts it imports from `./contracts` — gives one
  contract two public paths and is removed. This applies to every supported
  ecosystem, in whatever form it forwards a contract: an `export … from`,
  a type alias, a `pub use`, or an `__all__` entry that only re-exports.
  A barrel that aggregates several modules may re-export them.
- Place code by responsibility; `lib/` and `utils/` are not dumping grounds for domain services, API clients, contracts, or storage. Folder structure follows project complexity, not a universal template.
- The **folder structure is a decision**, asked with the architectural style
  and settled before any file is created. The trees of every style, per
  ecosystem, are defined in `references/folder-architectures.md`; the user
  sees the folders before they exist. A style presented only as a one-line
  hint is not a choice the user can make.
- In a **feature-driven architecture**, group the code by business capability
  (`app/`, `shared/`, `features/<feature>/`), give **each unit its own folder**
  with its implementation and its types, and let each feature expose a
  **public API** (`index.ts`). The test joins the unit folder **only where
  the ecosystem co-locates tests** (TypeScript, JavaScript, Angular, Vue,
  Go, Python); where the ecosystem mirrors the test tree (Java, Kotlin,
  C#/.NET, PHP, Ruby) the test belongs to that mirrored tree, not to the unit
  folder. A feature imports from `app/` and `shared/` and **never the
  internals of another feature**; cross-feature interaction goes through the
  other feature's public API or the global state. A unit that needs a second
  consumer is promoted to `shared/`, not copied. The system decides the
  owner, the folder, and the public surface of every unit before creating
  the files.
- Ground stack-specific structure and test conventions in that stack's
  official documentation or a recognized authoritative style guide. Never
  extrapolate one ecosystem's rule to another. Every structural rule and
  every folder tree names its source, and the kind of source is stated:
  an ecosystem official rule, a recognized community methodology, or a policy
  of this system. A community methodology is never presented as a framework
  standard, and a framework's own convention wins when the two differ.

## Scope and Evidence Boundaries

The target project is the only writable scope for a task.

- A path, repository, folder, artifact, or example supplied as evidence,
  reference, comparison, or context is read-only unless the user explicitly
  authorizes changes to it.
- Before modifying code, identify the target project and any evidence-only
  paths in the task context or scope manifest.
- Never treat a reference path as an implicit task, and never "fix" an
  example merely because it demonstrates a problem.
- If a change outside the target project appears necessary, stop and ask
  for explicit authorization before touching it.
- This applies to every client, agent, workflow, and automation.

## Task Branch Policy

Before modifying application code for a task:

1. Inspect Git status and the appropriate base branch.
2. Understand the task and available design context.
3. Ask the user for the exact branch name if it was not provided.
4. Validate the name and check for existing branches.
5. Preserve uncommitted work.
6. Create or prepare the requested branch/worktree.
7. Verify the working location.

Never invent the branch name or discard user work.

## Engineering Intent Modes

Distinguish the following outcomes from natural-language requests:

- Architecture or solution design.
- Time and cost estimation.
- Requirements analysis or technical investigation.
- Software implementation.
- Validation, debugging, or code review.

Do not force the user to learn new commands when the intended outcome is clear from the request.

- For architecture-only and estimation-only requests, do not start implementation or modify application source code unless separately authorized.
- For estimation requests, follow the Estimation Workflow in `workflows/estimation.md`; it produces a durable document and never implements.
- For implementation requests, use the existing Task-to-Code workflow.

## Requirement Alignment Before Implementation

For software implementation tasks:

1. Understand the business objective and inspect available task, design, and repository context.
2. Identify relevant acceptance criteria, constraints, non-goals, and material unknowns.
3. Infer routine technical details from the existing codebase, framework conventions, official documentation, and current project policies.
4. Ask concise questions when important business requirements are missing, conflicting, or cannot safely be inferred.
5. Present a concise summary of the agreed behavior and proposed approach before implementation.
6. Ensure the task is aligned with the user before beginning code changes.

Constraints:
- Do not produce a lengthy Markdown specification as a substitute for conversation.
- Treat the original task — provider ticket, artifact, or short direct description — as the source of truth in its original wording, and keep any working interpretation inside the session, persisted only through the checkpoint mechanism for continuity. Do not require the user to rewrite a short task into an elaborate prompt or to maintain a separate task-specification document.
- Do not ask questions whose answers are already available in the repository, task, design, configuration, or prior conversation.
- Do not ask the user to choose a framework, library, design pattern, file structure, or implementation detail when the correct choice can be inferred safely from evidence and existing conventions.
- Follow the existing branch-name policy before modifying application code.

## Autonomous Implementation After Alignment

Once the task scope and approach are aligned, independently carry out routine implementation work. Make decisions required to:

- Choose functions, components, names, types, and local abstractions.
- Reuse existing dependencies and project utilities.
- Implement tests and regression coverage.
- Consult official documentation for version-sensitive or unfamiliar behavior.
- Configure and run the existing formatter, linter, type checker, tests, and build as appropriate.
- Fix routine defects discovered during implementation and validation.
- Perform localized refactoring necessary to maintain code quality **only when required by the task**; unsolicited restructuring, component extraction, splitting, or new abstractions are out of scope and become Engineering Insights.
- Choose the smallest maintainable solution consistent with project conventions.

- Do not pause for user approval for each file edit, test, formatting change, local refactoring, or ordinary corrective adjustment within the approved scope.
- Do not weaken existing quality, security, testing, or validation standards for speed.
- For new dependencies, follow `workflows/dependency-assessment.md`. Preserve its explicit authorization requirements and mandatory formatter/linter exception.

## Human Decision Gates

Ask the user before proceeding only when the decision materially requires human judgment or authorization, including:

- Ambiguous or conflicting business requirements that cannot be resolved from available evidence.
- Significant architectural choices with meaningful long-term consequences.
- A **new project**: the user chooses the architectural style. Present the
  applicable styles with their trade-offs and ask which one to use. A
  recommendation is not a decision, and the structure is not created before
  the answer. Use `workflows/project-initialization.md`.
- A new project settles its **target version** of the language, framework, and runtime before its structure is created. The source of truth is the source analysis, then the ecosystem's official documentation, then a stated default. A version is never chosen because it happens to be installed, and the official documentation of the detected stack is consulted **for that version** before any stack-specific decision. The consultation is recorded.
- Consulted documentation must be **current**. Consult the ecosystem's version guide first to learn what is current, prefer a version-agnostic source, and use a source pinned to one release only when the guidance differs between releases. A pinned source is registered with its version and a review rule, so a stale one is visible instead of silently applied. The consulted version and the consultation date are recorded, and a source that does not cover the target version is reported rather than applied.
- A new project settles its **startup decision set** in one pass, before
  any structure is created: the architectural style, the API surface, the
  data and persistence layer, identity and access, asynchronous work and
  integrations, operations, and the decisions of the project type. The
  catalog is `references/project-startup-decisions.md`. Resolve each
  decision from the source analysis first, then from the detected
  conventions, then from a stated default. Ask the user once, grouped by
  theme, only for the decisions that materially change the outcome and have
  no defensible default. Do not re-ask what the source analysis already
  answers, and never decide silently.
- A new project is then **created step by step**, in the order defined by
  `references/project-creation-order.md`, and each step is verified before
  the next one starts. Every step states where the project grows next, so
  the structure has an extension point instead of a parallel structure. The
  creation checklist from that reference is completed and reported.
- A new project is created **progressively, with user-controlled stopping
  points**. Agree the scope catalog first: the modules, the public surfaces,
  and the dependencies with their reasons. The catalog is a ceiling, not a
  floor. After each step, verify it, update the progress tracker, report
  what the next step would add, and **wait**. Do not generate the remaining
  steps to "finish" the project, and never exceed the agreed scope without
  asking. Stopping at a checkpoint is a normal outcome, not a defect.
- Creating a project from zero requires the explicit `create-project` command.
  Without it, do not create a project from scratch; ask the user first.
- A proposed new runtime or development dependency under the canonical dependency policy.
- Changes that materially affect scope, backward compatibility, security, sensitive data, or production behavior.
- Destructive actions or changes to shared or production resources.
- New paid services, billable usage, or additional cost requiring approval.
- A material departure from the agreed approach.
- Creating a commit, including `git commit` and `git commit --amend`.

Present the recommended option, relevant alternatives, concrete trade-offs, and the reason a human decision is needed.

- Do not ask for decisions already governed by a clear repository policy.
- For architecture changes, follow the existing architecture approval rules. Do not classify every component or function choice as a significant architectural decision.

## Commit Approval Policy

Creating a commit always requires explicit human approval in the current
conversation.

- Never run `git commit`, `git commit --amend`, or any command that creates
  a commit without explicit approval of the proposed scope and message.
- Invoking `/commit`, `/task`, or any other command authorizes **preparing and
  validating a commit proposal**, not creating commits.
- A general request to implement a task is not authorization to commit.
- The required sequence is: inspect the change set → run the project's gates
  → present the proposed commit groups, messages, and files → ask for explicit
  approval → create the commits only after an affirmative answer.
- If the user declines or requests changes, do not commit; adjust the proposal
  and ask again.
- If the approved scope or message changes after approval, re-present the
  proposal and ask again.
- This policy applies to every client, agent, and workflow, including
  automation. There is no implicit or standing commit authorization.
- Preparation commands that never create commits, such as a dry run or a
  commit proposal, remain available without this gate.

## Progress Updates Without Approval Fatigue

During implementation, provide concise status updates at meaningful milestones: requirements and approach aligned, implementation milestone completed, important new finding or decision required, validation results, final diff and completion summary. Each update must explain what changed, the evidence available, and the next step. An update is not a request for approval unless a decision gate actually applies.

If new evidence changes a material requirement or approved design, pause and explain the reason before continuing beyond the approved scope. Otherwise, proceed autonomously.

Use checkpoints for long-running tasks according to `workflows/session-resilience.md`. Do not require the user to restart the terminal as the normal recovery method.

## Implementation and Validation

- Follow existing project conventions.
- Code, identifiers, comments, JSDoc, and TSDoc must be in English.
- Human interaction may be in Spanish or English.
- Test and validate changes using available project tooling.
- Run the structural compliance check in `workflows/output-compliance.md` over
  the change set before reporting a task as implemented. It verifies the
  mechanical rules that prose cannot enforce: arrow functions over `function`
  declarations, type declarations separated from runtime behavior, contract
  files grouped by consumer, and unit-folder test co-location. A `FAIL`
  result is a defect of the task, not a finding to defer.
- Format and lint changes with the project's configured tools.
- Report checks that were not run.
- Never claim a test or build passed without evidence.
- Keep unrelated refactoring out of scope.
- Reuse previously completed analysis after alignment when proceeding from architecture/estimation to implementation.

## Context and Session Resilience

Long-running tasks must survive context compaction (`/compact`) and a new
session (`/new`) without losing verified progress or requiring the user to
close and reopen the terminal.

- Keep automatic compaction enabled.
- Prune or discard old tool output where the client supports it, instead of
  disabling compaction.
- Do not introduce undocumented context-threshold configuration keys.
- Keep token and context spend proportional to the task: prefer targeted
  reads and searches over loading whole files, bound tool output, consume
  subagent summaries instead of full transcripts, avoid re-reading or
  re-deriving established information, and load only relevant Skills and
  references. Never trade correctness, safety, or validation for token
  savings.
- Checkpoint at logical phase boundaries, before expensive validations, and
  before a context reset when needed.
- Do not depend solely on a displayed context percentage to decide when to
  checkpoint.
- Never require closing and reopening the terminal as the normal recovery
  path.
- Do not add an experimental compaction plugin unless a demonstrated
  requirement cannot be met with stable built-in functionality.

### Minimum Sufficient Context

Use the smallest context that is sufficient to produce a correct, validated
result. "Minimum tokens" is not a license to skip required evidence.

- Before broad exploration, record a Context Manifest: target project,
  evidence-only paths, task type, provisional rigor, required Skills and
  references, likely files, and an exploration budget.
- Use the soft exploration budgets and stop rules in
  `workflows/session-resilience.md`. They cover exploration only, not
  mandatory implementation, tests, formatting, build, or validation.
- If the budget is exceeded or two consecutive searches add no new evidence,
  summarize what is known and either continue with a recorded reason or ask
  before expanding scope.
- Measure token use after the task where the client exposes it, and calibrate
  budgets from evidence. Do not invent token targets.
- Never trade correctness, safety, or validation for lower token use.

The checkpoint and recovery behavior is defined canonically in
`workflows/session-resilience.md`, using the existing command conventions in
`commands/checkpoint.md` and `commands/resume-work.md`.

A checkpoint is written to the provider-neutral user-level handoff directory
outside the target project's source tree. Never store credentials, tokens,
or secrets in a checkpoint. Never overwrite a different task's checkpoint.
Checkpointing must not modify source files as a side effect.

Recovery treats a checkpoint as a guide, not as proof that the described
state still exists. Verify the actual repository, branch, HEAD, and
working-tree state, detect divergence, preserve current work, and continue
from the first incomplete action instead of repeating completed phases.
Never claim a prior validation passed without recorded evidence or
re-executing the relevant check.

## Self-Learning and Maturity

The system improves itself by capturing lessons from real work, consolidating
them, and promoting the stable, general ones into the canonical knowledge
layer. This is provider-neutral and is not a retrospective of people or
teams.

- The behavior is defined canonically in `workflows/self-learning.md`.
- **Read the applicable learnings before acting.** At the start of a task,
  read the store for the project scope and for the global scope, and apply
  every relevant entry. **Repeating a lesson the store already records is a
  defect of the task**, and the task is not complete until the repetition is
  corrected. An empty or missing store is reported and does not block the
  task.
- **Every task is executed to a Staff standard, however simple it is.** The
  maturity level describes how much direction and review the work needs,
  never how much quality it gets. Execution quality is constant; only the
  process rigor, the planning depth, and the review intensity are
  proportional to the task. A level below Staff never lowers the standard of
  the delivered code.
- Learnings are stored in the provider-neutral user-level state root
  (`${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}`), outside every
  target project and the canonical repository. Never store credentials,
  tokens, secrets, or personal data.
- Promotion to the canonical layer (Skills, workflows, AGENTS.md, or a
  reference) is always a proposal that requires human approval; the system
  never edits the canonical repository automatically.
- Maturity is descriptive and evidence-based, never self-declared authority,
  and never a reason to weaken safety, quality, or cost guardrails.

## Formatting and Linting Policy

This section is the single source of truth for the formatting and linting
policy. Skills and workflows reference it instead of maintaining
divergent copies.

Every software project the system works on must have a configured
formatter and a configured linter appropriate for its detected stack,
languages, and frameworks. This is mandatory, not an optional
recommendation.

Scope: this requirement covers the application code of software projects.
It does not apply to documentation-only knowledge repositories such as
the canonical ai-engineering repository itself. Never create a
package.json, install formatter or linter tooling, or add scripts in
ai-engineering solely to satisfy this policy. Verification infrastructure
for this system's own documents, if ever needed, is a separate decision.

- Classify the target repository first: mandatory setup applies only to
  software projects with application source code, never to
  documentation-only repositories.
- Detect the actual stack, versions, package manager, repository
  structure, and existing tooling before selecting tools.
- Preserve suitable existing tooling and configuration.
- Prefer established, maintained, open-source tools with conventional
  configuration; avoid redundant tools.
- Provide reproducible commands to run formatting and linting.
- Apply this requirement to new projects and ordinary implementation
  tasks, not only when `/validate` is invoked.
- Never classify missing formatting or linting tooling as
  `NOT_APPLICABLE` within the defined scope. An out-of-scope
  documentation-only repository is a scope determination, not a missing
  tool classification.
- When required tooling, configuration, scripts, or dependencies are
  missing, set them up as part of the work using the existing package
  manager and compatible versions, and disclose every change.
- A project is not fully validated while mandatory formatting or linting
  remains unconfigured or unverified.
- A successful build does not substitute for successful formatting or
  linting.

Default tool choices by ecosystem:

- JavaScript and TypeScript (React, Next.js, Vue, Node.js, React Native):
  Prettier for formatting and ESLint for linting, with the appropriate
  framework plugins.
- JavaScript and TypeScript additionally install the **structural compliance
  rules** defined in `references/linter-rules.md`: `func-style`, the
  `no-restricted-syntax` selector for function expressions, and the local
  `structural/no-mixed-types-and-runtime` rule, plus the contract consumer
  script. A linter that runs without these rules does not enforce the
  structural policy of this system and is not a complete setup.
- Every supported ecosystem installs the **type-per-file rule** when its
  linter can express it: Checkstyle `OneTopLevelClass` for Java, StyleCop
  `SA1402` for C#/.NET, and PHP_CodeSniffer
  `PSR1.Classes.ClassDeclaration` for PHP. The rule is language agnostic
  and applies to every supported ecosystem. When an ecosystem linter cannot
  express it, the rule is covered by the structural compliance check and the
  report states that it is covered by verification rather than by an
  installed rule.
- The **arrow function preference** is a TypeScript and JavaScript
  convention. Never impose it on an ecosystem that has no arrow syntax; the
  detected ecosystem's own convention prevails.
- Angular: Prettier and Angular ESLint, preserving Angular template and
  TypeScript linting.
- Ionic: the tooling of its underlying Angular or React stack, including
  relevant template checks.
- Python: Ruff format and Ruff check, unless a consistent formatter and
  linter already exist.
- Java: a compatible formatter such as google-java-format plus a linter
  such as Checkstyle or an equivalent project-standard tool.
- SQL: dialect-aware formatting and linting such as SQLFluff where
  compatible with the selected dialect.
- Other languages: the ecosystem-standard formatter and linter.

Do not force Prettier onto file types it does not support. Account for
monorepos, multiple languages, generated files, templates, and database
code, and ensure relevant source files are covered. Do not install paid
tools or introduce paid services.

## Dependency Policy

Every software implementation task must complete the evidence-based
dependency assessment defined in `workflows/dependency-assessment.md`
before its implementation plan is finalized. That workflow is the single
canonical procedure; Skills and workflows reference it instead of
duplicating its decision policy.

- Prefer no new dependency when the existing stack, native APIs, or
  project abstractions are sufficient, and check for a compatible already
  installed dependency before proposing a new one.
- Adding or installing a new runtime or development dependency requires
  explicit authorization for that specific option and scope.
- A general approval to implement a task is not authorization to introduce
  arbitrary libraries.
- The mandatory formatter and linter setup authorized above is the only
  exception; do not extend it to unrelated libraries or frameworks.
- Never install paid services or dependencies, initiate billable usage, or
  change an approved package, version, license, cost, or purpose without
  renewed authorization.
- This policy applies to target software projects, not to the
  documentation-only ai-engineering repository.

## Pull Request Policy

- /generate-pr generates copy-ready PR/MR content only.
- /review-pr generates copy-ready review feedback only.
- The user creates the PR/MR and manually publishes review comments.
- PR review must not edit code, publish comments, approve, merge, push,
  or update remote tasks.
- Preserve the user's active workspace and use an isolated worktree
  when necessary.

## Provider and Cost Policy

- MCP is optional.
- Prefer local Git, existing authorized access, and supplied artifacts
  when sufficient.
- Prefer open-source and free tools.
- Organization-provided paid licenses may be used when authorized.
- Do not initiate additional paid usage without approval.
- Never commit credentials or tokens.

## Client Compatibility

Keep the engineering knowledge provider-neutral.

Do not assume that OpenCode command syntax is supported by Claude Code
or Codex.

Client-specific commands and agent configuration must delegate to the
shared Skills and workflows instead of duplicating their logic.

`AGENTS.md` is the universal entry point. Every mandatory rule keeps its
canonical wording here, so it reaches every client: OpenCode loads it as
shared instructions, Claude Code loads it through `CLAUDE.md`, and Codex
loads it from the repository root. A rule that exists only inside a Skill
does not reach a client without Skills, nor an agent that never loads that
Skill. The registry in `references/mandatory-rules.md` records the canonical
wording, the anchor, and the required surfaces of every mandatory rule, and
the reachability check in `workflows/system-validation.md` verifies it.

**The system is portable across clients and operating systems.** The
repository is the source of truth, not any installation of it: a client that
cannot discover Skills still receives every mandatory rule, and a client that
only looks in a fixed directory is pointed at the canonical path. The
procedures assume a POSIX shell and degrade to `NOT VERIFIED` with the
reason when a tool is missing; they never hardcode an operating-system path
or a client-specific directory. The contract is defined in
`references/portability.md`.

## Official Documentation Grounding

Use references/official-documentation.md when framework-specific or
version-sensitive information is relevant.

Determine the installed technology and version before selecting guidance.

Consult official documentation on demand for:

- unfamiliar APIs
- framework-specific implementation decisions
- deprecated or experimental features
- version compatibility
- security-sensitive behavior
- migrations and upgrades
- uncertain performance or rendering behavior

Do not consult every documentation source for every task.

Prefer official documentation for technical facts, existing project
conventions for compatibility, and tests for validating actual behavior.

If authoritative information is unavailable, explain the limitation.

Do not add documentation MCP servers or external dependencies when
available local references or direct public URLs are sufficient.

## Official Documentation and External Skills

Use `references/official-documentation.md` as the shared index of
authoritative technical documentation.

Detect the installed language, framework, runtime, and version before
selecting documentation or external Skills.

Retrieve documentation on demand. Do not load entire sites or reference
all supported technologies during every task.

Use `integrations/skill-catalog.md` to identify candidate Skills.

Use `integrations/external-skill-registry.md` to verify Skills actually
installed in the canonical repository.

Do not assume a candidate Skill is installed or approved.

External Skills may complement internal Skills but must not override:

- AGENTS.md
- workspace safety
- user-provided task requirements
- branch policies
- read-only PR and Code Review policies
- security and cost restrictions

Prefer official documentation when it is sufficient.

Install external Skills only when their source, license, maintenance,
compatibility, supporting files, and practical value have been evaluated.

Prefer on-demand knowledge retrieval and existing project tools over
loading every Skill or introducing additional dependencies.
