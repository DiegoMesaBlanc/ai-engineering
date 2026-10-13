# Provider Integration Proof

## Purpose

Establish a repeatable, evidence-based method to verify which task, design,
and Git provider capabilities actually work in the current environment.

Documented support, client configuration, successful authentication,
verified read access, and write capabilities are distinct. Simulated
results are never reported as proof of a real integration.

This workflow is the canonical definition of provider verification.
`commands/provider-check.md` is the client adapter that delegates here.

## Scope

Evaluate providers in these capability families:

- **Task providers**: Jira, Azure DevOps, GitHub and GitLab Issues.
- **Design providers**: Figma, Penpot, or authorized alternatives.
- **Git and repository-hosting providers**: GitHub, GitLab, Azure Repos,
  Bitbucket.

Evaluate only the providers and access methods relevant to the current
environment. Do not assume every listed provider must be installed,
configured, or verified. The absence of an optional provider is not a
failure of the provider-agnostic Core.

Prefer the least invasive inspection: existing authorized client
integrations, relevant configuration metadata, available CLIs, and access
mechanisms. Never print credentials or token values. Do not inspect
unrelated secrets or read credential stores merely to discover whether an
integration exists.

## Capability Statuses

Use a consistent status model for every provider capability:

- DOCUMENTED_ONLY: support is described but no configuration has been
  demonstrated.
- NOT_CONFIGURED: evidence indicates that the integration is not
  configured.
- CONFIGURED_UNVERIFIED: configuration is present, but authentication or
  capability has not been proven.
- AUTHENTICATED: authentication was successfully established, but the
  requested read capability has not yet been proven.
- READ_VERIFIED: an actual authorized read succeeded against a known
  resource.
- BLOCKED: a concrete limitation prevented verification.
- SIMULATED: behavior was tested using a fixture or mock rather than a
  real provider.

Never mark a provider READ_VERIFIED based only on configuration,
documentation, successful authentication, or a simulated fixture.

The generic support classification (CORE / OPTIONAL / CANDIDATE in
`integrations/provider-matrix.md`) is separate from these
machine-specific runtime statuses.

## Verify in This Order

1. Documented support: the provider appears in the matrix or provider
   documentation.
2. Client configuration: an integration entry is present in the client
   configuration, inspected without revealing its values.
3. Authentication: a trusted CLI or client confirms an authenticated
   session, without revealing secret values.
4. Read verification: an actual authorized read succeeds against a known
   resource.
5. Write tracking: record whether a controlled write test is authorized.

## Core Capabilities

Local Git and filesystem access are CORE capabilities. Verify local read
with real commands, for example `git status`, `git log`, or `git remote
-v`. These require no external provider.

## Read versus Write

Track read and write capabilities independently. Declared permissions,
configuration, or membership do not prove that a write operation succeeds.

The current system remains read-only for remote PR publication, review
comments, approvals, merges, pushes, and task-status updates. Do not
execute remote writes to test them.

Report write capability as NOT TESTED or an equivalent unverified status
unless an explicit future authorization for a controlled write test
exists.

## Provider Probes

### Task and Design Providers

Environment probes, in order:

- client configuration: detect whether an integration is declared, for
  example an MCP entry in the client configuration, without printing its
  values
- available CLI: check for provider-specific CLIs, for example `gh` for
  GitHub Issues
- no credential-store inspection

When no configuration and no CLI are present, report NOT_CONFIGURED. When
only documentation exists, report DOCUMENTED_ONLY.

### Git and Repository-Hosting Providers

- local remote: `git remote -v`
- CLI authentication: `gh auth status` when `gh` is available
- real read: `git ls-remote <remote>` or an authorized read against a
  known repository

`git ls-remote` against the actual configured remote is an authorized read
of repository references: record the command and observed output as
evidence.

## Simulated Results

Use an existing fixture or demonstration mechanism when appropriate. If no
suitable mechanism exists, prefer a small, focused fixture that mirrors the
relevant contract model (task-provider, design-provider, or
git-provider) rather than introducing a new testing framework.

Label every simulated result SIMULATED. A simulation never changes a real
provider's status to READ_VERIFIED or AUTHENTICATED.

Never create test resources in external services or modify remote data.

## Record Format

Record for each provider and capability:

- provider
- resource type
- verification method
- status
- evidence (command or probe and observed result)
- limitations
- real or simulated

Never record credential values.

## Safety

- Do not create provider credentials or prompt for secret values.
- Do not install MCP integrations or dependencies solely for this phase.
- Do not purchase or initiate paid usage.
- Do not create PRs, comments, approvals, merges, commits, or pushes.
- Do not enable or test remote write capabilities.
- Do not read unrelated credential stores.

## Output

Return a table per capability family:

| Provider | Access method | Status | Evidence | Real/Simulated | Limitations |
|---|---|---|---|---|---|

Distinguish static checks, simulated fixtures, actual successful reads,
and capabilities that could not be verified. Report the absence of
optional external providers as NOT_CONFIGURED and explain why real read
access could not be verified when that is the case. Never invent a
successful test.