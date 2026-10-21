# Mandatory Rules Registry

## Purpose

`ai-engineering` is the single source of truth for the engineering system.
This registry records every mandatory rule, the file that holds its
**canonical wording**, and the surfaces where an agent must be able to read
it.

It exists to prevent two failures observed in practice:

- a rule that exists only inside an optional Skill, so an agent that never
  loads that Skill never sees it;
- the same rule restated in many files, where the copies drift apart.

## Model

- The **canonical wording** of an always-active rule lives in `AGENTS.md`,
  which is the shared entry point and is loaded in every session.
- `AGENTS.md` is the universal entry point for every client: OpenCode loads
  it as shared instructions, Claude Code through `CLAUDE.md`, and Codex from
  the repository root. A rule that lives only in a Skill does not reach a
  client without Skills, nor an agent that never loads that Skill.
- Every other surface **references** the rule and states the operational
  instruction. Surfaces do not restate the normative wording.
- The **anchor** is a short, stable phrase used to verify mechanically that
  the rule is present on a surface. The check in
  `workflows/system-validation.md` (V2.32) verifies every row of this
  registry.

## Surfaces

| Surface | Loaded when | Role |
|---|---|---|
| `AGENTS.md` | Always | Canonical wording of every always-active rule |
| `skills/<name>/SKILL.md` `description` | Skill selection | Trigger: the words that make an agent load the Skill |
| `skills/<name>/SKILL.md` body | Skill loaded | Operational instruction for the rule |
| `commands/<name>.md` | Command invoked | Instruction for the command path |
| `workflows/<name>.md` | Workflow running | Instruction for the workflow path |
| `agents/<name>.md` | Agent acting | Instruction for the agent |

## Registry

| ID | Mandatory rule | Canonical wording | Anchor | Required surfaces |
|---|---|---|---|---|
| M-01 | TypeScript and JavaScript use arrow functions; `function` only for a generator, a `this` binding arrow syntax cannot provide, the `arguments` object, or `new.target` | `AGENTS.md` — Engineering Principles | `arrow functions` | `AGENTS.md`, `skills/implementation-recipes/SKILL.md`, `workflows/task-to-code.md`, `commands/task.md`, `agents/orchestrator.md` |
| M-02 | Interfaces, types, enums, and contracts live in a dedicated file in the folder of the layer that owns them, never mixed with runtime behavior; several types are grouped in one file only when they share the same exclusive consumer | `AGENTS.md` — Engineering Principles | `dedicated file` | `AGENTS.md`, `skills/implementation-recipes/SKILL.md`, `skills/architecture-advisor/SKILL.md`, `workflows/task-to-code.md`, `commands/task.md`, `skills/project-analysis/SKILL.md` |
| M-03 | The structural rules take precedence over a detected project convention that violates them; such a convention is a defect to correct, not to copy | `AGENTS.md` — Engineering Principles | `precedence over a detected` | `AGENTS.md`, `skills/implementation-recipes/SKILL.md`, `skills/project-analysis/SKILL.md`, `commands/task.md` |
| M-04 | The target project is the only writable scope; evidence-only paths are read-only without explicit authorization | `AGENTS.md` — Scope and Evidence Boundaries | `only writable scope` | `AGENTS.md`, `workflows/task-to-code.md`, `agents/orchestrator.md`, `skills/discovery/SKILL.md` |
| M-05 | A commit requires explicit approval in the current conversation | `AGENTS.md` — Commit Approval Policy | `explicit approval` | `AGENTS.md`, `workflows/commit.md`, `commands/commit.md`, `skills/git-workflow/SKILL.md` |
| M-06 | A new runtime or development dependency requires explicit authorization | `AGENTS.md` — Dependency Policy | `explicit authorization` | `AGENTS.md`, `workflows/task-to-code.md`, `workflows/dependency-assessment.md` |
| M-07 | Every software project has a configured formatter and a configured linter | `AGENTS.md` — Formatting and Linting Policy | `configured linter` | `AGENTS.md`, `workflows/project-validation.md` |
| M-08 | The structural compliance check runs over the change set before a task is reported as implemented; a `FAIL` is a defect of the task | `AGENTS.md` — Implementation and Validation | `output-compliance.md` | `AGENTS.md`, `workflows/task-to-code.md`, `workflows/project-validation.md`, `commands/validate.md` |
| M-09 | Correctness, safety, and validation are never traded for lower token use | `AGENTS.md` — Context and Session Resilience | `Never trade correctness` | `AGENTS.md`, `workflows/session-resilience.md`, `references/token-efficiency.md` |
| M-10 | Stack-specific decisions consult the official documentation for the detected stack and version | `AGENTS.md` — Official Documentation Grounding | `official documentation` | `AGENTS.md`, `workflows/task-to-code.md`, `skills/implementation-recipes/SKILL.md`, `skills/architecture-advisor/SKILL.md` |
| M-11 | A Context Manifest is recorded before broad exploration | `AGENTS.md` — Minimum Sufficient Context | `Context Manifest` | `AGENTS.md`, `workflows/session-resilience.md`, `workflows/task-to-code.md`, `skills/task-context/SKILL.md` |
| M-12 | A detected architecture that is absent or incoherent requires options and a human decision before implementing | `AGENTS.md` — Human Decision Gates | `Human Decision Gates` | `AGENTS.md`, `workflows/task-to-code.md`, `skills/architecture-advisor/SKILL.md` |
| M-13 | A module that consumes a contract does not re-export it; the owning file is the single public path, and a pass-through re-export is removed — in every supported ecosystem and in every form it forwards a contract | `AGENTS.md` — Engineering Principles | `pass-through re-export` | `AGENTS.md`, `skills/implementation-recipes/SKILL.md`, `skills/architecture-advisor/SKILL.md`, `workflows/task-to-code.md`, `workflows/code-audit.md`, `skills/engineering-review/SKILL.md` |
| M-14 | A new project requires the user to choose the architectural style; a recommendation is not a decision, and the structure is not created before the answer | `AGENTS.md` — Human Decision Gates | `architectural style` | `AGENTS.md`, `workflows/project-initialization.md`, `workflows/task-to-code.md`, `skills/architecture-advisor/SKILL.md`, `agents/orchestrator.md`, `commands/task.md` |
| M-15 | A new project is created step by step, each step verified before the next one starts, and every step states where the project grows next; the creation checklist is completed and reported | `AGENTS.md` — Human Decision Gates | `creation order` | `AGENTS.md`, `workflows/project-initialization.md`, `references/project-creation-order.md` |
| M-16 | A new project is created progressively with user-controlled stopping points: the scope catalog is agreed first and is a ceiling, each step ends in a checkpoint that reports the next step and waits, and the progress tracker records where the creation stopped | `AGENTS.md` — Human Decision Gates | `stopping points` | `AGENTS.md`, `workflows/project-initialization.md`, `references/project-creation-order.md`, `commands/create-project.md` |
| M-17 | Creating a project from zero requires the explicit `create-project` command; without it, the agent asks instead of creating one | `AGENTS.md` — Human Decision Gates | `create-project` | `AGENTS.md`, `commands/create-project.md`, `workflows/project-initialization.md` |

## Adding a rule

1. Write the canonical wording in `AGENTS.md`, under the section that owns
   it.
2. Add a row to this registry with its anchor and required surfaces.
3. Add the operational instruction to each required surface, referencing the
   canonical wording instead of restating it.
4. Run the V2.33 check. A rule that is not reachable from every required
   surface fails the check.

## Removing a rule

Remove the canonical wording, the registry row, and every operational
instruction that referenced it. A surface that keeps a reference to a
removed rule fails the V2.33 check.
