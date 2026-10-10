# Provider Policy

## Objective

Keep the Engineering System free-first, open-source-first, portable, and
independent from commercial services.

---

# Priority

Evaluate providers in this order:

1. Open-source and locally executable
2. Free service with no mandatory paid plan
3. Official free hosted service
4. Very low-cost service
5. Paid service only when no practical free alternative exists

---

# Requirements

For every provider evaluate:

- license
- source availability
- local execution
- hosted execution
- authentication requirements
- usage limits
- pricing
- API/MCP availability
- data privacy
- portability
- maintenance status
- client compatibility

---

# Core Rule

The Engineering Core must never require a commercial provider.

A provider is optional infrastructure.

---

# MCP Rule

MCP is an integration mechanism, not an architectural dependency.

The Core must work without MCP.

MCP adapters may implement provider contracts.

---

# Credential Rule

Never store credentials in:

- ai-engineering repository
- project repository
- Skills
- Agents
- Workflows
- Commands
- Markdown configuration committed to Git

Credentials belong to the user's local client/provider configuration.

---

# Provider Evaluation

## Cost Classification

Classify every provider as:

- OPEN_SOURCE
- FREE
- ORGANIZATION_LICENSED
- LOW_COST
- PAID
- UNKNOWN

Use the same classification in provider-matrix.md.

A provider's software license and service cost are different attributes.

A provider can be open-source while the hosted service requires payment.

An organization-provided license can be used when authorized without
making that paid service a mandatory Core dependency.

Do not initiate additional paid usage without approval.

---

# Removal Rule

A provider must be replaceable without changing:

- Core principles
- Skills
- Workflows
- task interpretation
- design analysis
- PR review logic

Only the provider integration should change.

## Organization-Provided Services

The system may use paid services or licensed MCP providers when the
organization already provides authorized access.

Examples include:

- company-provided API tokens
- organization-managed MCP services
- licensed design tools
- enterprise task-management integrations
- company-approved AI models and gateways

Before using an organization-provided service, determine when possible:

- whether use is authorized
- which account or credential is required
- which capabilities are available
- whether usage generates additional costs
- whether the service can access the required project data

Do not purchase subscriptions or initiate paid usage independently.

Do not assume a supplied token is unlimited or free to use.

## Optional Integrations

MCP is optional.

Use direct repository access, local Git, supplied artifacts, existing
connectors, or documented provider APIs when they provide a simpler
solution.

Do not install an MCP solely because it exists.

## Read-Only Default

For PR analysis and review, prefer read-only access.

Request only the capabilities needed to:

- retrieve PR metadata
- retrieve diffs
- inspect source files
- retrieve task context
- retrieve design context
- read previous review comments

Do not enable remote write capabilities for read-only workflows.

## Credential Separation

Credentials must remain in the appropriate local or organization-managed
configuration.

Never commit credentials into:

- ai-engineering
- a project repository
- Skills
- Agents
- Workflows
- Commands
- documentation

Use the least privileged credentials appropriate for the task.