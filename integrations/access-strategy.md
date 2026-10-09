# Provider Access Strategy

## Purpose

Determine the simplest safe method to access tasks, repositories, PRs, and
design artifacts.

The Engineering System must not require MCP as its default mechanism.

## General Priority

Choose the first available method that provides enough information for
the requested task.

1. Existing local repository and files.
2. User-provided URL accessible through existing authorized tools.
3. Existing authenticated CLI or client connection.
4. Authorized read-only MCP or provider API.
5. User-supplied diff, source files, screenshots, or exported documents.
6. Ask for missing information only when it materially affects the result.

This priority may change when a provider offers significantly better or
more reliable access through an available integration.

## Repository Access

For local repositories:

- inspect the Git remote
- inspect the branch and commit
- inspect the working-tree status
- inspect only relevant files and dependencies

For PR reviews:

- identify source and target commits
- preserve the active workspace
- use an isolated worktree when necessary
- inspect relevant surrounding code
- never publish or modify remote changes

If local credentials cannot retrieve the PR source, use another authorized
read-only method or request the missing artifacts.

## Task Access

Accept:

- task URLs
- identifiers
- task descriptions
- exported Markdown
- JSON
- attachments
- provider integrations

Retrieve acceptance criteria, comments, links, and dependencies only when
available and relevant.

Do not require a task-management MCP when the task information supplied
by the user is sufficient.

## Design Access

Accept:

- URLs
- node/frame/page identifiers
- images and screenshots
- PDFs
- diagrams
- wireframes
- HTML/CSS
- exported design specifications
- local design artifacts

Prefer a task-relevant design selection.

Retrieve the surrounding screen only when needed to understand layout,
interaction, or responsive behavior.

If the design URL is private or inaccessible, do not invent its contents.
Request an export or screenshot when necessary.

## MCP Selection

Before enabling an MCP:

1. Determine which information is missing.
2. Determine whether local files or existing tools provide it.
3. Select the provider with the required capability.
4. Prefer read-only permissions.
5. Enable only relevant tools.
6. Avoid enabling duplicate providers for the same task.
7. Disable or exclude unrelated MCP tools when practical.

## Cost and Authorization

Prefer:

- local tools
- open-source tools
- free tiers
- organization-provided licenses
- authorized low-cost services

Do not purchase a service or initiate paid usage without authorization.

Keep credentials outside the repository.

## Token Efficiency

- Load only relevant Skills.
- Retrieve only relevant task fields.
- Retrieve only relevant design regions.
- Analyze the diff before expanding repository context.
- Avoid repeating unchanged analysis.
- Use compact review memory for follow-up commits.
- Do not retrieve large design documents or entire repositories by default.

## Failure Handling

If access fails:

- report what is inaccessible
- continue with reliable available information
- identify important gaps
- ask for the smallest additional artifact required

Never fabricate successful access.
