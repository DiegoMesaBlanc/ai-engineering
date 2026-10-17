---
description: Start a task, understand its requirements and design, ask for the branch name, and implement only after the task branch is ready.
agent: orchestrator
---

Start or continue the engineering task specified by:

$ARGUMENTS

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

Detect engineering intent from the request (architecture/solution design, time/cost estimation, requirements analysis/technical investigation, software implementation, validation/debugging/code review). For architecture-only or estimation-only, do not modify application source code unless separately authorized. For implementation, align requirements before code changes and then proceed autonomously for routine work, providing concise status updates without approval fatigue; pause only when a material decision gate applies.

Key required behaviors within that workflow:

- complete the dependency assessment in
  `workflows/dependency-assessment.md` and record its outcome before
  finalizing the plan
- present meaningful decisions when approval is required, including any
  proposed new dependency
- evaluate architecture, reuse, and code organization before implementing,
  and validate the result against the proposed design and conventions
- apply proportional planning rigor: derive it from the Phase 8 complexity
  classification via `workflows/task-planning.md`, keep simple tasks
  lightweight, and define the expected behavior before implementing complex
  or high-risk changes
- base the architecture diagnosis on implementation evidence (detected,
  inferred, or absent), and when no coherent architecture is detected,
  conventions conflict, or a new architecture must be chosen, present the
  options and wait for the user's decision before implementing
- implement following current conventions, then validate and review
- present the Git diff and commit proposal

Prefer no new dependency when the existing stack, native APIs, or project
abstractions are sufficient. Adding or installing a new runtime or
development dependency requires explicit authorization, except for the
mandatory formatter and linter setup defined in AGENTS.md. Never treat
approval to implement the task as authorization to introduce libraries.

## PR policy

Generate copy-ready PR title and description when requested.

Do not create or publish a PR/MR.

Do not push, merge, publish comments, or update a remote task.
