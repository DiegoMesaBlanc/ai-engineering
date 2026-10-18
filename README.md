# AI Engineering System

A provider-agnostic, free-first engineering workflow with shared engineering guidance and client-specific entry points.

## Design principles

- One canonical repository for Skills, agents, commands, contracts, and workflows.
- Load Skills on demand; do not load the entire knowledge base for every task.
- Prefer local Git and supplied artifacts when sufficient; MCP is optional.
- Detect project stack, architecture, repository topology, and conventions from evidence.
- Evaluate architecture, reuse, and code organization before implementing, and validate the result against the proposed design and project conventions.
- Ask the user for an exact branch name before implementation if one was not supplied.
- Preserve user changes and use isolated worktrees for PR inspection when needed.
- `/pr` generates copy-ready PR text; it does not create a PR.
- `/review-pr` creates evidence-based, copy-ready feedback grounded in the reviewed revision; it does not publish comments or modify remote repositories.
- Prefer free/open-source options, but use company-provided licenses when authorized.
- Use official documentation on demand for the detected technology and version.
- Evaluate external Skills across the full supported stack without installing one per framework by default.
- Require a configured formatter and linter for every software project, per its detected stack, with reproducible commands and honest validation results; documentation-only repositories are not converted to satisfy this policy.
- Delivery and CI/CD checks are read-only by default; deployment and publication require explicit authorization for the specific operation.

## Current top-level structure

- `agents/`: OpenCode agents and orchestration.
- `commands/`: commands such as `/task`, `/pr`, `/review-pr`, `/system-check`, `/validate`, `/checkpoint`, `/resume-work`, `/provider-check`, and `/delivery-check`.
- `contracts/`: provider-neutral task, design, and Git contracts.
- `integrations/`: provider access, security, cost, and compatibility policy.
- `references/`: official documentation indexes and technical references.
- `integrations/skill-catalog.md`: evaluation decisions for external Skills.
- `integrations/external-skill-registry.md`: provenance and license records for Skills actually installed.
- `skills/`: reusable domain and engineering knowledge.
- `workflows/`: end-to-end engineering processes.
- `AGENTS.md`: canonical shared engineering instructions.
- `CLAUDE.md`: Claude Code compatibility entry point importing `AGENTS.md`.

Do not create `core/` or `external/` solely to match a previous folder proposal. Add a directory only when it owns real, non-duplicated content.

## Commands

- `/task`: start or continue an engineering task.
- `/pr`: generate copy-ready PR/MR content.
- `/review-pr`: generate copy-ready review feedback.
- `/system-check`: read-only validation of repository consistency and
  OpenCode and Codex CLI integration.
- `/validate`: validate project changes using existing tools and
  risk-based checks without modifying application source code.
- `/checkpoint`: write a user-level checkpoint of the current task state
  outside the target project's source tree, without modifying source files
  or remote state.
- `/resume-work`: recover a long-running task from a checkpoint by verifying
  actual repository state and continuing from the first incomplete action.
- `/provider-check`: verify actual task, design, and Git provider
  capabilities with evidence-based statuses, read-only and without
  exposing credentials.
- `/delivery-check`: read-only assessment of the current project's
  CI/CD, security, and delivery readiness, with evidence-based statuses
  and without modifying the project or performing remote writes.
- `/estimate`: estimate effort, duration, and optional cost from a supplied
  artifact (document, text, link, file, or image), producing a durable document
  in the current working directory without starting implementation.

OpenCode supports Markdown commands and agents with permissions defined in
frontmatter. Those files are OpenCode adapters; they do not create slash
commands in other clients. Codex uses the shared `AGENTS.md` and native
`SKILL.md` format, but its CLI does not automatically discover this
repository's `commands/` directory.

### System validation versus project validation

- `/system-check` audits the AI Engineering System, file structure,
  Skills, workflows, references, and client configuration.
- `/validate` audits the current software project's changes using
  detected tooling, relevant tests, static checks, and build validation,
  and configures the mandatory formatter and linter when they are missing.
- `/delivery-check` assesses CI/CD, security, and delivery readiness of
  the target project in read-only mode; it does not change the project.

### Provider verification

