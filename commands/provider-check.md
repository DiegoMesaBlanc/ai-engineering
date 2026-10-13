---
description: Verify actual task, design, and Git provider capabilities with evidence-based statuses, read-only, without exposing credentials or simulating proof.
agent: auditor
---

# /provider-check

Run the Provider Integration Proof workflow:

`workflows/provider-integration-proof.md`

## Input

Optional `$ARGUMENTS`:

- provider family or provider name, to limit the check
- known resource reference, for example a repository URL, used for an
  authorized real read

If no argument is supplied, inspect the current environment.

## Mandatory Behavior

1. Inspect available provider capabilities.
2. Verify only capabilities that can be checked with existing authorized
   access.
3. Prefer read-only checks.
4. Use supplied or otherwise explicitly authorized resources when a real
   provider read is required.
5. Avoid installing integrations, requesting paid services, changing
   provider settings, or prompting for secrets merely to make the report
   appear complete.
6. Produce a concise, evidence-backed report.

If a provider is absent, report NOT_CONFIGURED. If no suitable test
resource is available, explain why real read access could not be verified.
Do not invent a successful test.

The absence of external providers must not be treated as a failure of the
provider-agnostic core.

## Output

For each capability family:

- provider
- access method
- status
- evidence
- real or simulated
- limitations
- write capability status

## Rules

- Never print credentials or token values.
- Never execute remote writes.
- Never claim READ_VERIFIED without an actual observed read.
- Never report a simulated result as real verification.
- Never install integrations, initiate paid usage, or change provider
  settings solely to complete the report.