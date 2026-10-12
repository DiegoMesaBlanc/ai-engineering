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
- Report checks that were not run.
- Never claim a test or build passed without evidence.
- Keep unrelated refactoring out of scope.

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
