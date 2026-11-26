---
id: skill-seniority@1
schemaVersion: 1
registryId: skill-registry@1
protocolId: domain-execution-protocol
seniorities: architect, tech-lead, senior-engineer, staff-reviewer
skillCount: 34
enumeratedOn: "2026-11-23"
---

# Skill seniority: skill-seniority@1

Data consumed by the six domain subagents. It maps each skill to the seniority
a subagent adopts while executing that skill. Seniority belongs to the step,
not to the subagent: the same subagent is an `architect` at one step and a
`tech-lead` at the next.

The orchestrator keeps routing in `manifests/skill-registry@1.md`. A registry
`target` is the default route, not an ownership lock. Any domain may load any
skill named in its own `context.skills` allowlist, including a skill whose
default route names another domain. That is what lets `mobile` load
`architecture-advisor` and `references/folder-architectures.md` for a React
Native folder tree.

This file is data, not executable behavior. No client projection reads it; the
canonical subagent bodies instruct the agent to resolve it at retrieval time.

## Rules

1. One row per skill. A skill has one seniority across every domain.
2. Resolve the row before loading the skill. Adopt that seniority for that step
   only, then release it when the step ends.
3. Record every step in the returned checklist as
   `[ x ] F<phase> · <seniority> · <skill> → <artifact>`.
4. A skill loaded in a phase other than its `phase` column keeps its seniority.
   The agent records the deviation and its reason in the same step line.
5. Phase ids are those in `references/domain-execution-protocol.md`.
6. Adding or removing a skill updates this table, `skillCount`, and
   `manifests/skill-registry@1.md` in the same change set.

## Catalog

| skill | seniority | phase | reason |
|---|---|---|---|
| accessibility | staff-reviewer | validate | Produces WCAG 2.2 evidence against a surface. |
| api-contract-design | architect | architect | Defines boundaries, schemas, and versioning. |
| architecture-advisor | architect | architect | Decides structure, patterns, and folder trees. |
| best-practices | senior-engineer | build | Carries the security and compatibility baseline into code. |
| complexity-advisor | tech-lead | understand | Sets proportional rigor before work starts. |
| context-compression | senior-engineer | build | Keeps bounded context during long work. |
| core-web-vitals | staff-reviewer | validate | Reports measured LCP, INP, and CLS. |
| database-engineering | architect | architect | Models schema, migrations, indexes, and query strategy. |
| dependency-management | tech-lead | prepare | Assembles the installation package for human approval. |
| design-analysis | architect | architect | Translates design artifacts into engineering requirements. |
| devops-engineering | senior-engineer | build | Authors pipelines and infrastructure under delivery gates. |
| discovery | tech-lead | understand | Establishes project state and evidence. |
| engineering-baseline | tech-lead | prepare | Adopts the onboarding baseline before code. |
| engineering-review | staff-reviewer | validate | Applies the Staff-bar correctness and design review. |
| frontend-design | senior-engineer | build | Exercises deliberate visual craft in the client. |
| git-workflow | tech-lead | prepare | Prepares the branch and the atomic change set. |
| implementation-recipes | senior-engineer | build | Carries the non-negotiable quality baseline for code. |
| internationalization | senior-engineer | build | Builds locale-aware text, direction, and formatting. |
| knowledge-graph | senior-engineer | build | Maps a codebase before reading whole files. |
| mobile-engineering | senior-engineer | build | Applies mobile lifecycle, platform, and storage constraints. |
| performance | staff-reviewer | validate | Reports before-and-after measurements of named paths. |
| project-analysis | tech-lead | understand | Determines architecture and conventions from evidence. |
| pull-request | tech-lead | deliver | Produces copy-ready delivery content. |
| replica | senior-engineer | build | Runs the clean-room replication workflow. |
| repository-strategy | architect | architect | Decides topology and ownership boundaries. |
| review-memory | senior-engineer | build | Carries review state across revisions. |
| security-review | staff-reviewer | validate | Returns independent weakness verdicts. |
| seo | staff-reviewer | validate | Produces search-visibility evidence. |
| stack-detection | tech-lead | understand | Detects stack and versions before decisions. |
| task-context | tech-lead | understand | Normalizes requirements and acceptance criteria. |
| test-driven-development | staff-reviewer | validate | Writes behavior-first regression coverage. |
| testing-strategy | staff-reviewer | validate | Selects and executes the applicable gates. |
| ui-ux-design | senior-engineer | build | Applies UX rules by platform. |
| vulnerability-catalog | staff-reviewer | validate | Supplies category evidence for security findings. |

## Maintenance

Enumerated on 2026-11-23 from `manifests/skill-registry@1.md` plus the three
skills introduced with this manifest. Verify an exact id match against the
registry in both directions: no missing and no extra rows.
