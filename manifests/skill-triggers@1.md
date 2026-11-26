---
id: skill-triggers@1
schemaVersion: 1
registryId: skill-registry@1
defaultProfile: lean@1
triggerCount: 15
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
| task-analysis | requisito, tarea, issue, acceptance, task context, ticket | lean@1 | task-context | root | underneath | |
| project-discovery | repositorio, stack, arquitectura actual, descubrir, inventory, project analysis | lean@1 | discovery, project-analysis, stack-detection | root | underneath | |
| complexity | complejidad, trivial, low, medium, high, critical, estimar esfuerzo | lean@1 | complexity-advisor | root | parallel | |
| architecture | arquitectura, patrón, diseño de código, estructura de carpetas, solid, contract | full@1 | architecture-advisor, api-contract-design, repository-strategy | root | parallel | |
| design-review | revisión de diseño, pre-code, design review, propuesta | full@1 | design-analysis | external-auditor | parallel | |
| security-audit | auditoría de seguridad, security audit, owasp audit, revisión de seguridad | full@1 | security-review, vulnerability-catalog, best-practices | external-auditor | parallel | |
| security | seguridad, security, vulnerabilidad, owasp, amenaza | full@1 | security-review, vulnerability-catalog | security | parallel | |
| engineering-review | code review, revisar cambios, calidad, engineering review, pull request review, revisión de pr, lint, typecheck | full@1 | engineering-review, review-memory | reviewer-pr | parallel | |
| gate-audit | auditoría de gates, verificar gates, gate evidence, verificación independiente de gates | full@1 | testing-strategy | external-auditor | parallel | |
| testing | test, prueba, testing, cobertura, tdd | full@1 | testing-strategy, test-driven-development | backend | parallel | |
| compliance | compliance, structural, output-compliance, verificación estructural | full@1 | — | external-auditor | underneath | |
| pull-request | pull request, pr, merge request, descripción de pr, generate-pr, contenido de pr | lean@1 | pull-request | writer-pr | underneath | |
| web-audit-a11y | accesibilidad, a11y, wcag | full@1 | accessibility | external-auditor | parallel | optional |
| web-audit-performance | performance, velocidad, load time, core web vitals, lcp, cls | full@1 | performance, core-web-vitals | external-auditor | parallel | optional |
| web-audit-seo | seo, search engine, meta tags, sitemap | full@1 | seo | external-auditor | parallel | optional |

## Notes

- `implementation-recipes` is in no `load` list: the orchestrator adds it only
  for tasks that write code (`manifests/skill-registry@1.md`,
  `docs/decisions/0003-domain-subagents.md`, decision 5). This replaces the old
  citation of the superseded `docs/decisions/0002-agent-teams.md`.
- A `root` delegate means the orchestrator keeps the work. A named delegate is
  one of the eight subagents (the six domains, `writer-pr` and `reviewer-pr`),
  or `external-auditor` for an independent fresh-context audit
  (`docs/decisions/0003-domain-subagents.md`).
- PR preparation and PR review are separate surfaces: `/generate-pr` loads
  `writer-pr`, which writes copy-ready content to the neutral state root;
  `/review-pr` loads `reviewer-pr`, which returns copy-ready findings and never
  writes. They are commands bound to the orchestrator, not substitutes for the
  subagents that carry the contract.
- An optional trigger loads its skills only when the approved requirement names
  that dimension. A missing tool or an unexposed capability is reported
  `NOT VERIFIED`, never silently skipped.
- No trigger stops a task. A profile budget overrun reports and refines
  retrieval (`docs/decisions/0002-agent-teams.md`, decision 3).
