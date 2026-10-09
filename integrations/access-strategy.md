
# Provider Access Strategy

## Purpose

Determine the simplest, safest, and least expensive method to access tasks,
repositories, Pull Requests / Merge Requests, and design artifacts.

The Engineering System must not require MCP as its default mechanism.

Provider access must remain independent from engineering knowledge,
workflows, and client-specific configuration.

## General Priority

Choose the first available method that provides sufficient information
for the requested task.

1. Existing local repository and files.
2. User-provided URLs accessible through available authorized tools.
3. Existing authenticated CLI or client connections.
4. Authorized read-only provider integrations or APIs.
5. MCP when structured or authenticated access is necessary.
6. User-supplied diffs, source files, screenshots, or exported documents.
7. Ask for missing information only when it materially affects the result.

This order may change when another method provides more reliable access
with less cost or context.

Do not configure a new provider merely because it is available.

## Repository Access

For local repositories:

- Inspect the Git remote.
- Inspect the current branch and commit.
- Inspect the working-tree status.
- Identify relevant files, dependencies, and configuration.
- Read additional files only when necessary.

For Pull Request / Merge Request reviews:

- Identify the source and target branches and commits when available.
- Retrieve the exact source revision when possible.
- Preserve the active workspace.
- Use an isolated Git worktree when necessary.
- Inspect relevant surrounding code and existing abstractions.
- Never publish or modify remote changes.

If the source revision cannot be retrieved, report the limitation and
use an authorized alternative or request the smallest useful artifact.

Never discard uncommitted changes or overwrite the user's working tree.

## Task Access

Accept task information through:

- task URLs
- task identifiers
- task descriptions supplied by the user
- Markdown or JSON files
- attachments
- existing provider integrations
- authorized CLI or API access

Retrieve acceptance criteria, comments, dependencies, attachments, and
related links only when available and relevant.

Prefer the original task source when accessible.

Do not require a task-management MCP when the supplied task information
is sufficient.

A task URL does not automatically provide access to private content.

## Design Access

Accept design references through:

- design URLs
- frame, node, page, or component identifiers
- screenshots and images
- PDFs
- diagrams
- wireframes
- HTML and CSS
- exported design specifications
- local design artifacts
- existing application screens

Prefer the smallest design selection relevant to the task.

Retrieve surrounding screen context only when needed to understand
layout, relationships, interactions, or responsive behavior.

A full-screen design may be used to understand the overall layout while
implementation remains limited to the task-specific region.

Do not require Figma or a prewritten design-analysis Markdown file.

If a design source is inaccessible, use available artifacts or request
an export or screenshot when the missing information materially affects
implementation.

Never invent inaccessible design content.

## Direct Reference Resolution

All workflows must accept direct references when the client supports the
required input and retrieval capabilities.

Accepted references include:

- task URLs and identifiers
- Pull Request / Merge Request URLs
- repository URLs
- design URLs
- documentation URLs
- local files
- user-provided descriptions and artifacts

For each reference:

1. Classify the resource.
2. Determine which information is required.
3. Check whether the information is already available.
4. Use local Git or filesystem access when appropriate.
5. Use direct URL retrieval when supported and sufficient.
6. Use an existing authenticated CLI, client, or read-only integration
   when necessary.
7. Use MCP when it provides required structured or authenticated access.
8. Request a minimal export, screenshot, diff, or pasted content only
   when essential information remains inaccessible.

When multiple references are supplied, classify and resolve them
independently.

Prefer relevant links already associated with the task or PR.

Do not retrieve every linked resource automatically.

## Client and Artifact Limitations

Choose an access method appropriate for the client, resource type, and
required information.

### Authentication

A URL does not grant authorization.

Do not assume that authentication from another browser session is
available to the AI coding client.

Never bypass authentication or access private resources without
authorization.

Do not expose credentials, cookies, tokens, or private content in logs
or generated reports.

### Textual Content

Use direct URL retrieval when the client supports it and the resource
is accessible.

A retrieved webpage does not prove that linked pages, attachments,
images, or comments were also retrieved.

Report which sources were actually inspected.

### Images and Binary Artifacts

Text retrieval may be insufficient for images, PDFs, and other binary
artifacts.

When necessary:

- use an artifact accessible to the client's file or image tools
- use an authorized provider integration
- request an export or screenshot

Do not claim that a binary artifact was inspected merely because its URL
was retrieved.

### Structured Design Data

Access to a design webpage is not equivalent to access to its structured
design model.

Prefer frame-, node-, page-, or component-level retrieval when the
authorized provider exposes it.

Otherwise, analyze the available visual artifact and report limitations.

Do not assume that a URL provides access to design variables, component
variants, layout metadata, or interactions.

### Provider Verification

Distinguish between:

- documented provider support
- configured client integration
- successful authentication
- verified read capability
- verified write capability

A provider appearing in documentation does not mean that it is configured.

Never claim successful access without evidence.

Read-only workflows require only the necessary read capability.

Never enable write access merely to retrieve context.

## MCP Selection

Before enabling or using an MCP:

1. Identify the missing information or capability.
2. Determine whether local files or existing tools provide it.
3. Select a provider with the required capability.
4. Prefer read-only permissions.
5. Enable only the relevant tools when practical.
6. Avoid duplicate providers for the same task.
7. Exclude unrelated MCP tools when practical.
8. Verify the required access before depending on it.

MCP is an optional integration mechanism, not a Core dependency.

Do not install an MCP solely because one exists.

## Cost and Authorization

Prefer:

- local tools
- open-source tools
- free services
- organization-provided licenses
- authorized low-cost services

Use an organization-provided paid service when it is authorized and
appropriate.

Do not purchase subscriptions or initiate additional paid usage without
authorization.

Distinguish the cost of provider access from the cost of AI model
inference.

A free MCP server does not guarantee that the underlying service or model
is free.

Credentials must remain outside the repository.

## Token Efficiency

- Load only relevant Skills.
- Retrieve only task fields needed for the current work.
- Retrieve only relevant design regions.
- Analyze the diff before expanding repository context.
- Inspect related code only when necessary.
- Avoid retrieving entire repositories or large design documents by default.
- Avoid repeating unchanged analysis.
- Use compact review memory for follow-up PR revisions.
- Do not load every provider's tools when one provider is sufficient.
- Expand context only to resolve ambiguity, verify a finding, or establish
  an important dependency.

Accuracy and security take precedence over token minimization.

## Failure Handling

If access fails:

1. Report what could not be accessed.
2. Continue with reliable available information when safe.
3. Identify any missing context that materially affects correctness.
4. Request the smallest additional artifact or permission required.

Never fabricate inaccessible content.

Never claim that a task, design, repository, or PR was successfully
inspected when it was not.

## Read-Only and Remote Write Policy

The current workflow supports preparing copy-ready engineering artifacts.

The system may:

- prepare a copy-ready PR/MR title and description
- prepare copy-ready PR review comments
- inspect repositories and diffs using authorized read access
- execute appropriate local validation when permitted

The current workflow does not authorize:

- creating or publishing PRs/MRs
- publishing review comments
- approving or rejecting PRs/MRs through a provider
- merging Pull Requests
- pushing source branches
- updating remote task status
- modifying remote repository data

The user performs these remote actions manually.

These restrictions apply even if a provider integration exposes write
capabilities.

Future changes to this policy require explicit authorization and
corresponding updates to the affected workflows and client permissions.