- `/provider-check` distinguishes documented support, client
  configuration, authentication, verified read access, and write
  capability.
- Proven capabilities are reported with observed evidence; simulated
  results are labeled SIMULATED and never reported as real verification.
- The absence of optional external providers is not a failure of the
  provider-agnostic core.

### Delivery and secure CI/CD readiness

- `/delivery-check` assesses the CI/CD, security, and delivery readiness
  of a target software project in read-only mode. It reports the detected
  stack, platform, configured quality gates, executed security controls,
  and delivery safeguards, and distinguishes configured from executed,
  passed, failed, blocked, and not verified.
- `/delivery-check` modifies nothing: assessment does not install, deploy,
  publish, or change CI configuration. Recommended controls are handled as
  scoped implementation work under the existing authorization and branch
  policies.
- `/delivery-check` is distinct from `/validate`, which validates the
  current change set with the project's tools, and from `/provider-check`,
  which verifies the capabilities of task, design, and Git providers.
- Missing controls are recommendations for a later implementation, not
  proof of execution. Documentation-only repositories are not converted
  into software projects to satisfy software-project CI/CD checks.
- Authoring a delivery pipeline is a separate, authorized mode of
  `workflows/secure-delivery.md`. It builds one artifact and promotes it
  across environments (local, development, QA, staging, production), runs
  the required validation gates before every deployment, keeps environment
  differences out of the artifact, and requires explicit approval for
  production. A failing pipeline is met with a proposed fix or rollback,
  never by weakening gates.

### Dependency decisions

- Every software implementation task completes the evidence-based
  dependency assessment in `workflows/dependency-assessment.md` before its
  implementation plan is finalized. The assessment records one outcome:
  `NO_NEW_DEPENDENCY`, `REUSE_EXISTING`, `NEW_DEPENDENCY_PROPOSED`, or
  `BLOCKED`.
- The assessment prefers the existing stack, native APIs, and project
  abstractions, and checks for a compatible installed dependency before
  proposing a new one. No new dependency is requested when none is needed.
- Adding or installing a new runtime or development dependency requires
  explicit authorization for that specific option, except for the mandatory
  formatter and linter setup. A general task approval is not authorization
  to introduce arbitrary libraries.
- This policy applies to target software projects. The documentation-only
  ai-engineering repository is never converted into a software package to
  satisfy it.

### Architecture-first engineering

- `/task` evaluates architecture, reuse, and code organization before
  implementing, for every framework, language, and application type. The
  canonical evaluation lives in `skills/architecture-advisor/SKILL.md` and
  is performed in the architecture phase of `workflows/task-to-code.md`.
- The evaluation covers where components, hooks, services, configuration,
  types, and interfaces belong; cohesion and separation of
  responsibilities; public exports, barrels, import aliases, and module
  conventions; design patterns only when justified; and styling decisions.
- The diagnosis is evidence-based and classified as detected, inferred, or
  absent; directory names alone do not establish an architecture. When no
  coherent architecture is detected, conventions conflict, or a new one must
  be chosen, options and trade-offs are presented and the user decides
  before implementation.
- After implementation, the result is validated against the proposed design
  and the project's conventions, in addition to functional tests.
- No universal folder structure, fixed file count, or mandatory pattern is
  imposed. Routine technical details remain autonomous; only significant
  product or architecture decisions reach a human decision gate.

### Contracts, recipes, and decisions

- When a task changes a boundary, interface, or integration, the API and
  integration contract is defined before implementation following
  `skills/api-contract-design/SKILL.md`; the contract covers schemas, error
  model, authorization, versioning, and compatibility.
- Concrete, stack-appropriate implementation guidance lives in
  `skills/implementation-recipes/SKILL.md`. It defines the non-negotiable
  quality baseline: quality is never proportional to task size, so even
  trivial changes follow the project's conventions, typing, error handling,
  security, and tests. Only planning rigor and test depth are proportional.
- When a task touches the database — schema, migration, query, index, or
  transaction — `skills/database-engineering/SKILL.md` guides safe,
  backward-compatible changes that protect integrity, performance, and
  production data. It is not loaded for ordinary feature work.
