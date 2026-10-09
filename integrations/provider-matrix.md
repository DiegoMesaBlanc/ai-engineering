# Provider Access Matrix

## Status Definitions

- CORE: works without an external provider integration.
- OPTIONAL: supported when the required access is available.
- CANDIDATE: evaluated as an option but not validated in the current setup.
- CONFIGURED: present in the client configuration.
- VERIFIED: successfully tested with the intended permissions.
- UNAVAILABLE: required access is not available.

A provider appearing in this file does not imply that it is configured.

## Access Matrix

| Capability | Source | Preferred Access | MCP Required | Status |
|---|---|---|---|---|
| Task | Markdown / JSON | Local files | No | CORE |
| Task | User description | Current conversation | No | CORE |
| Task | Public URL | Direct retrieval when supported | No | OPTIONAL |
| Task | Jira | Existing authorized access or integration | Conditional | CANDIDATE |
| Task | Azure DevOps | Existing authorized CLI/API or integration | Conditional | CANDIDATE |
| Task | GitHub / GitLab Issues | Authorized access or integration | Conditional | CANDIDATE |
| Repository | Local Git | Git CLI and filesystem | No | CORE |
| Repository | Public Git URL | Git or direct retrieval | No | OPTIONAL |
| PR / MR | Local changes | Git status, log, and diff | No | CORE |
| PR / MR | Hosted provider | Authorized read-only access | Conditional | CANDIDATE |
| Design | Images / screenshots | Supported image input | No | CORE |
| Design | SVG / HTML / CSS | Local files and available rendering | No | CORE |
| Design | PDF / exported specification | File inspection | No | CORE |
| Design | Public textual URL | Direct retrieval when supported | No | OPTIONAL |
| Design | Figma / Penpot structured data | Authorized provider integration | Conditional | CANDIDATE |
| Design | Private design source | Authorized integration or export | Conditional | CANDIDATE |

## Provider Verification

For each optional provider, distinguish:

- documented support
- configured client
- successful authentication
- verified read access
- verified write access

Do not classify a provider as VERIFIED unless the relevant capability
has been tested.

## Access Limitations

Direct URL access does not guarantee authentication.

A text retrieval tool may not be sufficient for:

- private task descriptions
- structured design nodes and variables
- binary images and PDFs
- authenticated PR diffs or comments

Prefer the simplest authorized access method that returns enough
information for the task.

Request an export, screenshot, or source file only when necessary.

## Cost

Provider access cost and AI model inference cost are separate.

A free MCP server does not guarantee free AI inference.

Classify providers as:

- OPEN_SOURCE
- FREE
- ORGANIZATION_LICENSED
- LOW_COST
- PAID
- UNKNOWN

Do not initiate additional paid usage without authorization.

## Current Scope

The repository defines provider-neutral contracts and access policies.

It does not automatically configure third-party services or credentials.

Do not assume any external provider is configured on the user's machine.

## Selection Rule

Prefer, in order:

1. Local files and Git.
2. Existing authorized client access.
3. Direct URL retrieval when sufficient.
4. Already-authenticated CLI/API.
5. MCP when required for authenticated or structured data.
6. Supplied artifacts when other access methods are unavailable.

Do not load multiple providers when one is sufficient.
