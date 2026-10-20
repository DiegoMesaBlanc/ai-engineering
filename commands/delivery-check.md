---
description: Read-only assessment of the CI/CD, security, and delivery readiness of the current project, with evidence-based statuses, without modifying the project or performing remote writes.
agent: auditor
---

# /delivery-check

Run the Secure Delivery Workflow:

`workflows/secure-delivery.md`

## Input

Optional `$ARGUMENTS`:

- project or source paths
- CI/CD platform or workflow files
- validation scope

If no argument is supplied, inspect the current project.

## Mandatory Behavior

1. Detect and report the project and its CI/CD environment.
2. Classify the repository as a software project or a
   documentation-only knowledge repository.
3. Assess existing CI/CD platform, workflow files, quality gates,
   security controls, and delivery safeguards.
4. Distinguish configured, executed, passed, failed, blocked, and not
   verified.
5. Report all seven items required by the Secure Delivery Workflow.
6. Never modify source files, install tools, change CI configuration,
   deploy, publish, or perform remote writes.

If no CI/CD platform is configured, report that honestly and recommend a
suitable option only when useful. Do not classify the project or the core
system as broken solely because an optional provider is absent.

Do not claim an audit was executed when the report only describes
configuration.

## Output

Produce the report defined by the workflow, including the detected stack and
CI/CD platform, the configured quality gates and their execution status, the
security controls detected, the missing controls, and what could not be
verified.

Do not restate the report structure or the status vocabulary here. The
workflow is the authoritative source; a copy in this adapter drifts from it.

Use the statuses: PASS, FAIL, BASELINE_FAILURE, BLOCKED, NOT_CONFIGURED,
NOT_RUN, NOT_VERIFIED, SIMULATED, and NOT_APPLICABLE only when a control
genuinely does not apply to the target project.

Every finding includes severity, location when available, evidence,
practical remediation, confidence, and verification limitations.

## Rules

- Read-only. Never modify the target project.
- Never print, reproduce, or store secret values.
- Never install dependencies, change CI configuration, deploy, publish,
  push, approve, or merge.
- Never claim a check was executed or passed without observed evidence.
- Never report simulated results as real security or delivery results.
- Authoring or modifying a pipeline is out of scope for this read-only
  command; it is handled inside a task under
  `workflows/secure-delivery.md` with explicit authorization.