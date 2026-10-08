# Pull Request Review Workflow

## Purpose

Perform a complete engineering review of a Pull Request or Merge Request.

The workflow evaluates:

Task
→ Design
→ Repository
→ Architecture
→ Source Branch
→ Diff
→ Tests
→ Integration

---

# Phase 1 — Identify PR

Retrieve:

- provider
- PR/MR ID
- title
- description
- author
- source branch
- target branch
- source commit
- target commit
- reviewers
- previous comments
- previous reviews
- linked tasks
- linked designs

---

# Phase 2 — Preserve Workspace

Inspect:

- current branch
- working tree
- staged changes
- unstaged changes

Never replace the user's active work.

Create an isolated worktree whenever possible.

---

# Phase 3 — Retrieve PR Code

Retrieve the exact source commit referenced by the PR.

Do not rely on an outdated local checkout.

Record:

Reviewed commit:
...

---

# Phase 4 — Target Branch

Retrieve the relevant target branch.

Determine:

- merge base
- target commit
- PR changes
- potential target changes

---

# Phase 5 — Repository Analysis

Analyze the PR branch:

- project structure
- stack
- framework
- architecture
- conventions
- abstractions
- state management
- APIs
- database
- testing
- CI/CD
- design system

---

# Phase 6 — Task Context

When a task exists:

Load:

`skills/task-context/SKILL.md`

Analyze:

- business intent
- functional requirements
- acceptance criteria
- constraints
- dependencies
- scope

---

# Phase 7 — Design Context

When a design exists:

Load:

`skills/design-analysis/SKILL.md`

Analyze:

- relevant screens
- components
- states
- interactions
- responsive behavior
- accessibility
- design-system usage

The design source may be:

- Figma
- Penpot
- Sketch
- Storybook
- PDF
- image
- screenshot
- HTML/CSS
- other artifact

---

# Phase 8 — Complexity

Load:

`skills/complexity-advisor/SKILL.md`

Determine:

- task complexity
- architectural risk
- implementation risk

---

# Phase 9 — Architecture

Load:

`skills/architecture-advisor/SKILL.md`

when justified.

Evaluate:

- architecture compatibility
- coupling
- abstraction
- patterns
- boundaries
- maintainability

---

# Phase 10 — Diff Review

Analyze:

- added files
- deleted files
- modified files
- renamed files
- dependencies
- configuration
- migrations
- tests

Do not evaluate changes only from line-level syntax.

---

# Phase 11 — Engineering Review

Evaluate:

## Correctness

Logic, edge cases, state transitions, concurrency, errors.

## Maintainability

Naming, cohesion, duplication, abstractions.

## Architecture

Boundaries, coupling, responsibilities, conventions.

## Design Patterns

Appropriateness and complexity.

## Security

Authentication, authorization, validation, injection, secrets.

## Performance

CPU, memory, rendering, network, database, caching.

## Testing

Unit, integration, E2E, regressions, failure paths.

## Accessibility

Relevant frontend accessibility requirements.

## Requirements

Task and acceptance criteria compliance.

---

# Phase 12 — Validation

When safe and appropriate:

- typecheck
- lint
- unit tests
- integration tests
- E2E
- build
- security validation

Never claim success without evidence.

---

# Phase 13 — Integration Analysis

When possible, create an isolated synthetic merge:

Target Branch
+
PR Source

Do not push this result.

Use it to detect:

- conflicts
- build failures
- type incompatibilities
- dependency conflicts
- integration regressions

---

# Phase 14 — Previous Review Analysis

Retrieve previous review comments.

Determine:

- resolved findings
- unresolved findings
- developer responses
- changed code
- regressions

Do not repeat resolved findings unnecessarily.

---

# Phase 15 — Findings

Classify:

BLOCKING
IMPORTANT
SUGGESTION
QUESTION
PRAISE

Each finding must contain:

- location
- problem
- impact
- evidence
- recommendation

---

# Phase 16 — Review Summary

Generate:

## PR REVIEW

### Verdict

Choose:

- Approve
- Approve with suggestions
- Request changes
- Unable to review

### Findings

...

### Positive Observations

...

### Risks

...

### Validation

...

### Reviewed Commit

...

---

# Phase 17 — Human Decision

Read-only:

Return the review.

Comment mode:

Request authorization before publishing comments.

Fix mode:

Request authorization before changing code.

---

# Phase 18 — Publish

After authorization:

- publish inline comments
- publish summary
- preserve previous findings
- do not approve automatically unless explicitly requested

---

# Phase 19 — Cleanup

Remove the temporary review worktree when safe.

Never remove:

- user work
- project branches
- PR branches
- committed repository data