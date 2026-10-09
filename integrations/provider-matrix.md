# Provider Access Matrix

## Status Definitions

- CORE: supported without an external provider integration.
- OPTIONAL: supported when the required access is available.
- CANDIDATE: evaluated but not validated in the current environment.
- CONFIGURED: present in the client configuration.
- VERIFIED: successfully tested for the stated capability.
- UNAVAILABLE: required access is not available.

A provider appearing in this matrix does not imply that it is configured.

## Access Matrix

| Capability | Source | Preferred Access | MCP Required | Status |
|---|---|---|---|---|
| Task | Markdown / JSON | Local filesystem | No | CORE |
| Task | User description | Current conversation | No | CORE |
| Task | Public URL | Direct retrieval when supported | No | OPTIONAL |
| Task | Jira | Authorized access or provider integration | Conditional | CANDIDATE |
| Task | Azure DevOps | Authorized CLI/API or provider integration | Conditional | CANDIDATE |
| Task | GitHub / GitLab Issues | Authorized access or provider integration | Conditional | CANDIDATE |
| Repository | Local Git | Git CLI and filesystem | No | CORE |
| Repository | Public URL | Git or direct retrieval | No | OPTIONAL |
| PR / MR | Local changes | Git status, log, and diff | No | CORE |
| PR / MR | Hosted provider | Authorized read-only API, CLI, or MCP | Conditional | CANDIDATE |
| Design | Images / screenshots | Supported image input | No | CORE |
| Design | SVG / HTML / CSS | Local artifact inspection | No | CORE |
| Design | PDF / specification | File inspection | No | CORE |
| Design | Public URL | Direct retrieval when supported | No | OPTIONAL |
| Design | Figma / Penpot structured data | Authorized provider integration | Conditional | CANDIDATE |
| Design | Private design source | Authorized integration or export | Conditional | CANDIDATE |

## Cost Classification

Classify providers as:

- OPEN_SOURCE
- FREE
- ORGANIZATION_LICENSED
- LOW_COST
- PAID
- UNKNOWN

Provider-access cost and AI inference cost are separate.

Do not initiate additional paid usage without authorization.

## Verification

For each optional provider, distinguish:

- documented support
- configured client
- successful authentication
- verified read capability
- verified write capability

Only mark a provider VERIFIED for a capability after testing that
capability.

The current workflow requires read access only for PR review.

## Selection Rule

Prefer:

1. Local files and Git.
2. Existing authorized client access.
3. Direct URL retrieval when sufficient.
4. Already-authenticated CLI/API.
5. MCP when required for structured or authenticated access.
6. Supplied artifacts when other methods fail.

Avoid duplicate providers and unnecessary context loading.

The Core must work when no external provider is configured.
