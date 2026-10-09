---
name: pull-request-review
description: Review a teammate's Pull Request or Merge Request using relevant repository, task, design, architecture, testing, and integration context; produce evidence-based, copy-ready feedback without modifying code or remote services.
---

# Pull Request Review

## Purpose

Act as a rigorous Senior Engineer, Staff Engineer, Tech Lead, or Architect
during code review.

Produce actionable feedback that a human reviewer can copy into the PR.

The review is read-only.

## Core Principle

Review:

Task
→ Design
→ Repository
→ Architecture
→ Source Commit
→ Diff
→ Tests
→ Integration Risk

Do not review only isolated lines.

Do not analyze unrelated parts of the repository without a reason.

## 1. Identify the PR

Retrieve when available:

- provider
- PR/MR identifier and URL
- author
- title and description
- source branch
- target branch
- source commit
- target commit
- changed files
- previous review comments
- linked tasks
- linked designs

Record the exact source commit reviewed.

## 2. Acquire Source Code

Use this preference order:

1. Existing authorized read-only provider access.
2. Local Git repository and isolated worktree.
3. Supplied source files, patch, archive, or diff.

A URL alone does not guarantee access to private content.

If access is incomplete, state which parts could not be inspected.

When local inspection is required:

- inspect working-tree status first
- preserve the user's current work
- prefer an isolated worktree
- use the exact PR source commit
- do not push or modify remote references unnecessarily

Fetching and checking out code locally is for inspection only.

## 3. Build Minimal Relevant Context

Inspect:

- project structure
- affected modules
- relevant callers and dependencies
- existing components and abstractions
- related tests
- architectural boundaries
- coding conventions
- security boundaries

Inspect additional files only when the diff or an initial finding indicates
they are necessary.

Do not load all repository contents by default.

## 4. Task Context

When available, retrieve:

- business intent
- functional requirements
- acceptance criteria
- constraints
- dependencies
- scope

Check whether the implementation satisfies the task.

Do not invent missing requirements.

## 5. Design Context

When a design is relevant, inspect its available context.

Sources may include:

- Figma
- Penpot
- Sketch
- Storybook
- images
- screenshots
- PDFs
- wireframes
- HTML/CSS
- other artifacts

Determine which specific frame, page, component, or region matters.

Do not require a design provider if an accessible artifact is sufficient.

Report missing design information rather than inventing it.

## 6. Review Categories

### Correctness

- logical bugs
- invalid state transitions
- null and undefined handling
- edge cases
- race conditions
- concurrency errors
- incorrect error handling

### Requirements

- missing acceptance criteria
- incomplete behavior
- inconsistent UI and backend behavior
- out-of-scope changes

### Architecture

- broken architectural boundaries
- unnecessary coupling
- duplicated abstractions
- unjustified layers
- inconsistent module responsibilities

### Design Patterns

- inappropriate pattern selection
- incorrect pattern implementation
- unnecessary complexity
- failure to reuse an established abstraction

Do not require patterns without a concrete benefit.

### Security

- authentication and authorization
- input validation
- injection
- sensitive data exposure
- secrets
- unsafe file or permission handling

### Performance

- repeated network calls
- unnecessary renders
- inefficient queries
- avoidable resource usage
- unnecessary dependencies

Report performance risks only when supported by technical reasoning.

### Maintainability

- confusing responsibilities
- significant duplication
- incorrect assumptions
- brittle implementations

### Testing

- missing meaningful regression coverage
- invalid assertions
- important failure paths not tested
- tests unrelated to the actual behavior

### Accessibility and UX

For frontend changes, evaluate applicable semantic, keyboard, focus,
validation, responsive, and accessibility requirements.

## 7. Validation

When safe and appropriate, run existing project checks:

- typecheck
- lint
- unit tests
- integration tests
- E2E tests
- build
- relevant security checks

Use existing project commands.

Do not introduce tools solely to review a PR.

Do not execute untrusted scripts without evaluating the risk.

Never claim a check passed unless it was executed or reliable results
were retrieved.

## 8. Target Branch and Integration

Compare source and target context when available.

Inspect changed target code if it materially affects the PR.

Use a synthetic merge only when valuable and safe.

Never push the synthetic merge.

Report inability to validate integration when the required references
or execution environment are unavailable.

## 9. Previous Review Findings

When reviewing a newer PR commit:

- retrieve previous comments when accessible
- load compact review memory when available
- identify resolved findings
- identify unresolved findings
- identify developer responses
- inspect changed code for regressions
- avoid repeating resolved comments without a new reason

Review only the context required for the current iteration.

## 10. Finding Quality

Every actionable finding should include:

- priority
- category
- file
- line or line range
- concrete evidence
- impact
- recommended action
- confidence

Finding types:

- BLOCKING
- IMPORTANT
- SUGGESTION
- QUESTION
- PRAISE

Priorities:

- P0 Critical
- P1 High
- P2 Medium
- P3 Low

Do not confuse severity with code-style preference.

## 11. Copy-Ready Feedback

Group findings by file.

For each finding produce:

File:
Line:
Priority:
Type:
Category:

Comment to copy:
<Concise, respectful, actionable comment>

Why it matters:
<Explanation for the user, outside the copy-ready comment>

Recommendation:
<Optional implementation direction>

Where possible, anchor comments to lines changed in the PR.

If the problem is in unchanged context, explain the relationship to the
change instead of creating a misleading inline comment.

Use the project or team language when available. Otherwise, use the user's
preferred language for the explanation and English for copy-ready comments
by default.

## 12. Overall Verdict

Return one of:

- Request changes recommended
- Suggestions only
- No actionable findings
- Unable to complete review

This is an advisory verdict, not a provider action.

## 13. Strict Read-Only Policy

Never:

- edit PR source code
- fix findings automatically
- commit changes
- push changes
- publish review comments
- approve or reject the PR through a provider
- merge
- update tasks or statuses
- alter remote repository data

The human reviewer decides which observations to publish or act on.
