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
| M-05 | A commit is created only after the system presents the change set for review and the user answers an approval question in the current conversation; an instruction to commit in the prompt, task, or command arguments is never explicit approval | `AGENTS.md` — Commit Approval Policy | `explicit approval` | `AGENTS.md`, `workflows/commit.md`, `commands/commit.md`, `skills/git-workflow/SKILL.md`, `workflows/task-to-code.md`, `commands/task.md`, `agents/orchestrator.md`, `workflows/system-validation.md`, `references/system-workflow-regression-matrix.md` |
| M-06 | A new runtime or development dependency requires explicit authorization | `AGENTS.md` — Dependency Policy | `explicit authorization` | `AGENTS.md`, `workflows/task-to-code.md`, `workflows/dependency-assessment.md` |
| M-07 | Every software project has a configured formatter and a configured linter, expressed only in the project's own configuration files | `AGENTS.md` — Formatting and Linting Policy | `configured linter` | `AGENTS.md`, `workflows/project-validation.md`, `references/linter-rules.md` |
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
| M-18 | A new project's target version is confirmed before its structure is created, and the official documentation of the detected stack is consulted for that version; a version is never chosen because it is installed | `AGENTS.md` — Human Decision Gates | `target version` | `AGENTS.md`, `workflows/project-initialization.md`, `references/project-creation-order.md`, `references/project-startup-decisions.md`, `skills/stack-detection/SKILL.md` |
| M-19 | Consulted documentation is current: the version guide is consulted first, a version-agnostic source is preferred, every pinned source is registered with its version and review rule, and the consulted version and date are recorded | `AGENTS.md` — Human Decision Gates | `version guide` | `AGENTS.md`, `references/official-documentation.md`, `workflows/project-initialization.md`, `skills/stack-detection/SKILL.md`, `workflows/system-audit.md` |
| M-20 | The applicable learnings are read before acting, and repeating a lesson the store already records is a defect of the task | `AGENTS.md` — Self-Learning and Maturity | `applicable learnings` | `AGENTS.md`, `workflows/self-learning.md`, `workflows/task-to-code.md`, `commands/task.md` |
| M-21 | Every task is executed to a Staff standard however simple it is; the maturity level changes the direction and review it needs, never the quality it gets | `AGENTS.md` — Self-Learning and Maturity | `Staff standard` | `AGENTS.md`, `workflows/self-learning.md` |
| M-22 | The system works on every client and operating system: the repository is the source of truth, the procedures assume a POSIX shell, degrade to NOT VERIFIED when a tool is missing, and hardcode no operating-system path or client directory | `AGENTS.md` — Client Compatibility | `POSIX shell` | `AGENTS.md`, `references/portability.md`, `workflows/system-audit.md`, `README.md` |
| M-23 | In a feature-driven architecture every unit has its own folder with its test, every feature exposes a public API, and no feature imports the internals of another | `AGENTS.md` — Engineering Principles | `public API` | `AGENTS.md`, `skills/architecture-advisor/SKILL.md`, `skills/implementation-recipes/SKILL.md`, `references/project-creation-order.md`, `workflows/output-compliance.md`, `workflows/code-audit.md` |
| M-24 | The folder structure is a decision asked with the architectural style, and the concrete trees of every style per ecosystem are defined so the user sees the folders before they exist | `AGENTS.md` — Engineering Principles | `Folder Architectures` | `AGENTS.md`, `references/folder-architectures.md`, `references/project-startup-decisions.md`, `workflows/project-initialization.md`, `skills/architecture-advisor/SKILL.md`, `references/project-creation-order.md` |
| M-25 | A monorepo requires a named concrete benefit — shared code or types, atomic changes across projects, common tooling, or consistent validation — and the topology decision records it; one team and a shared release train are not a benefit | `AGENTS.md` — Engineering Principles | `concrete benefit` | `AGENTS.md`, `skills/repository-strategy/SKILL.md`, `workflows/task-to-code.md`, `workflows/project-initialization.md` |
| M-26 | Every structural rule and folder tree names its source and the kind of source: an ecosystem official rule, a recognized community methodology, or a policy of this system; a methodology is never presented as a framework standard | `AGENTS.md` — Engineering Principles | `names its source` | `AGENTS.md`, `references/folder-architectures.md`, `references/official-documentation.md` |
| M-27 | Packages are installed as one installation package per task or phase: a single dependency is the minimal adjustment and is installed directly, and a package with several dependencies is presented once with the installation list and installed after the user's evaluation; the gate is never applied once per library, and a package is never split to fit the single-dependency case | `AGENTS.md` — Dependency Policy | `installation package` | `AGENTS.md`, `workflows/dependency-assessment.md`, `workflows/project-validation.md`, `workflows/project-initialization.md`, `references/project-creation-order.md` |
| M-28 | While a phase is in progress, every status update carries a checklist of the concrete steps of that phase, in every task including a single-phase one, detected from the work and each step naming the artifact it produces; a checklist of phase names is a defect, and the checklist is session state so following the work needs no commit | `AGENTS.md` — Progress Updates and Phase Gates | `phase checklist` | `AGENTS.md`, `workflows/session-resilience.md`, `workflows/task-to-code.md`, `references/project-creation-order.md` |
| M-29 | A phase plan is shown once before the work starts in every task, with at least one phase and no maximum, each phase naming the outcome it produces; a plan with zero phases is a defect, a single-phase task shows a one-phase plan, and the phase plan, the task plan, and the phase checklist are distinct artifacts | `AGENTS.md` — Progress Updates and Phase Gates | `phase plan` | `AGENTS.md`, `workflows/session-resilience.md`, `workflows/task-to-code.md`, `agents/orchestrator.md` |
| M-30 | In a feature-driven architecture the feature root holds only its responsibility folders and its public API; components, hooks, services, types, and local state each live in their own folder, a source file never sits at the feature root, a component and a contract never share a folder, and every ecosystem's translation of the responsibility folders is defined | `AGENTS.md` — Engineering Principles | `responsibility folders` | `AGENTS.md`, `references/folder-architectures.md`, `skills/architecture-advisor/SKILL.md`, `skills/implementation-recipes/SKILL.md`, `workflows/output-compliance.md`, `workflows/code-audit.md` |
| M-31 | Every implementation task records a design-pattern selection from the task's design forces, never from a quota or a stack: every selected pattern has a signal, an existing mechanism is preferred, the set is composed and deduplicated, the count is derived, and the selection is language- and framework-neutral with no maximum — a task may need 0, 2, 32, or 100+ patterns | `AGENTS.md` — Engineering Principles | `design forces` | `AGENTS.md`, `references/design-patterns.md`, `skills/architecture-advisor/SKILL.md`, `skills/complexity-advisor/SKILL.md`, `workflows/task-to-code.md`, `workflows/code-audit.md`, `skills/engineering-review/SKILL.md`, `commands/task.md`, `agents/orchestrator.md` |
| M-32 | A software project is offered the opt-in engineering baseline once when `docs/engineering-baseline.md` is absent: acceptance is a separate change set, "not now" asks again on a later task, and "never" is recorded and respected; a general request to implement a task is not an answer | `AGENTS.md` — Human Decision Gates | `opt-in engineering baseline` | `AGENTS.md`, `workflows/engineering-baseline.md`, `commands/adopt-baseline.md`, `skills/engineering-baseline/SKILL.md`, `workflows/task-to-code.md`, `workflows/project-initialization.md`, `commands/task.md`, `agents/orchestrator.md` |
| M-33 | A target project receives only its standard formatter and linter configuration: no custom rule file, plugin, or checker script of this system is installed in it, the structural policy adds no dependency, and the policy is verified inside the system by `workflows/output-compliance.md` | `AGENTS.md` — Formatting and Linting Policy | `no custom rule file` | `AGENTS.md`, `references/linter-rules.md`, `workflows/output-compliance.md`, `workflows/project-validation.md`, `skills/implementation-recipes/SKILL.md`, `workflows/task-to-code.md`, `references/system-workflow-regression-matrix.md`, `workflows/system-validation.md` |
| M-34 | A phase boundary is an approval gate: the system reports what the phase produced, asks in the current conversation, and starts the next phase only after an affirmative answer; the task request, an earlier continue, the plan approval, silence, a timeout, or a resumed checkpoint is not approval, and the last phase has no transition gate | `AGENTS.md` — Progress Updates and Phase Gates | `phase gate` | `AGENTS.md`, `workflows/session-resilience.md`, `workflows/task-to-code.md`, `commands/task.md`, `agents/orchestrator.md`, `workflows/system-validation.md`, `references/system-workflow-regression-matrix.md` |
| M-35 | The phase checklist is live and never a closing summary: it leads every update that reports a change and is re-emitted at every step transition — the phase start, a step completed, a step blocked, a new step, each user turn, and the phase end — it is mirrored to the client's native task or todo surface when one exists with the conversation copy as the authority, and a panel that renders only client state, such as the list of configured MCP connections, is never claimed as updated | `AGENTS.md` — Progress Updates and Phase Gates | `every step transition` | `AGENTS.md`, `workflows/session-resilience.md`, `workflows/task-to-code.md`, `agents/orchestrator.md`, `references/system-workflow-regression-matrix.md`, `workflows/system-validation.md` |

## Superseded Phrasings

A retired wording is listed here so it cannot come back. When a rule
changes, its previous wording is registered here and the check in
`workflows/system-audit.md` fails if it reappears anywhere in the canonical
content.

| Retired wording | Replaced by |
|---|---|
| Single-consumer props/local view models and small private non-exported helpers are exempt | M-02 |
| single-consumer types co-located | M-02 |
| one per file by default | M-02 |
| structural compliance rules are installed in the project | M-33 |
| contract consumer script | M-33 |
| the local `structural/no-mixed-types-and-runtime` rule | M-33 |
| a task with a single phase shows no plan | M-29 |
| Progress Updates Without Approval Fatigue | M-34 |
| An update is not a request for approval unless a decision gate actually applies | M-34 |
| When a task has more than one phase, the agent shows the **phase plan** | M-29 |
| No exceptions | M-02 |
| for all function declarations | M-01 |
| React/Next.js function style | M-01 |

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
