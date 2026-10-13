# Secure Delivery Workflow

## Purpose

Assess, configure, and improve CI/CD, security, and delivery controls in
target software projects, using the project's actual stack, existing
tooling, risk, and the authorized delivery platform.

This workflow is evidence-driven, portable, modular, and low-cost. It is
read-only by default.

This workflow is the canonical definition of delivery validation.
`commands/delivery-check.md` is the client adapter that delegates here.

## Boundaries

Distinguish between these operation modes:

- read-only assessment of an existing project
- recommendations for missing controls
- authorized implementation of configuration changes
- execution of tests, scans, builds, and pipeline validation
- publication and deployment, which require their own explicit
  authorization

A read-only assessment must not silently turn into implementation work.

If missing controls need implementation, include them in the task plan,
preserve unrelated changes, and follow the authorization and branch
policies in AGENTS.md.

Never perform remote writes, deployments, or publication from a
read-only assessment.

## Phase 1 — Detect the Project and CI/CD Environment

Classify the repository first:

- software project with application source code
- documentation-only knowledge repository

For a software project, determine when applicable:

- project type and supported stacks
- package manager, build system, dependency manifests, and lockfiles
- existing CI/CD platform and workflow configuration (GitHub Actions,
  GitLab CI, Azure Pipelines, or another detected system)
- existing testing, type-checking, linting, formatting, and build commands
- existing security analysis, dependency auditing, and secret-scanning
  controls
- container, infrastructure-as-code, mobile signing, deployment, or
  release components
- existing environments, approval gates, and release conventions
- relevant project-specific instructions and documentation

Prefer extending an existing working pipeline over replacing it.

Do not assume that every project needs containers, infrastructure
scanning, mobile signing, deployment workflows, or every possible
security tool.

Do not introduce GitHub Actions merely because it is familiar when the
project already uses another provider.

For documentation/configuration-only repositories, do not impose
application CI/CD or create a package.json solely to satisfy
software-project checks.

## Phase 2 — CI Quality Gates

For target software projects, ensure delivery validation covers the
appropriate existing checks:

- formatter and its non-mutating format check
- linter
- relevant unit and integration tests
- type checking when applicable
- build or compilation
- relevant contract, end-to-end, or migration checks when warranted by
  scope and risk

The mandatory formatter/linter policy for software projects remains in
force. Missing tooling must trigger the established setup workflow
(AGENTS.md and `workflows/project-validation.md`) rather than being
reported as `NOT_APPLICABLE`.

Run focused checks first and broaden validation according to risk,
evidence, project policy, and available resources.

Reuse the project's existing scripts and package manager. Do not blindly
overwrite scripts, lockfiles, configuration, or CI workflows.

A successful build alone is not sufficient evidence that all quality
gates passed.

## Phase 3 — Security Checks

Select security controls according to the actual application, stack,
risk, and existing tooling. Do not force every check onto every project.

### Secrets

- Exposed credentials or sensitive tokens in source, configuration, logs,
  and delivery definitions.
- Use of approved secret injection mechanisms.
- Risk of leaking secrets in pipeline output or artifacts.

Never print, reproduce, or store discovered secret values. Redact
sensitive evidence and report only the location and type of concern
needed to remediate it.

### Dependencies and Supply Chain

- existing dependency audit capabilities
- known vulnerable dependencies, when data is available
- lockfile and dependency reproducibility
- unnecessary or unpinned dependencies and delivery actions where
  applicable
- relevant artifact integrity and provenance controls

Distinguish the presence of an audit tool from an actual successful
vulnerability scan. Report a clean dependency audit only when the audit
command executed successfully and its vulnerability database was
available.

### Static and Infrastructure Analysis

- existing SAST or language-specific security analyzers
- injection, authorization, and configuration risks relevant to the
  changed code
- container image scanning when containers exist
- infrastructure-as-code checks when infrastructure definitions exist
- mobile signing and release-secret handling when applicable

Prefer established, maintained, open-source tools compatible with the
project's stack. Reuse already approved organizational tools when
available.

Do not install paid services, activate paid plans, introduce billable
usage, or install additional MCP integrations without explicit
authorization.

A read-only assessment must not install dependencies or alter project
configuration merely to improve its own report. When implementation is
authorized, set up missing tools as part of the scoped work and report
all changes.

## Phase 4 — Secure CI/CD Design Principles

When assessing or modifying pipelines, consider:

- least-privilege permissions
- separation of validation, packaging, publishing, and deployment
- secret isolation and masking
- safe handling of untrusted pull-request code
- dependency and executable pinning appropriate to the platform
- concurrency, cancellation, caching, and artifact retention
- required quality gates before release
- protected production environments and explicit approval gates
- reproducible builds and auditable release inputs
- rollback or recovery strategy when meaningful
- avoiding unnecessary CI minutes, repeated installs, and paid resources

Apply these principles proportionately. Do not redesign a pipeline solely
to satisfy a theoretical checklist.

Never weaken required checks or security controls to obtain a green
pipeline.

Do not run production deployments, publish releases, push images, change
remote environment settings, or modify branch protections without
explicit authorization for that specific operation.

A declaration that a pipeline is secure or production-ready requires
supporting evidence; a static review alone is not proof of runtime
behavior.

## Phase 5 — Execution and Authorization

Read-only assessment is the default. Do not silently switch to
implementation.

Execution of tests, scans, builds, and pipeline validation:

- is allowed when authorized and appropriate
- uses existing project commands
- records the command, result, exit code, and evidence
- never weakens or skips configured gates for a green result

Implementation of configuration changes:

- belongs to the scoped task plan
- preserves unrelated changes
- follows the branch and authorization policies in AGENTS.md

Publication and deployment:

- requires its own explicit authorization for that specific operation
- is never performed by a read-only assessment

Never create commits, push branches, create or publish PRs, approve or
merge, publish review comments, or update remote tasks.

## Reporting

### Statuses

Use at least:

- PASS
- FAIL
- BASELINE_FAILURE: a pre-existing failure unrelated to the current
  change
- BLOCKED: a concrete limitation prevented the check
- NOT_CONFIGURED
- NOT_RUN
- NOT_VERIFIED
- SIMULATED
- NOT_APPLICABLE: only when a control genuinely does not apply to the
  target project

Missing mandatory formatter or linter tooling in scope is `SETUP_REQUIRED`
under the AGENTS.md policy, never `NOT_APPLICABLE`.

### Findings

Every finding includes:

- severity proportionate to demonstrated impact
- location when available
- evidence
- practical remediation
- confidence
- verification limitations

Do not label a security control as passed solely because a configuration
file exists.

Do not report a clean dependency audit when the audit could not run or
its vulnerability database was unavailable.

Do not treat simulated scans as real security results.

Do not turn an unknown state into PASS.

### Output

Use the `/delivery-check` report format described in
`commands/delivery-check.md`:

1. Project type and detected stack.
2. Existing CI/CD platform and relevant workflow files.
3. Configured quality gates and their actual execution status.
4. Security controls detected and the scope of checks actually executed.
5. Delivery, artifact, environment, and deployment safeguards where
   relevant.
6. Missing controls, evidence-supported findings, and prioritized
   recommendations.
7. What could not be verified and why.

## Safety

- Read-only by default.
- Never print, reproduce, or store secret values.
- Never install tools, modify configuration, or alter project files
  during a read-only assessment.
- Never deploy, publish, push, approve, merge, or modify remote
  environments without explicit authorization for that specific
  operation.
- Never introduce paid usage, billable integrations, or MCP integrations
  without explicit authorization.
- Report checks as NOT VERIFIED when the available tools cannot
  establish their result.