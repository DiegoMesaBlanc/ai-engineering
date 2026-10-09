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

## Direct Reference Resolution

All workflows must accept direct references when the client supports the
required input and retrieval capabilities.

Accepted references include:

- task URLs and identifiers
- Pull Request / Merge Request URLs
- repository URLs
- design URLs
- documentation URLs
- local files and user-provided artifacts

## Resolution Order

For each reference:

1. Classify the resource.
2. Check whether its content is already available in the current context.
3. Use local Git or filesystem access if appropriate.
4. Use available direct URL retrieval when the resource is accessible.
5. Use an existing authenticated client, CLI, or read-only integration
   when required.
6. Use MCP only when it provides necessary access or structured data that
   simpler methods cannot provide.
7. Request a minimal export, screenshot, diff, or pasted content only
   when essential information remains inaccessible.

## Private Resources

A URL does not provide authentication.

Never bypass authentication or attempt to access content without
authorization.

Do not expose tokens, cookies, or private content in logs or generated
reports.

## Multiple References

If multiple URLs are supplied, classify them independently as task,
design, repository, PR, or documentation references.

Prefer links already present in the task or PR.

Do not retrieve every linked resource automatically. Retrieve only
resources relevant to the current task and expand context when needed.

## Continuation with Partial Context

If one reference fails, continue with other reliable sources when safe.

Clearly identify which sources were accessible and which were not.

Do not claim that an inaccessible task, design, or repository was
successfully inspected.

## Client Retrieval Limits

The access method must match the artifact type.

### Textual URLs

Use direct URL retrieval when the client supports it and the resource is
accessible.

Do not assume that authentication in a separate web browser is shared
with the AI coding client.

### Images and Binary Artifacts

A textual URL-fetching tool may not retrieve an image, PDF, or other binary
artifact as usable content.

When required:

- use an artifact accessible to the client's file/image tools
- use an authorized provider integration
- request an export or screenshot if necessary

### Structured Design Data

Do not treat access to a design webpage as equivalent to access to its
structured design model.

Prefer frame/node-level retrieval when the authorized provider exposes it.

Otherwise, analyze the available visual artifact and report limitations.

### Provider Validation

Before recommending an integration, distinguish:

- documented support
- installed configuration
- successful authentication
- verified read capability
- verified write capability

Read-only workflows require only the necessary read capability.

Never enable write access merely to retrieve context.

## Artifact and Client Limitations

Use an access method appropriate for the resource type.

### Textual URLs

Use direct retrieval if the client supports it and the resource is
accessible.

Do not assume that authentication in another browser session is shared
with the AI coding client.

### Images and Binary Artifacts

Textual URL-fetching tools may not retrieve images, PDFs, or other binary
artifacts as usable content.

When needed:

- use an artifact accessible to the client's file/image tools
- use an authorized integration
- request an export or screenshot

### Structured Design Data

Access to a design webpage is not equivalent to access to its underlying
design model.

Prefer frame/node-level retrieval when the authorized provider exposes it.

Otherwise, analyze the accessible visual artifact and report limitations.

### Provider Verification

Distinguish documented support from configured access and verified
capabilities.

Read-only workflows need only read access.

Never enable remote write operations merely to retrieve context.