- Significant, hard-to-reverse decisions are recorded briefly only when they
  meet the threshold, following the Decision Records guidance in
  `skills/architecture-advisor/SKILL.md`. Trivial or easily reversible
  decisions are not documented.
- For work with user-facing text, localization is settled early (required
  locales, RTL, locale-aware formatting); `skills/internationalization/SKILL.md`
  then guides correct externalization, pluralization, formatting, direction,
  and translation workflow. It is not loaded for work without user-facing text.
- Mobile interfaces get mobile-specific analysis and implementation guidance
  from the Mobile Analysis section of `skills/design-analysis/SKILL.md` and the
  Mobile section of `skills/implementation-recipes/SKILL.md`, instead of
  reusing desktop assumptions.

### Proportional planning (SDD)

- `/task` plans proportionally to the change. Complexity is classified and a
  rigor level is recommended in `skills/complexity-advisor/SKILL.md`; the
  procedure lives in `workflows/task-planning.md`, and the retained result
  belongs to `skills/task-context/SKILL.md`.
- Rigor levels: R1 Brief (trivial, low), R2 Moderate (medium), R3 Complex
  (high), R4 High-risk (critical). Simple changes are not forced into a heavy
  specification; complex or high-risk changes define their expected behavior
  before implementing.
- Approval is triggered by the Human Decision Gates (risk, irreversible or
  high-impact operations, security or data, cost, scope, shared or production
  resources, project rules), not by the classification label.
- Plans are session state by default; a durable artifact is created only when
  traceability, approval, or an explicit request requires it.

### System workflow regression matrix

- `references/system-workflow-regression-matrix.md` is a lightweight,
  optional reference for protecting the transversal behavior of the system.
  It is not a mandatory procedure for every task and does not define a closed
  architecture, a fixed process order, or a new layer.
- Scenarios are selected according to the files, policies, and behaviors a
  change affects, not run wholesale by default. Broader integral validation
  is reserved for significant changes such as the orchestrator, permission
  boundaries, or interaction between several workflows.
- Recorded outcomes are classified REAL, SIMULATED, or STATIC and are never
  conflated. The matrix requires no new command, no package.json, and no
  testing framework in the canonical documentation repository.

### Self-learning and maturity

- The system improves itself by capturing lessons from real work,
  consolidating them, and proposing the stable, general ones into the
  canonical knowledge layer. The procedure is
  `workflows/self-learning.md`, and it is not a retrospective of people or
  teams.
- Learnings and checkpoints live in a provider-neutral user-level state root,
  `${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}`, outside every target
  project and the canonical repository. The same root is used by OpenCode,
  Claude, Codex, and Aider; no client has a hardcoded path.
- Promotion to a Skill, workflow, `AGENTS.md`, or a reference is always a
  proposal that requires human approval; the system never edits the canonical
  repository automatically. Maturity (L1 Guided to L5 Adaptive) is
  evidence-based and descriptive, never self-declared authority.

### Estimation

- Estimation is a standalone activity, not a step of implementation.
  `/estimate` takes a supplied artifact and follows `workflows/estimation.md`,
  which is the canonical definition of the accepted artifacts, the procedure,
  and the report. Estimation never modifies application source, configuration,
  a branch, or remote state.

## Codex CLI setup

Codex CLI reads `AGENTS.md` files in the working project and supports Skills
in its native skill directories. It does not automatically discover this
repository's `skills/` directory from another project. To use the canonical
Skills without changing global configuration, give Codex access to this
repository and explicitly direct it to read the relevant `SKILL.md` files.
This uses the installed CLI; it does not install a plugin or create a slash
command.

Set the canonical repository path and verify it:

```sh
export AI_ENGINEERING_HOME="$HOME/Documents/ai-engineering"
git -C "$AI_ENGINEERING_HOME" status --short --branch
```

From the target project, start an implementation task with the original task
wording. Codex must inspect the project and complete architecture analysis
before changing application code:

