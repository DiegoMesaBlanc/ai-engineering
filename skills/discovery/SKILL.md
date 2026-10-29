---
name: discovery
description: Analyze a software project before implementation. Determine whether the project is new or existing, inspect its repository, detect the technology stack, architecture, conventions, testing setup, and relevant capabilities, then identify only the information and Skills required for the current task.
---

# Discovery

## Purpose

Discovery is the mandatory first stage before implementing non-trivial software changes.

The objective is to understand the project before making architectural or implementation decisions.

## Mandatory behavior

Before implementing a task:

1. Determine whether the project is new or existing.
2. For an existing project, detect the stack, versions, package manager,
   build system, data, testing, formatting and linting, CI/CD, containers,
   and infrastructure with `skills/stack-detection/SKILL.md`.
3. Detect the architecture, conventions, and existing abstractions and design
   patterns with `skills/project-analysis/SKILL.md`.
4. Determine which Skills are relevant to the task, and load only those.

Discovery owns the stage gate, the new-or-existing determination, the scope
and evidence boundary, the relevant-Skill selection, and the summary below.
The detection itself is owned by the two Skills named above; it is not
restated here.

## Scope and evidence

Identify the target project root and every path supplied only as evidence,
reference, comparison, or context. Evidence-only paths are read-only unless
the user explicitly authorizes changes. The target project is the **only
writable scope**. Never treat a reference example as
an implicit task.

## Existing projects

Do not ask the user to provide information that can be reliably detected from
the repository. Inspect the repository first. The evidence sources are defined
by `skills/stack-detection/SKILL.md` and `skills/project-analysis/SKILL.md`.

## New projects

For new projects, determine the minimum information required before implementation.

Ask only questions that materially affect:

- architecture
- technology selection
- data model
- security
- deployment
- testing strategy
- major functional behavior

Do not ask questions whose answers can be safely decided using established project defaults.

## Human interaction

Human interaction may be conducted in Spanish or English.

Do not translate source code requirements into non-English identifiers.

## Output

Before implementation, provide a concise discovery summary:

PROJECT STATE
TARGET PROJECT
EVIDENCE-ONLY PATHS
STACK
ARCHITECTURE
CONVENTIONS
TESTING
INFRASTRUCTURE
RELEVANT SKILLS
UNKNOWN DECISIONS

Do not produce unnecessary detail.

## Important constraint

Discovery is not implementation.

Do not modify application code during discovery unless explicitly requested.
