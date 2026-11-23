---
id: skill-registry@1
schemaVersion: 1
source: skills/*/SKILL.md
enumeratedOn: "2026-11-21"
skillCount: 31
---

# Skill registry: skill-registry@1

Inventory of the 31 skills in `skills/` and their routing targets. A target is
either a subagent that loads the skill on demand or `root` when the orchestrator
keeps it. This registry is data, not executable behavior: the orchestrator reads
it to decide delegation and never loads the whole catalog eagerly.

The path of every row is `skills/<id>/SKILL.md`.

## Registry

| id | target | flags |
|---|---|---|
| accessibility | subagent:audit-a11y | optional |
| api-contract-design | subagent:designer-contracts | |
| architecture-advisor | subagent:advisor-architecture | |
| best-practices | subagent:reviewer-engineering | |
| complexity-advisor | subagent:advisor-complexity | |
| context-compression | root | |
| core-web-vitals | subagent:audit-cwv | optional |
| database-engineering | subagent:reviewer-engineering | |
| design-analysis | subagent:analyst-task-context | |
| discovery | subagent:analyst-project | |
| engineering-baseline | root | |
| engineering-review | subagent:reviewer-engineering | |
| frontend-design | root | |
| git-workflow | root | |
| implementation-recipes | root | required for code changes |
| internationalization | subagent:reviewer-engineering | |
| knowledge-graph | root | |
| performance | subagent:audit-performance | optional |
| project-analysis | subagent:analyst-project | |
| pull-request | subagent:writer-pr | |
| replica | root | |
| repository-strategy | subagent:advisor-architecture | |
| review-memory | root | |
| security-review | subagent:reviewer-security | |
| seo | subagent:audit-seo | optional |
| stack-detection | subagent:analyst-project | |
| task-context | subagent:analyst-task-context | |
| test-driven-development | subagent:reviewer-testing | |
| testing-strategy | subagent:reviewer-testing | |
| ui-ux-design | root | |
| vulnerability-catalog | subagent:reviewer-security | |

## Maintenance

When a skill is added or removed, update this table and `skillCount` in the
frontmatter, then verify that the ids match the directories on disk exactly —
no missing and no extra entries. Enumerated on 2026-11-21 from
`find skills -mindepth 1 -maxdepth 1 -type d`.
