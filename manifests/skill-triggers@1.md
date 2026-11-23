---
id: skill-triggers@1
schemaVersion: 1
registryId: skill-registry@1
defaultProfile: lean@1
triggerCount: 13
---

# Skill triggers: skill-triggers@1

Proxy that decides which context a task loads before any skill or subagent is
dispatched. The root session reads this small index first and resolves only the
skills it needs; the full catalog is never loaded eagerly.

A trigger matches by substring against the task text, case-insensitive. The
first matching entry in order wins; a task with no match keeps the lean kernel.
Skills in `load` are ids from `manifests/skill-registry@1.md`.

## Triggers

| id | match (any substring) | profile | load | delegate | mode | flags |
|---|---|---|---|---|---|---|
| task-analysis | requisito, tarea, issue, acceptance, task context, ticket | lean@1 | task-context | analyst-task-context | underneath | |
| project-discovery | repositorio, stack, arquitectura actual, descubrir, inventory, project analysis | lean@1 | discovery, project-analysis, stack-detection | analyst-project | underneath | |
| complexity | complejidad, trivial, low, medium, high, critical, estimar esfuerzo | lean@1 | complexity-advisor | advisor-complexity | parallel | |
| architecture | arquitectura, patrón, diseño de código, estructura de carpetas, solid, contract | full@1 | architecture-advisor, api-contract-design, repository-strategy | advisor-architecture, designer-contracts | parallel | |
| design-review | revisión de diseño, pre-code, design review, propuesta | full@1 | design-analysis | auditor-design | parallel | |
| engineering-review | code review, revisar cambios, calidad, engineering review, lint, typecheck | full@1 | engineering-review, best-practices, database-engineering, internationalization | reviewer-engineering | parallel | |
| security | seguridad, security, vulnerabilidad, owasp, auditoría de seguridad | full@1 | security-review, vulnerability-catalog | reviewer-security | parallel | |
| testing | test, prueba, testing, cobertura, tdd | full@1 | testing-strategy, test-driven-development | reviewer-testing | parallel | |
| compliance | compliance, structural, output-compliance, verificación estructural | full@1 | — | compliance-runner | underneath | |
| pull-request | pull request, pr, merge request, descripción de pr, generate-pr | lean@1 | pull-request | writer-pr | underneath | |
| web-audit-a11y | accesibilidad, a11y, wcag | full@1 | accessibility | audit-a11y | parallel | optional |
| web-audit-performance | performance, velocidad, load time, core web vitals, lcp, cls | full@1 | performance, core-web-vitals | audit-performance, audit-cwv | parallel | optional |
| web-audit-seo | seo, search engine, meta tags, sitemap | full@1 | seo | audit-seo | parallel | optional |

## Notes

- `implementation-recipes` is in no `load` list: the orchestrator adds it only
  for tasks that write code (`docs/decisions/0002-agent-teams.md`, decision 4).
- An optional trigger resolves only if its optional subagent projections were
  created in Phase C4; otherwise the orchestrator reports the capability as
  `NOT VERIFIED` and proceeds without it.
- No trigger stops a task. A profile budget overrun reports and refines
  retrieval (`docs/decisions/0002-agent-teams.md`, decision 3).
