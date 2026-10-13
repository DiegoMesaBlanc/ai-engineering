# Provider Access Matrix

## Status Definitions

- CORE: supported without an external provider integration.
- OPTIONAL: supported when the required access is available.
- CANDIDATE: evaluated but not validated in the current environment.
- CONFIGURED: present in the client configuration.
- VERIFIED: successfully tested for the stated capability.
- UNAVAILABLE: required access is not available.

A provider appearing in this matrix does not imply that it is configured
or verified.

Cost classification, client configuration, and verified capability are
separate attributes. Keep them in separate columns or sections.

## Runtime Verification Status

The statuses above classify documented support and client configuration.
Machine-specific runtime verification uses the Provider Integration Proof
workflow (`workflows/provider-integration-proof.md`) and these statuses:

- DOCUMENTED_ONLY: support is described but no configuration has been
  demonstrated.
- NOT_CONFIGURED: evidence indicates that the integration is not
  configured.
- CONFIGURED_UNVERIFIED: configuration is present, but authentication or
  capability has not been proven.
- AUTHENTICATED: authentication was established, but the requested read
  capability has not yet been proven.
- READ_VERIFIED: an actual authorized read succeeded against a known
  resource.
- BLOCKED: a concrete limitation prevented verification.
- SIMULATED: behavior was tested using a fixture or mock rather than a
  real provider.

Read and write capabilities are tracked independently. Simulated results
never change a real provider's status. Do not commit credentials, personal
account information, or unredacted machine-specific runtime configuration.

## Access Matrix

| Capability | Provider | Access Method | Cost Classification | Status |
|---|---|---|---|---|
| Task | Markdown / JSON | Local filesystem | FREE | CORE |
| Task | User description | Current conversation | FREE | CORE |
| Task | Public URL | Direct retrieval when supported | UNKNOWN | OPTIONAL |
| Task | Jira | Authorized access or provider integration | UNKNOWN | CANDIDATE |
| Task | Azure DevOps | Authorized CLI/API or provider integration | UNKNOWN | CANDIDATE |
| Task | GitHub / GitLab Issues | Authorized access or provider integration | UNKNOWN | CANDIDATE |
| Repository | Local Git | Git CLI and filesystem | FREE | CORE |
| Repository | Public URL | Git or direct retrieval | UNKNOWN | OPTIONAL |
| PR / MR | Local changes | Git status, log, and diff | FREE | CORE |
| PR / MR | Hosted provider | Authorized read-only API, CLI, or MCP | UNKNOWN | CANDIDATE |
| Design | Images / screenshots | Supported image input | FREE | CORE |
| Design | SVG / HTML / CSS | Local artifact inspection | FREE | CORE |
| Design | PDF / specification | File inspection | FREE | CORE |
| Design | Public URL | Direct retrieval when supported | UNKNOWN | OPTIONAL |
| Design | Figma | Authorized provider integration | UNKNOWN | CANDIDATE |
| Design | Penpot | Local, API, or provider integration | OPEN_SOURCE | CANDIDATE |
| Design | Private design source | Authorized integration or export | UNKNOWN | CANDIDATE |

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

Do not mark a provider CONFIGURED or VERIFIED merely because its
documentation exists or its name appears in this matrix.

Follow `workflows/provider-integration-proof.md` for the repeatable
verification procedure and record machine-specific results there or in a
local report; do not commit personal account information or credentials to
this matrix.

If the license or the actual cost of the service has not been confirmed,
classify its cost as UNKNOWN.

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
