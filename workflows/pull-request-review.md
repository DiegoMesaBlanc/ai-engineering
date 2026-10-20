# Pull Request Review Workflow

## Goal

Produce a rigorous, evidence-based, read-only review of a colleague's
Pull Request / Merge Request.

The deliverable is copy-ready feedback the user can paste.

No remote writes are allowed.

The workflow reviews the change; it does not implement fixes.

This workflow is **not** the post-implementation review of your own change.
That one is `skills/engineering-review/SKILL.md`, used by
`workflows/task-to-code.md` Phase 10 to validate a change the agent just
implemented. A review of someone else's pull request uses this workflow.

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

Record the exact source revision and base reviewed.

## Phase 2 — Access Strategy

Choose the lowest-cost option that supports a useful review.

Preference:

1. Existing read-only access.
2. Local Git repository.
3. Supplied diff/source artifacts.

When private content is inaccessible, request only the missing information.

Use local evidence first. Use external providers or MCP only when needed
and authorized under the provider and access policies.

## Phase 3 — Preserve Workspace

Inspect current Git status.

Never discard user changes.

Use an isolated worktree when checking out source code.

Do not switch or overwrite the user's primary branch.

Never modify the working tree as part of a review.

A review may span multiple sessions. Before an expensive review step, record
a checkpoint as described in `workflows/session-resilience.md` so the review
can continue after `/compact` or `/new` without reconstructing state.

## Phase 4 — Exact Version

Identify and record the exact reviewed revision.

Compare with the base/target commit when available.

Do not silently review a different branch, a stale revision, or an
unrelated local diff.

Line numbers, references, and behavior must correspond to the actual
reviewed revision. If the diff lacks context, inspect the relevant file and
surrounding implementation.

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

If no task or design is available, continue using the code and PR
description.

Do not invent requirements or design specifications.

## Phase 7 — Review Dimensions

Select review checks according to the affected code and project stack.

Consider, when relevant:

- Functional correctness and acceptance criteria.
- Regressions and backward compatibility.
- Type safety and invalid states.
- Input validation and security boundaries.
- Authorization and authentication.
- Error handling and failure recovery.
- Asynchronous behavior, cancellation, and race conditions.
- State management and side effects.
- Accessibility and keyboard interaction.
- UI consistency against available design evidence.
- Performance and resource usage.
- Data integrity and database changes.
- Test adequacy and missing regression cases.
- Formatter, linter, and existing project conventions.

Do not force every dimension onto every PR. Concentrate on likely defects
in the actual change.

For UI and design reviews, distinguish programmatically verified behavior
from visual observations. Never claim exact visual matching without
adequate evidence.

Use these Skills when relevant:

- Project Analysis
- Stack Detection
- Complexity Advisor
- Architecture Advisor
- Testing Strategy
- Engineering Review

Load only the Skills the change requires.

## Phase 8 — Validate

Run relevant existing checks if safe and useful.

Record actual results.

Do not perform unnecessary full builds or test suites when a smaller check
provides adequate evidence for the specific concern.

Never claim that a test or command was executed unless its actual result is
available.

When execution is not permitted or feasible, state that the conclusion is
based on static inspection.

Never install dependencies or run potentially destructive commands as part
of a review.

## Phase 9 — Evidence Requirements

Every finding must include:

- Severity: Critical, High, Medium, or Low.
- Exact file and line, or the narrowest available relevant range.
- The specific behavior or defect.
- Why it is incorrect or risky.
- Reproducible evidence, a direct code reference, or a clearly explained
  execution path.
- A practical suggested correction.
- Confidence: High, Medium, or Low.

Do not invent line numbers, test results, runtime behavior, requirements,
or design specifications.

Line numbers must be verified against the actual reviewed revision. If the
diff does not provide sufficient context, inspect the relevant file and
surrounding implementation.

Distinguish confirmed defects from reasonable concerns that require human
verification.

Do not report stylistic preferences as defects unless an explicit project
rule or requirement supports them.

## Phase 10 — Validate Findings Against Context

Apply the canonical procedure in `skills/engineering-review/SKILL.md`
("Validate Findings Against Context"). It is not restated here.

## Phase 11 — Severity and Confidence

Use these general definitions:

- Critical: a severe, immediate-impact issue such as a demonstrable
  security compromise, major data corruption, or similarly catastrophic
  failure.
- High: a substantial defect that blocks an important user flow, exposes
  sensitive data, or causes significant incorrect behavior.
- Medium: a meaningful defect with narrower scope, impact, or conditions.
- Low: a limited-impact defect or concrete minor issue worth fixing.

Adjust severity to actual evidence and project context.

Do not inflate severity to make the report appear comprehensive. Do not
assign Critical or High solely because code handles payments,
authentication, or other sensitive domains.

Set confidence to High only when the evidence is verified and direct;
otherwise use Medium or Low and state the missing verification.

Separate confirmed findings from observations, questions, and
design-review items.

## Phase 12 — Review Memory

When a previous review iteration exists:

- compare source commit
- retrieve compact previous findings
- track unresolved/resolved findings
- identify regressions
- avoid duplicate feedback

## Phase 13 — Report

Produce a concise, copy-ready review.

### Review context

- PR or change identifier.
- Reviewed branch/revision and base, when known.
- Task or acceptance criteria used.
- Scope and evidence sources.
- Checks actually executed.

### Findings

Order findings by severity, then confidence.

For each finding:

- `[Severity] Short title`
- `File:line`
- **Evidence:** verified code path, requirement, or execution result.
- **Impact:** concrete failure mode.
- **Recommendation:** actionable correction.
- **Confidence:** High / Medium / Low.

If no actionable defects are found, explicitly state that no findings were
identified within the reviewed scope. Do not claim the change is bug-free.

### Verification gaps

Record relevant checks that could not be performed and explain why.

### Non-blocking observations

Include only useful observations that do not meet the bar for a confirmed
defect.

### Summary

Choose one evidence-supported recommendation:

- Request changes.
- Review comments / follow-up recommended.
- No actionable findings identified.

Do not say "Approved" or imply that a remote PR state was changed.

## Phase 14 — Human Action

The user decides whether to:

- copy a comment
- ignore a suggestion
- request clarification
- ask the developer for a correction

The workflow does not publish findings or modify code.

## Phase 15 — Cleanup

Remove a temporary worktree only when safe.

Never delete user work or project branches.

---

## Strict Read-Only Policy

Never:

- edit source files or fix findings automatically
- create commits
- push branches
- publish review comments
- approve or reject a PR through a provider
- merge branches
- update remote tasks or statuses
- alter remote repository data

The human user performs remote actions manually.
