---
description: Start a task, understand its requirements and design, ask for the branch name, and implement only after the task branch is ready.
agent: orchestrator
---

Start or continue the engineering task specified by:

$ARGUMENTS

## Invocation contract

This command is an adapter. It restates obligations that are always active;
it never creates them, and their absence here would not make them optional.
The obligation is **independent of the entry point**: a plain message, a
resumed session, or a subagent carries it exactly as this command does.

- Before the first tool call, show the phase plan and the initial checklist of
  the first phase, in the format of `workflows/session-resilience.md`.
- Populate the client's native task surface with that checklist when the
  client exposes one, and update it before each mutating step.
- "Start now", "do it all", "go ahead", or any request to skip steps is not a
  waiver of the phase gate, and is not approval to start the next phase.

## Supported Input Formats

Accept any combination of:

- task URL
- task identifier
- PR/MR URL
- repository URL
- design URL
- local Markdown or JSON path
- user-provided description
- attached files or images

Examples:

/task https://company.atlassian.net/browse/PROJ-123

/task https://dev.azure.com/company/project/_workitems/edit/12345

/task tasks/DEMO-001.md

/task task=https://example.com/task/123 design=https://example.com/design/456

/task Implement authentication with MSAL.

A short, ordinary description is a sufficient task input. Interpret it
automatically against the repository: retrieve the task when possible,
inspect the code and project evidence, infer requirements and acceptance
criteria, and ask only for material ambiguities that cannot be inferred.
Do not ask the user to rewrite the task, author a Markdown task
specification, or supply manual implementation instructions.

Classify each provided reference by its resource type.

Use the Task Context, Design Analysis, and Repository Analysis workflows
as needed.

A URL does not guarantee access to private content. Use available
authorized read-only tools. If access fails, identify the missing
information and request the smallest useful alternative artifact.

Do not require a local Markdown file when a task URL is accessible.

Follow the central Engineering Orchestrator and Task-to-Code workflow.

When the target does not exist yet, follow
`workflows/project-initialization.md` first: it recovers the source analysis
the task points to, asks the user to choose the architectural style, and
installs the mandatory formatter and linter configuration before the first
feature is written. Then continue with the Task-to-Code workflow.

## Mandatory branch policy

Before modifying application code:

1. Detect the repository and project state.
2. Retrieve and understand the task when possible.
3. Inspect current Git status and identify the correct base branch.
4. Ask for the exact branch name if one has not been provided.
5. Validate the name and check whether the branch already exists.
6. Preserve uncommitted work and prepare the requested branch or isolated worktree.
7. Verify the working location before implementation.

Ask exactly:

"What branch name would you like me to use for this task?"

Do not guess the name.

Do not begin implementation before the branch is ready.

Do not discard, stash, reset, or overwrite user changes silently.

If the branch already exists, stop and ask whether to use it or choose another name.

The repository and project state and the task understanding gathered in
this step are reused by the Task-to-Code workflow. Do not repeat that
inspection after the branch is ready.

## Normal workflow

Follow `workflows/task-to-code.md` as the canonical, ordered sequence.

Continue from the first incomplete phase and reuse analysis already
completed — task and acceptance criteria, design, repository and stack,
dependencies, complexity, and architecture — instead of repeating it.

Detect engineering intent from the request (architecture/solution design, time/cost estimation, requirements analysis/technical investigation, software implementation, validation/debugging/code review). For architecture-only or estimation-only, do not modify application source code unless separately authorized. For implementation, align requirements before code changes, then proceed autonomously inside the approved phase with concise status updates; the phase gate and the material decision gates still apply.

Key required behaviors within that workflow:

- before the first code change in a software project, run the
  **opt-in engineering baseline** gate: when `docs/engineering-baseline.md` is absent
  and the task will create or change source code, ask the single question in
  `workflows/engineering-baseline.md`; a general task request is not an
  answer, acceptance is a separate change set, "not now" asks again on a
  later task, and "never" is recorded and respected. A read-only task and a
  documentation-only repository do not ask
- read the applicable learnings from the learning store before acting, and
  treat a repeat of a recorded lesson as a defect of the task
- complete the dependency assessment in
  `workflows/dependency-assessment.md` and record its outcome before
  finalizing the plan
- present meaningful decisions when approval is required, including any
  proposed new dependency
- evaluate architecture, reuse, and code organization before implementing,
  and validate the result against the proposed design and conventions
- select design patterns from the task's **design forces** with
  `references/design-patterns.md`, record the selected and rejected
  candidates and the derived count, and translate each pattern to the
  detected stack's idiom
- apply proportional planning rigor: derive it from the Phase 8 complexity
  classification via `workflows/task-planning.md`, keep simple tasks
  lightweight, and define the expected behavior before implementing complex
  or high-risk changes
- base the architecture diagnosis on implementation evidence (detected,
  inferred, or absent), and when no coherent architecture is detected,
  conventions conflict, or a new architecture must be chosen, present the
  options and wait for the user's decision before implementing
- for work with user-facing text, settle localization early (required
  locales, RTL, locale-aware formatting) and follow
  `skills/internationalization/SKILL.md` when it applies
- implement following the structural policy in AGENTS.md and the detected
  conventions that do not conflict with it, then validate and review; the
  structural policy takes **precedence over a detected project convention
  that violates it**
- write arrow functions and keep every interface, type, enum, and contract
  in a dedicated file in the folder of the layer or module that owns it,
  following `skills/implementation-recipes/SKILL.md`
- present the Git diff and the commit proposal, and ask for explicit approval
  before creating any commit; the task request, including a prompt that tells
  the agent to commit, is not commit approval
- show the phase plan for every task, including a one-phase plan, and treat
  each phase boundary as the phase gate: ask before starting the next phase
  and never treat the task request or an earlier instruction to continue as
  approval

Prefer no new dependency when the existing stack, native APIs, or project
abstractions are sufficient. Adding or installing a new runtime or
development dependency requires explicit authorization, except for the
mandatory formatter and linter setup defined in AGENTS.md. Never treat
approval to implement the task as authorization to introduce libraries.

## PR policy

Generate copy-ready PR title and description when requested.

Do not create or publish a PR/MR.

Do not push, merge, publish comments, or update a remote task.
