---
id: lean@1
schemaVersion: 1
registryId: skill-registry@1
profile: lean@1
budget:
  tokens: 8000
  enforcement: report
---

# Context profile: lean@1

Minimum discovery kernel for routing a task to a subagent. The root session
loads only the eager skills; the remaining catalog stays routed and is loaded on
demand by the dispatched subagent. This profile selects context only: it does
not activate or modify a client.

## Selection

| Kind | Skills | Note |
|---|---|---|
| eager | `task-context`, `discovery` | Loaded by the root session for routing. |
| required | `implementation-recipes` | Only for tasks that write or change code (`agents/orchestrator.md:244-248`). Not loaded in read-only tasks. |
| routed | all others | Loaded on demand by the dispatched subagent. |

## Budget

| Field | Value |
|---|---|
| tokens | 8000 |
| enforcement | `report` |

Soft, report-only. Exceeding the budget reports and refines retrieval; it never
stops the task. ECC's `mode: blocking` is rejected
(`docs/decisions/0002-agent-teams.md`, decision 3).
