---
id: full@1
schemaVersion: 1
registryId: skill-registry@1
profile: full@1
budget:
  tokens: 20000
  enforcement: report
---

# Context profile: full@1

Complete skill set for deep audit and design tasks. Loaded only when the task's
rigor or the delegation target requires the full catalog. The root session still
routes through the registry rather than loading everything eagerly.

## Selection

| Kind | Skills | Note |
|---|---|---|
| eager | `task-context`, `discovery`, `implementation-recipes` | `implementation-recipes` only for code-changing tasks (`agents/orchestrator.md:244-248`). |
| required | `implementation-recipes` | Only for tasks that write or change code. Not loaded in read-only tasks. |
| routed | all others, on demand by capability | The subagent resolves only what it needs. |

## Budget

| Field | Value |
|---|---|
| tokens | 20000 |
| enforcement | `report` |

Soft, report-only. Exceeding the budget reports and refines retrieval; it never
stops the task. ECC's `mode: blocking` is rejected
(`docs/decisions/0002-agent-teams.md`, decision 3).