```sh
codex -C "$PWD" --add-dir "$AI_ENGINEERING_HOME" \
  "Read '$AI_ENGINEERING_HOME/AGENTS.md', '$AI_ENGINEERING_HOME/workflows/task-to-code.md', '$AI_ENGINEERING_HOME/workflows/dependency-assessment.md', '$AI_ENGINEERING_HOME/skills/discovery/SKILL.md', '$AI_ENGINEERING_HOME/skills/project-analysis/SKILL.md', and '$AI_ENGINEERING_HOME/skills/architecture-advisor/SKILL.md'. Follow the canonical task workflow. Treat commands/task.md as an OpenCode adapter, not a Codex command. My task is: <describe the task>."
```

Codex must ask for the exact branch name before changing application code
when the task did not provide one. Preserve the shared repository: use it as
read-only guidance and make task changes only in the target project.

To validate a completed project change, invoke the project workflow explicitly:

```sh
codex -C "$PWD" --add-dir "$AI_ENGINEERING_HOME" \
  "Read '$AI_ENGINEERING_HOME/AGENTS.md' and '$AI_ENGINEERING_HOME/workflows/project-validation.md'. Validate this project's current changes using its configured tooling. Report commands and actual results, including failed, blocked, and unexecuted checks. Do not install dependencies or modify application source during validation."
```

To audit the engineering system itself, ask Codex to read
`$AI_ENGINEERING_HOME/workflows/system-validation.md` and report each check
with evidence. That workflow validates system consistency; it does not
assert that a custom Codex slash command exists or that Codex was executed
unless an actual CLI run is recorded.

## OpenCode setup (macOS)

Set the path to the canonical repository:

```sh
export AI_ENGINEERING_HOME="$HOME/Documents/ai-engineering"
```

Verify the repository:

```sh
git -C "$AI_ENGINEERING_HOME" status --short --branch
```

OpenCode's global configuration directory is the expanded value of
`$HOME/.config/opencode`.

Before creating links, inspect every destination:

```sh
ls -ld "$HOME/.config/opencode/AGENTS.md" \
  "$HOME/.config/opencode/commands" \
  "$HOME/.config/opencode/agents" \
  "$HOME/.claude/skills" 2>/dev/null
```

If a destination does not exist, link it to the canonical repository:

```sh
ln -s "$AI_ENGINEERING_HOME/AGENTS.md" \
  "$HOME/.config/opencode/AGENTS.md"

ln -s "$AI_ENGINEERING_HOME/commands" \
  "$HOME/.config/opencode/commands"

ln -s "$AI_ENGINEERING_HOME/agents" \
  "$HOME/.config/opencode/agents"
```

Never overwrite an existing configuration or symlink without inspecting
its target.

Skills are stored in the canonical repository's `skills/` directory.
Expose them through the global Skill directory inspected above
(`$HOME/.claude/skills`):

- If `$HOME/.claude/skills` does not exist, link the canonical directory:

  ```sh
  ln -s "$AI_ENGINEERING_HOME/skills" "$HOME/.claude/skills"
  ```

- If it already exists as a symlink to the canonical `skills/` directory,
  reuse it and do not change it.

- If it exists but points elsewhere, or already contains other Skills, do
  not overwrite it. Link only the missing canonical Skills individually and
  skip names that already exist:

  ```sh
  for dir in "$AI_ENGINEERING_HOME"/skills/*; do
    target="$HOME/.claude/skills/$(basename "$dir")"
    [ -e "$target" ] || ln -s "$dir" "$target"
  done
  ```

Never overwrite an existing directory, symlink, or Skill.

Restart OpenCode after modifying discovery configuration.

The installation is not complete until `/system-check` reports PASS for
OpenCode discovery: global AGENTS.md, commands, agents, and Skills.

The installation must use one canonical repository. Client adapters must
not duplicate Skills, contracts, or workflows.

## Branch policy

For a task, the assistant must ask for the exact branch name before changing application code. It must validate the name, inspect branch collisions and working-tree state, preserve uncommitted changes, create/prepare the branch or an isolated worktree, and verify the working location. It must not push unless separately requested.

## PR policy

The assistant prepares copy-ready titles, descriptions, checks, and feedback, grounding review findings in the reviewed revision with evidence, severity, and confidence. The user manually creates the PR and pastes review comments. Current workflows must not publish comments, create PRs, approve, merge, or update remote tasks.
