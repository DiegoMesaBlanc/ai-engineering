# Pull Request Review Workflow

## Goal

Produce a rigorous, context-aware, read-only review of a colleague's
Pull Request / Merge Request.

The deliverable is feedback the user can copy and paste.

No remote writes are allowed.

---

## Phase 1 — Identify

Accept a PR/MR URL, provider and identifier, local repository, or supplied
diff.

Determine available:

- PR metadata
- source and target commits
- linked task
- linked design
- prior comments

Do not ask the user for data that can be reliably retrieved.

## Phase 2 — Access Strategy

Choose the lowest-cost option that supports a useful review.

Preference:

1. Existing read-only access.
2. Local Git repository.
3. Supplied diff/source artifacts.

When private content is inaccessible, request only the missing information.

## Phase 3 — Preserve Workspace

Inspect current Git status.

Never discard user changes.

Use an isolated worktree when checking out source code.

Do not switch or overwrite the user's primary branch.

## Phase 4 — Exact Version

Identify and record the source commit.

Compare with the target commit when available.

Do not silently review a different or stale revision.

## Phase 5 — Minimal Context

Inspect changed files first.

Follow references into related components, services, tests, configuration,
or architecture only when needed.

Do not load the complete repository into model context by default.

## Phase 6 — Task and Design

When available:

- load Task Context
- load Design Analysis

Retrieve only relevant acceptance criteria and design regions.

If no task or design is available, continue using the code and PR description.

## Phase 7 — Review

Use:

- Project Analysis
- Stack Detection
- Complexity Advisor
- Architecture Advisor
- Testing Strategy
- Engineering Review

Load only Skills required for the changes.

## Phase 8 — Validate

Run relevant existing checks if safe and useful.

Record actual results.

Do not perform unnecessary full builds or test suites when a smaller check
provides adequate evidence for the specific concern.

## Phase 9 — Review Memory

When a previous review iteration exists:

- compare source commit
- retrieve compact previous findings
- track unresolved/resolved findings
- identify regressions
- avoid duplicate feedback

## Phase 10 — Report

Group findings by file.

Each finding must include:

- file and line
- priority
- category
- evidence
- comment to copy
- rationale
- recommendation

Provide a concise overall assessment.

If no actionable findings exist, say so explicitly.

## Phase 11 — Human Action

The user decides whether to:

- copy a comment
- ignore a suggestion
- request clarification
- ask the developer for a correction

The workflow does not publish findings or modify code.

## Phase 12 — Cleanup

Remove a temporary worktree only when safe.

Never delete user work or project branches.