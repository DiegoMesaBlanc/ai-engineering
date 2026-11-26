# Context: research mode

Load this bundle for subagents performing technical investigation, estimation,
or source analysis — tasks whose outcome is a decision, a report, or a durable
document rather than application code. A research outcome authorizes no file
change in any project (M-91); an estimate is the single M-91 exception, permitted
to write the estimation document required by `workflows/estimation.md` as the only
path in the approved manifest, with any other changed file a `SCOPE-001` `FAIL`.

## Always active (kernel pointers)

- `AGENTS.md` — engineering intent modes and scope and evidence boundaries.
- `references/official-documentation.md` — the shared index of authoritative
  technical documentation; consult on demand, never load the whole site.
- `workflows/session-resilience.md` — token and context efficiency; the
  minimum sufficient context and the stop rules.

## On demand by capability

- Task and requirements: `skills/task-context/SKILL.md`.
- Repository and stack: `skills/discovery/SKILL.md`,
  `skills/project-analysis/SKILL.md`, `skills/stack-detection/SKILL.md`.
- Estimation: `workflows/estimation.md` (whole projects only; never implement).
- Architecture options: `skills/architecture-advisor/SKILL.md`,
  `skills/repository-strategy/SKILL.md`.
- Iterative retrieval: `skills/knowledge-graph/SKILL.md`,
  `skills/context-compression/SKILL.md`.

## Boundaries

- Research authorizes no source change; the approved manifest is empty. A
  changed file is a scope failure (`SCOPE-001`).
- Evidence-only paths are read-only. A reference example is never an implicit
  task.
- A claim is kept only with the evidence it cites; a count, version, or
  approval is re-derived from the enumerated source, not from memory.
- Budgets are report-only; exceeding one refines retrieval, never stops the
  investigation.
