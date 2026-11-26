---
name: devops-engineering
description: Author CI/CD pipelines, environments, and infrastructure so one built artifact is promoted across environments through explicit gates and digest-verified approvals.
---

# DevOps Engineering

## Purpose

Carry the delivery and infrastructure constraints that apply when a change
touches a pipeline, deployment, environment, container, or operational
configuration. This skill is a shared capability and may be loaded by any
domain whose change reaches delivery.

## When to load

Load when the task touches CI/CD, deployment, infrastructure as code,
containers, environments, secrets in delivery, or artifact promotion.

## Procedure

1. Read `workflows/secure-delivery.md` in full and follow it. In read-only
   assessment mode, produce findings only and implement nothing. In authoring
   mode, build one artifact and promote that same artifact across local,
   development, QA, staging, and production.
2. Identify the built artifact by digest. Every delivery gate and the deploy
   step consume the artifact by that digest. Never check out a dispatched ref
   and then deploy or publish a re-derived source.
3. Bind an approval to the artifact: the approval names the promoted digest,
   the gate compares it with the artifact about to deploy, and the approver is
   not the actor that dispatched the run. Reviewers of a platform environment
   are outside the repository and reported `NOT VERIFIED`.
4. Give every network call an explicit timeout on its own statement. Every
   workflow step that calls the network declares its own `timeout-minutes`; a
   job timeout does not bound a step.
5. Never let a gate fail silently. A gate step with `|| true` or
   `continue-on-error: true` is a defect. A failing gate is handled by a bounded
   fix or a rollback, never by weakening the gate.
6. Never combine a wildcard origin with credentials. Allowed origins are an
   explicit list. Never grant a new permission, role, scope, or access rule the
   approved requirement does not name.
7. Keep development tools and introspection endpoints behind an environment
   check that is false in production. Never send a credential or session data
   over cleartext outside an exact local host. Parse the URL; reject userinfo
   and an empty host.
8. Do not downgrade a dependency to remediate an advisory. Apply the dependency
   policy through `dependency-management` at `tech-lead` seniority.
9. Require explicit approval for production and any remote operation. The
   orchestrator performs remote writes after that approval; this skill only
   produces the pipeline and the approval question.

## Boundaries

- No production or remote write from this skill, including a test deployment.
- No credential in the repository or in a generated report.
- No wildcard credential origin and no unconditional debug route.
- No gate bypass and no silent failure.

## Output

Return the pipeline or assessment, the artifact digest and gate list, the
approval question, the evidence for each control, and the exact checks not run.
End with `TERMINADO` or `DETENIDO` with the reason.

## References

- `workflows/secure-delivery.md`, `workflows/commit.md`
- `workflows/output-compliance.md` (`GATE-002`, `SAFE-002`, `DELIVERY-001`,
  `DELIVERY-002`, `AUTH-002`)
- `dependency-management`, `security-review`
