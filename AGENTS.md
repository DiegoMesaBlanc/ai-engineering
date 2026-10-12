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

Do not duplicate Skills, workflows, or architecture definitions.

## Engineering Principles

- Inspect before assuming.
- Understand the task before implementation.
- Detect the actual stack, repository topology, and architecture.
- Reuse existing project abstractions when appropriate.
- Prefer the simplest solution that correctly solves the task.
- Evaluate monorepos, microfrontends, Atomic Design, and design patterns
  only when evidence justifies them.
- Require human approval for significant architectural changes.
- Load only relevant Skills and supporting references.

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

## Implementation and Validation

- Follow existing project conventions.
- Code, identifiers, comments, JSDoc, and TSDoc must be in English.
- Human interaction may be in Spanish or English.
- Test and validate changes using available project tooling.
- Format and lint changes with the project's configured tools.
- Report checks that were not run.
- Never claim a test or build passed without evidence.
- Keep unrelated refactoring out of scope.

## Context and Session Resilience

Long-running tasks must survive context compaction (`/compact`) and a new
session (`/new`) without losing verified progress or requiring the user to
close and reopen the terminal.

- Keep automatic compaction enabled.
- Prune or discard old tool output where the client supports it, instead of
  disabling compaction.
- Do not introduce undocumented context-threshold configuration keys.
- Checkpoint at logical phase boundaries, before expensive validations, and
  before a context reset when needed.
- Do not depend solely on a displayed context percentage to decide when to
  checkpoint.
- Never require closing and reopening the terminal as the normal recovery
  path.
- Do not add an experimental compaction plugin unless a demonstrated
  requirement cannot be met with stable built-in functionality.

The checkpoint and recovery behavior is defined canonically in
`workflows/session-resilience.md`, using the existing command conventions in
`commands/checkpoint.md` and `commands/resume-work.md`.

A checkpoint is written to the user-level handoff directory outside the
target project's source tree. Never store credentials, tokens, or secrets in
a checkpoint. Never overwrite a different task's checkpoint. Checkpointing
must not modify source files as a side effect.

Recovery treats a checkpoint as a guide, not as proof that the described
state still exists. Verify the actual repository, branch, HEAD, and
working-tree state, detect divergence, preserve current work, and continue
from the first incomplete action instead of repeating completed phases.
Never claim a prior validation passed without recorded evidence or
re-executing the relevant check.

## Formatting and Linting Policy

This section is the single source of truth for the formatting and linting
policy. Skills and workflows reference it instead of maintaining
divergent copies.

Every project must have a configured formatter and a configured linter
appropriate for its languages and frameworks. This is mandatory, not an
optional recommendation.

- Detect the actual stack, versions, package manager, repository
  structure, and existing tooling before selecting tools.
- Preserve suitable existing tooling and configuration.
- Prefer established, maintained, open-source tools with conventional
  configuration; avoid redundant tools.
- Provide reproducible commands to run formatting and linting.
- Apply this requirement to new projects and ordinary implementation
  tasks, not only when `/validate` is invoked.
- Never classify missing formatting or linting tooling as
  `NOT_APPLICABLE`.
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

## Pull Request Policy

- /pr generates copy-ready PR/MR content only.
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
