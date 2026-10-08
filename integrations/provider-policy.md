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

Every new provider must be classified as:

OPEN
FREE
LOW_COST
PAID
UNKNOWN

The reasoning must be documented.

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