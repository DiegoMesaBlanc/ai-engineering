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