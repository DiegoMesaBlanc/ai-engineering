# Project Initialization Workflow

## Purpose

Create a new project from scratch: recover the source analysis, settle the
architecture **with the user**, establish the repository, and install the
mandatory quality tooling before the first feature is written.

## When to use

- The user invoked `commands/create-project.md`, the explicit signal to create
  a project from scratch. Without that command, do not create a project from
  zero: ask the user first.
- The target does not exist yet: a new service, application, library, or
  package.
- The task is to build a backend, frontend, mobile app, or library from
  scratch.
- The task points to an existing analysis, such as an estimation document,
  and asks to reuse it instead of re-deriving the requirements.

## Boundaries

- Do not invent requirements that the source analysis already settles.
- Do not implement a feature in this workflow. It ends with an initialized
  project that the Task-to-Code sequence continues from.
- Do not choose the architectural style without the user.

## Phase 1 — Recover the Source Analysis

1. Resolve the artifact the task points to: a document path, a link, or a
   task reference. Treat it as read-only evidence.
2. Read it completely before designing anything.
3. Extract, without re-deriving:
   - the product goal;
   - the scope, including what is explicitly out of scope;
   - the stack assumptions;
   - the open questions.
4. A requirement stated by the analysis is an **explicit** requirement. A
   requirement the analysis does not cover is **unknown**, never implied.
   Never fill a gap with an assumption that contradicts the analysis.
5. Ask the user only about the unknowns that block the architecture decision
   or the stack. Do not re-ask what the analysis already answers.
6. Report what was reused and what remains open, before proposing the
   architecture.

If the task points to no analysis, state that the requirements are derived
from the task itself and confirm the scope with the user.

## Phase 2 — Detect the Stack and the Target Version

- Detect the language, runtime, framework, and versions the analysis
  assumes, and confirm them against the project brief.
- **Confirm the target version** of the language, framework, and runtime.
  Its source of truth is, in order: the source analysis, the ecosystem's
  official documentation for the detected stack, then a stated default. When
  the analysis states a version that differs from the current stable
  release, report the difference and confirm which one to use.
- **Consult the official documentation** of the detected stack **for that
  version** before any stack-specific decision. Consult the ecosystem's
  version guide first to learn what is current, and prefer a
  version-agnostic source over one pinned to an older release. Record the
  source, the version it documents, the date of the consultation, and the
  decisions it informed. A source that does not cover the target version is
  reported, never applied silently.
- **Compare the target version with the environment.** A version is never
  chosen because it happens to be installed. When the environment does not
  have the target version, report it: the project is created for the target
  version, and any check that could not run against it is reported as
  `NOT VERIFIED` with the reason.
- A stack assumption in the analysis that conflicts with the detected
  environment is a material question for the user, not a silent correction.

## Phase 3 — Startup Decision Set

Settle every decision that creating this project entails, in one pass, before
creating the structure. The catalog is
`references/project-startup-decisions.md`: it lists the decisions per project
type, with the recommended default, when the user must decide, and where the
answer comes from.

### 3.1 Resolve each decision

For every decision in the catalog, resolve it in this order:

1. **Already answered by the source analysis**: reuse it. Do not re-ask.
2. **Governed by a detected convention or an existing policy**: apply it and
   state which one. Do not ask.
3. **Has a defensible default**: take the default, state it explicitly, and
   proceed.
4. **Materially changes the outcome and has no default**: ask the user.

For a backend, the catalog covers the API surface, the data and persistence
layer, identity and access, asynchronous work and integrations, and
operations. For a frontend, the library, a CLI, or a mobile application, it
covers the decisions of that type. Apply the universal decisions to all of
them.

### 3.2 Ask once, grouped

Apply the asking rules of `references/project-startup-decisions.md`
("Asking rules"). They are the canonical rules and are not restated here.

### 3.3 Architectural style

Present the applicable styles with concrete trade-offs, adapted to the
detected stack and the scope recovered in Phase 1:

| Style | When to use | Structure hint |
|---|---|---|
| Layered | Simple domains, single team, low coupling | `src/{api,services,domain,infrastructure}/` |
| Feature-first | Multiple domains, independent features | `src/features/<feature>/{api,domain,ui,contracts}/` |
| Hexagonal / Ports & Adapters | Multiple external integrations, testability critical | `src/{domain,application,adapters}/` with ports in domain |
| DDD (Domain-Driven Design) | Complex domain logic, ubiquitous language | `src/{domain,application,infrastructure,interfaces}/` |
| Modular Monolith | Multiple bounded contexts, single deployable | `src/modules/<context>/{domain,application,api}/` + `src/shared/` |
| Event-driven | Asynchronous integration between bounded contexts | `src/{producer,consumer,contracts,messaging}/` |

