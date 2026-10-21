---
description: Create a new project from scratch, progressively and with user-controlled stopping points, with the scope catalog agreed first and every step verified and checkpointed before the next one starts.
agent: orchestrator
---

Run the Project Initialization Workflow:

`workflows/project-initialization.md`

## Command

`/create-project` is the explicit signal to create a project from scratch.
Without it, do not create a project from zero: ask the user first.

## Input

Optional arguments in `$ARGUMENTS`:

- The product goal, the source analysis to reuse, or the scope of the
  project to create.
- A path to an existing analysis, such as an estimation document, to reuse
  instead of re-deriving the requirements.

## Mandatory Behavior

1. **This command is the explicit signal to create a project from scratch.**
   Without it, do not create a project from zero. When a task needs a
   project that does not exist, ask the user whether to run this command
   first.
2. Recover the source analysis and reuse it. Do not invent requirements the
   analysis already settles.
3. Confirm the stack against the official documentation.
4. Settle the startup decision set: reuse what the analysis answers, apply
   what a detected convention governs, state what takes a defensible
   default, and ask the user **once**, grouped by theme, only for the
   decisions that materially change the outcome. The architectural style is
   always asked, and asked first.
5. Agree the **scope catalog**: the modules, the public surfaces, and the
   dependencies with their reasons. It is a ceiling, not a floor.
6. Create the project **one step at a time**, following
   `references/project-creation-order.md`. After each step: verify it,
   update the progress tracker, report what the step produced and what the
   next step would add, and **wait** for the user to continue or stop.
7. Never exceed the agreed scope without asking. One vertical slice at a
   time, and the ecosystem minimum for dependencies.
8. When the user stops, leave the project at that checkpoint, record it in
   the tracker, and report the resume point. Stopping is a normal outcome.

## Safety

- Do not create a commit, push, or change remote state. The initial commit
  requires the explicit approval of the commit approval policy.
- Do not install a dependency without the authorization the dependency
  policy requires, except the mandatory formatter and linter setup.
- Do not print credentials or secret values.

## Output

Return the report defined by the workflow: the source analysis reused, the
stack, the startup decision set, the scope catalog, the progress tracker
with the current checkpoint and the next suggested step, the tooling
installed, the Folder Architecture Map, and the resume point.
