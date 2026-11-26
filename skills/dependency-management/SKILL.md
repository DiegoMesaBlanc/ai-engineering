---
name: dependency-management
description: Assemble the evidence-based installation package a task needs — exact names, versions, scope, reason, dependencies brought, and alternatives — for human approval before anything is installed.
---

# Dependency Management

## Purpose

Produce the installation package that `workflows/dependency-assessment.md`
requires before implementation. This skill is the invocable surface of that
workflow. It does not replace or restate the policy.

## When to load

Load at the `prepare` phase, at `tech-lead` seniority, whenever a task may need
a runtime or development dependency. Also load when a dependency remediation
is proposed, including an audit suggestion to downgrade.

## Procedure

1. Read `workflows/dependency-assessment.md` in full and follow its phases.
2. Inspect manifests, lockfiles, installed direct and relevant transitive
   dependencies, the package manager, and existing abstractions. Reading is
   the only action allowed in this step.
3. Decide exactly one outcome: no new dependency, a compatible installed
   dependency, or a new dependency justified by evidence.
4. If a package is needed, assemble the whole set as one installation package.
   The unit of approval is the package, never a single library at a time.
5. For every entry state the exact name, exact version, runtime or development
   scope, reason, what it brings, and the alternatives including doing nothing.
6. Reject any remediation that downgrades a direct dependency. A vulnerable
   transitive package is fixed by updating the direct dependency that brings
   it, by a documented override of that one package, or by recording the
   blocker with the fixed version.
7. Hand the package to the orchestrator as a proposal. Installation happens
   only after explicit human approval of that exact package and scope.

## Boundaries

- A recommendation is not an installation. Never run an installer, package
  manager write command, or lockfile write.
- Never initiate paid usage or add a paid dependency without authorization.
- Never expose credentials, registry tokens, or private registry URLs.
- Mandatory formatter and linter setup follows its own authorization in
  AGENTS.md and is declared separately from the task package.
- The formatter and linter are configured from the project's own files; no
  custom rule file or checker script is added to the target project.

## Output

Return the chosen outcome, the installation package table when one is needed,
the evidence used to inspect the project, and the approval question. End with
`TERMINADO` or `DETENIDO` with the reason.

## References

- `workflows/dependency-assessment.md`
- `AGENTS.md` ("Dependency Policy", "Formatting and Linting Policy")