**Present the concrete trees, not one-line hints.** The trees of every
style, translated to the detected ecosystem, are defined in
`references/folder-architectures.md`. Show the applicable ones with their
folders. A style presented only as a hint is not a choice the user can make.

Microservices are a **deployment** decision, not a code-organization
default. Do not offer them without evidence that independent deployment is
a stated requirement.

Then ask explicitly, showing the trees:

> What architecture and folder structure should the new project use? I
> recommend \<style\> because \<reason from the scope and the analysis\>.
> Its folders would be \<the tree from the reference\>. The alternatives
> are \<the other applicable styles\>, with the folders \<their trees\>, and
> they differ in \<trade-offs\>.

The question is about the architecture **and the folders**. A user who
answers only the style still gets a proposed tree, and the tree is
confirmed before anything is created.

**Ports and Adapters is always among the options when it applies.** It fits
a project with several external integrations, and it is a legitimate choice
even when a simpler style would also work. It is never filtered out because
it looks advanced.

### 3.4 Topology decision

Decide the repository topology, and record the reason. A monorepo requires a
**concrete benefit** — shared code or types, atomic changes across projects,
common tooling, or consistent validation. One team and a shared release
train are not that benefit: when the applications share no code and no
change spans them, separate repositories are the simpler topology, and a
coordinated release is a pipeline concern. See
`skills/repository-strategy/SKILL.md`.

### 3.5 Scope agreement

Agree the **catalog of what the project will contain** before creating
anything: the modules, the public surfaces, and the dependencies with their
reasons. The catalog is a ceiling the creation never exceeds without
asking. See "Scope Control" in `references/project-creation-order.md`.

### 3.6 Record

- Record the chosen style, the reason, and the rejected alternatives.
- Record every decision resolved by categories 1 to 3, with its source.
- Record each decision the user answered, with the answer.
- Do not create the repository structure, and do not write code, before the
  decision set is settled.

## Phase 4 — Create the Project, Step by Step, Stopping at Every Checkpoint

Create the project in the order defined by
`references/project-creation-order.md`. The order matters: each step depends
on the previous one, and each step leaves a defined extension point, which
is what makes the project grow well instead of decaying.

**The creation is progressive and the user controls where it stops.** It is
not a single pass that produces a finished project. The **stopping points**
are the checkpoints of `references/project-creation-order.md`.

### 4.1 Agree the scope first

Before creating anything, agree the **catalog of what the project will
contain**: the modules, the public surfaces, and the dependencies with
their reasons. The catalog is a ceiling. See "Scope Control" in
`references/project-creation-order.md`.

Never exceed the catalog without asking. A project that ends with five
endpoints nobody discussed, or twenty dependencies nobody requested, is a
defect.

### 4.2 Execute one step, then stop

For each step in order:

1. Execute **only that step**.
2. Verify it with the verification the step defines.
3. Update the progress tracker in `references/project-creation-order.md`.
4. Report: what the step produced, how it was verified, the tracker state,
   and the next suggested step with exactly what it would add.
5. **Wait.** The user may continue, stop, or reorder.

Do not start the next step on your own initiative. A step that is "mostly
done" produces a structure that has to be reworked, and a step the user
never wanted is work that has to be undone.

### 4.3 Stopping is a normal outcome

When the user stops, the project is left exactly at that checkpoint, with
the tracker recording it. Report the resume point: the next `pending` step,
and what the next session needs to confirm before continuing.

A session that ends at a checkpoint is **complete, not broken**. Do not
generate the remaining steps to "finish" it, and do not treat the pause as a
defect to fix.

### 4.4 Apply the growth rule

The creation order exists so the project can grow. When a new piece of code
does not fit the growth rule of its layer, the structure is wrong, not the
code. Do not create a parallel structure to avoid changing the agreed one.

### 4.5 Never exceed the agreed scope

- One vertical slice at a time. The first is complete before the second
  starts.
- A new project starts with the ecosystem minimum. Every additional
  dependency is named, justified, and approved.
- Do not pre-create contracts, endpoints, or modules for work that is out of
  scope.

## Phase 5 — Report

Report:

- the source analysis that was reused, and the requirements taken from it;
- the stack, confirmed or corrected;
- the startup decision set: the decisions the user answered, the decisions
  resolved by default, and the source of each one;
- the creation progress tracker from `references/project-creation-order.md`,
  with each step marked `done`, `pending`, or `stopped here`;
- the scope catalog agreed in Phase 3, and the confirmation that the
  creation did not exceed it;
- the resume point: the next `pending` step and what it would add;
- the architectural style chosen by the user, and the alternatives rejected;
- the structure created, step by step, and the extension point each layer
  leaves for the next feature;
- the tooling installed and the commands to run it;
- the Folder Architecture Map;
- what remains open, and the next phase.

Then continue with `workflows/task-to-code.md` from Phase 1. The project is
initialized; it is not finished. A new project follows the canonical
Task-to-Code sequence after this workflow.
