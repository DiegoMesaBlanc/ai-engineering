---
id: skill-registry@1
schemaVersion: 1
source: skills/*/SKILL.md
enumeratedOn: "2026-11-23"
skillCount: 34
---

# Skill registry: skill-registry@1

Inventory of the 34 skills in `skills/` and their routing targets. A target is
either a subagent that loads the skill on demand or `root` when the orchestrator
keeps it. A target is the orchestrator's default route, not an ownership lock:
`references/domain-execution-protocol.md` lets any domain subagent load a skill
named in its own `context.skills` allowlist. Audit defaults point to
`external-auditor`; PR content defaults to `writer-pr`, and PR review defaults
to `reviewer-pr`. This registry is data, not executable behavior: the
orchestrator reads it to decide delegation and never loads the whole catalog
eagerly.

The path of every row is `skills/<id>/SKILL.md`.

## Registry

| id | target | flags |
|---|---|---|
| accessibility | subagent:external-auditor | audit default; domains load via allowlist |
| api-contract-design | subagent:backend | |
| architecture-advisor | root | shared by all six domains |
| best-practices | subagent:external-auditor | audit default; security domain loads via allowlist |
| complexity-advisor | root | |
| context-compression | root | |
| core-web-vitals | subagent:external-auditor | audit default |
| database-engineering | subagent:data | |
| dependency-management | root | |
| design-analysis | subagent:frontend | |
| devops-engineering | subagent:devops | |
| discovery | root | |
| engineering-baseline | root | |
| engineering-review | subagent:reviewer-pr | PR review default |
| frontend-design | subagent:frontend | |
| git-workflow | subagent:devops | |
| implementation-recipes | root | required for code changes |
| internationalization | subagent:frontend | |
| knowledge-graph | root | |
| mobile-engineering | subagent:mobile | |
| performance | subagent:external-auditor | audit default |
| project-analysis | root | |
| pull-request | subagent:writer-pr | PR content default |
| replica | subagent:frontend | |
| repository-strategy | root | shared by four domains (frontend, data, devops, security) |
| review-memory | subagent:reviewer-pr | PR review default |
| security-review | subagent:external-auditor | audit default |
| seo | subagent:external-auditor | audit default |
| stack-detection | root | |
| task-context | root | |
| test-driven-development | subagent:backend | |
| testing-strategy | subagent:external-auditor | independent gate audit default |
| ui-ux-design | subagent:frontend | |
| vulnerability-catalog | subagent:external-auditor | audit default |

## Maintenance

When a skill is added or removed, update this table and `skillCount` in the
frontmatter, then verify that the ids match the directories on disk exactly —
no missing and no extra entries. Enumerated on 2026-11-23 from
`find skills -mindepth 1 -maxdepth 1 -type d | wc -l`, which returned 34.
