# Domain Execution Protocol

## Purpose

Define how a domain subagent executes work so that seniority follows the skill
being used, not the agent that was dispatched. The orchestrator and the primary
`coder` surface dispatch six domain subagents; each subagent loads skills on
demand and changes seniority at every step. This reference is the shared
behavior of all six definitions, so the rule is authored once.

## Model

- `agents/orchestrator.md` is the composition root. It alone binds the client
  tools and selects the concrete adapters, and it selects and dispatches
  subagents. `agents/coder.md` is the primary implementation surface: it writes
  the target project and dispatches the read-only domain subagents and
  `explore` for findings. It is not a second composition root and does not bind
  client tools. A task leader may request a dispatch through the composition
  root. The canonical id is `explore`; dispatch it as `Explore` in Claude.
- A domain subagent is a thin retrieval and execution unit. It owns a domain,
  an allowlist of skills, and this protocol. It does not own a persona.
- Skills are **shared capabilities**, not domain-owned assets. The `target`
  column in `manifests/skill-registry@1.md` is the orchestrator's default
  route. It does not prevent another domain from loading the same skill.
- Seniority is resolved per step from `manifests/skill-seniority@1.md`. The
  subagent adopts it for that step and records it in its returned checklist.

## Worked example

A `mobile` task needs a folder architecture. Its allowlist names
`architecture-advisor`. The subagent loads that skill at seniority
`architect`, follows it to `references/folder-architectures.md`, and uses the
React Native tree there (the file defines React Native feature folders and the
`android/` and `ios/` root projects). It then loads `dependency-management` at
seniority `tech-lead` and `testing-strategy` at seniority `staff-reviewer`.
`implementation-recipes` is the one mandatory skill for code changes: `coder`
loads it at seniority `senior-engineer` when it writes code; a read-only domain
returns the plan without loading it. No other domain definition changes. The same
sequence applies to `devops`, `frontend`, `backend`, `data`, or `security` with
their own skill allowlists.

## Seniority ladder

| Seniority | Owns | Does not own |
|---|---|---|
| `architect` | Structure, contracts, patterns, folder trees, schema shape | Unapproved architecture changes |
| `tech-lead` | Rigor, sequencing, dependency package assembly, delivery | Installation or commit |
| `senior-engineer` | Implementation under the quality baseline | Weakening gates for speed |
| `staff-reviewer` | Evidence-based findings and gate execution | Editing the code it reviews |

## Phases

A domain subagent runs the phases the task needs, in order. A phase may be
skipped with a recorded reason. A phase that changes a material requirement or
approved design stops the subagent and returns the reason to the orchestrator.

| Phase | Id | Seniority | Entry | Exit artifact |
|---|---|---|---|---|
| 1 Understand | `understand` | `tech-lead` | Task and evidence available | Normalized requirements, detected stack, rigor |
| 2 Architect | `architect` | `architect` | Requirements understood | Structure decision and pattern selection with forces |
| 3 Prepare | `prepare` | `tech-lead` | Structure decided | Branch plan and dependency installation package proposal |
| 4 Build | `build` | `senior-engineer` | Package approved by the human | Implementation plan or code-ready change set with validation commands |
| 5 Validate | `validate` | `staff-reviewer` | Change set exists | Gate results with evidence and findings by severity |
| 6 Deliver | `deliver` | `tech-lead` | Gates green | Copy-ready delivery content and returned checklist |

## Step contract

For every step the subagent writes one checklist line before moving on:

```
[ x ] F2 · architect · architecture-advisor → structure decision (pattern set + forces)
```

A line names the phase, the seniority resolved from the manifest, the skill
loaded, and the artifact produced. A step that produces no artifact does not
get a line. The returned checklist carries every line in execution order.

## Boundaries

- A domain subagent is `readOnly`. It proposes artifacts and returns evidence;
  it never writes to the target repository, installs packages, commits,
  publishes, or writes to the home or client-configuration directories.
- The `tech-lead` step on dependencies produces an installation package only.
  `workflows/dependency-assessment.md` requires explicit human authorization
  before anything is installed; the orchestrator performs the authorized
  installation.
- The `staff-reviewer` step reports findings with `file:line` and a severity.
  It never edits the code under review. An applicable `NOT VERIFIED` blocks.
- Every external call carries a timeout enforced by the client tool. A client
  with no enforced timeout is reported `NOT VERIFIED`, never claimed as bound.
- Scope and evidence-only paths come from the orchestrator's task context. A
  path supplied as reference stays read-only.

## Completion

A domain subagent returns `findings`, `evidence`, `verdict` where applicable,
the produced `artifacts`, and its `checklist`, then ends with `TERMINADO` and
the result or `DETENIDO` with the reason. A result without its checklist is
incomplete and the orchestrator asks for it before reporting the phase.

## Validation

Check a domain definition against this protocol: its allowlist contains only
real skill ids, every step in its sample trace resolves a seniority row, no
step grants a write capability to a reader, and the returned checklist format
matches the step contract above. Verify the manifest and the registry name the
same skill set in both directions.
