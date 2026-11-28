# Context: dev mode

Load this bundle for subagents working in development mode: analysis, design,
and implementation of a task that is not yet a finished change set. It is a
small, explicit context — the root session sends this, not the whole catalog.

## Always active (kernel pointers)

- `AGENTS.md` — engineering principles and mandatory guardrails. Never trim,
  never restate; follow it as the universal entry point.
- `workflows/task-to-code.md` — the canonical phase sequence. Read it before
  proposing implementation steps.
- `references/folder-architectures.md` — where code, types, contracts, and
  tests belong.

## On demand by capability

- Stack-specific implementation guidance: `skills/implementation-recipes/SKILL.md`
  (only for tasks that write or change code).
- Task understanding: `skills/task-context/SKILL.md`.
- Complexity: `skills/complexity-advisor/SKILL.md`.
- Architecture and contracts: `skills/architecture-advisor/SKILL.md`,
  `skills/api-contract-design/SKILL.md`.
- Project and stack discovery: `skills/discovery/SKILL.md`,
  `skills/project-analysis/SKILL.md`, `skills/stack-detection/SKILL.md`.

## Boundaries

- Read-only subagents do not edit repository files or write to the home
  directory; they return findings.
- The composition root is `agents/orchestrator.md`; it alone binds the client
  tools and selects the concrete adapters. `coder` is the primary implementation
  surface and may dispatch the read-only domain subagents; it is a primary
  agent outside the v1 subagent contract and is not a second composition root.
  A subagent does not dispatch another subagent unless its contract lists
  `delegate`.
- Budgets are report-only. Exceeding a budget reports and refines retrieval;
  it never stops the task.
