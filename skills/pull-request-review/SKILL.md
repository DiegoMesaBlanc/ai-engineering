---
name: pull-request-review
description: Perform context-aware Pull Request and Merge Request reviews by analyzing the actual PR branch, repository architecture, task requirements, design context, tests, security, performance, and integration impact.
---

# Pull Request Review

## Purpose

Perform a senior-level engineering review of a Pull Request or Merge
Request.

The review must evaluate the complete engineering context, not only the
diff.

The reviewer should reason as a Senior Engineer, Staff Engineer,
Tech Lead, or Architect depending on task complexity.

---

# Core Principle

A PR review is not:

"Review the changed lines."

A PR review is:

Task
→ Requirements
→ Design
→ Repository
→ Architecture
→ PR Branch
→ Diff
→ Tests
→ Integration
→ Risk

The review must evaluate whether the implementation is correct,
appropriate, maintainable, secure, testable, and consistent with the
project.

---

# Step 1 — Identify the PR

Retrieve:

- provider
- PR/MR identifier
- title
- description
- source branch
- target branch
- source commit
- target commit
- author
- linked task
- linked design
- previous review comments
- previous review status

---

# Step 2 — Preserve the User Workspace

Never replace the user's current working branch.

Before checking out PR code:

- inspect repository status
- detect uncommitted changes
- preserve current work
- create an isolated review workspace

Prefer a dedicated Git worktree when Git is available.

Example conceptual structure:

review workspace
└── provider-project-pr-id

The review workspace must be disposable and isolated from the user's
primary checkout.

---

# Step 3 — Retrieve the PR Source

Obtain the exact source commit of the PR.

Prefer:

PR source branch +
source commit

Do not review a stale local branch when a newer remote PR commit exists.

If the provider exposes a PR-specific source reference, use it.

Record the exact commit reviewed.

---

# Step 4 — Retrieve Target Branch

Retrieve the latest target branch state relevant to the PR.

Determine:

- merge base
- changed commits
- files changed
- target branch changes

Do not assume the PR is based on the current local target branch.

---

# Step 5 — Analyze the Actual PR Branch

Switch the isolated review workspace to the PR source commit.

Inspect:

- project structure
- entry points
- affected modules
- dependencies
- architecture
- coding conventions
- existing abstractions
- state management
- API boundaries
- database boundaries
- testing strategy
- design system
- error handling
- logging
- security boundaries

The reviewer must understand the repository before judging the changes.

---

# Step 6 — Task Context

Retrieve the associated task when available.

Analyze:

- business intent
- requirements
- acceptance criteria
- dependencies
- constraints
- comments
- linked tasks

Compare the implementation against the actual task.

Detect:

- missing acceptance criteria
- incomplete behavior
- out-of-scope behavior
- contradictory implementation
- hidden assumptions

---

# Step 7 — Design Context

When a design artifact exists:

- retrieve it
- analyze relevant screens
- compare implementation with design requirements
- verify responsive behavior
- verify important states
- verify accessibility considerations
- identify reused components
- identify design/implementation gaps

The design source may be:

- Figma
- Penpot
- Sketch
- Storybook
- screenshots
- PDF
- image
- HTML/CSS
- other design systems

Do not assume Figma.

---

# Step 8 — Diff Analysis

Inspect the PR diff.

Analyze:

- added code
- removed code
- modified code
- renamed files
- dependency changes
- configuration changes
- database migrations
- tests
- infrastructure changes

Do not evaluate changed lines without repository context.

---

# Step 9 — Architecture Review

Determine whether the implementation:

- follows the current architecture
- bypasses architectural boundaries
- introduces unnecessary coupling
- duplicates existing abstractions
- introduces unnecessary layers
- introduces unnecessary patterns
- increases complexity without value
- creates technical debt
- breaks existing conventions

Prefer the smallest valid architectural change.

---

# Step 10 — Design Pattern Review

Determine whether:

- no pattern is needed
- an existing project abstraction should be reused
- the selected pattern is appropriate
- the selected pattern is implemented correctly
- the pattern introduces unnecessary complexity

Do not recommend patterns merely because they exist.

---

# Step 11 — Correctness

Look for:

- incorrect logic
- edge cases
- null/undefined handling
- incorrect state transitions
- race conditions
- concurrency problems
- incorrect error handling
- transaction issues
- inconsistent state
- invalid assumptions

Prioritize actual bugs over stylistic preferences.

---

# Step 12 — Performance

Frontend:

- unnecessary renders
- expensive calculations
- unnecessary effects
- duplicate network requests
- inefficient lists
- unnecessary bundle growth
- excessive state updates

Backend:

- N+1 queries
- inefficient database access
- unnecessary network calls
- excessive memory usage
- blocking operations
- incorrect caching
- timeout problems
- retry storms
- connection misuse

Only report performance issues when technically justified.

---

# Step 13 — Security

Inspect:

- authentication
- authorization
- input validation
- injection
- XSS
- CSRF
- sensitive data exposure
- secrets
- unsafe deserialization
- dependency risks
- file access
- permission boundaries

Security findings should have concrete reasoning.

---

# Step 14 — Testing

Determine whether the PR has appropriate tests.

Evaluate:

- unit tests
- integration tests
- E2E tests
- edge cases
- failure cases
- regression coverage

Do not require every change to have every kind of test.

Use the smallest sufficient testing strategy.

---

# Step 15 — Validation

When safe and appropriate, execute:

- typecheck
- lint
- tests
- build
- relevant security checks

Do not execute untrusted code blindly.

Never expose credentials or secrets to test processes.

If execution is unsafe or unavailable, report that limitation.

---

# Step 16 — Synthetic Merge Analysis

When possible, create an isolated temporary merge of:

target branch

- PR source

without modifying the remote PR.

Use this to detect:

- merge conflicts
- type errors after integration
- build failures
- dependency conflicts
- incompatible assumptions

Do not push the synthetic merge.

---

# Findings

Every finding must explain:

1. What is wrong.
2. Why it matters.
3. Where it occurs.
4. How it can be improved.

Avoid vague comments.

Bad:

"This is not clean."

Good:

"This service now performs authorization checks directly inside the
controller, while the existing module centralizes authorization in
`AuthorizationService`. Bypassing that boundary can make future
endpoints inconsistent and makes the policy harder to test. Reuse the
existing authorization abstraction."

---

# Severity

Use:

BLOCKING
IMPORTANT
SUGGESTION
QUESTION
PRAISE

Severity should reflect impact, not personal preference.

---

# Priority

Optional priority:

P0 Critical
P1 High
P2 Medium
P3 Low

---

# Review Comments

Inline comments should be:

- specific
- actionable
- technically justified
- concise
- respectful

Prefer comments tied to a precise code location.

Do not produce large numbers of low-value comments.

---

# Review Summary

Return:

## PR REVIEW

### Verdict

- Approve
- Approve with suggestions
- Request changes
- Unable to review

### Findings

- Blocking: N
- Important: N
- Suggestions: N

### Correctness

...

### Architecture

...

### Design / UX

...

### Security

...

### Performance

...

### Testing

...

### Integration Risk

...

### Positive Observations

...

### Recommended Actions

...

### Validation

...

### Reviewed Commit

...

---

# Review Quality Rules

1. Review the real PR source commit.
2. Review the repository, not only the diff.
3. Review the task.
4. Review the design when available.
5. Compare against existing architecture.
6. Prefer evidence over opinion.
7. Report bugs before style.
8. Do not invent requirements.
9. Do not recommend unnecessary architecture.
10. Do not recommend design patterns without value.
11. Do not request tests that provide no meaningful protection.
12. Do not comment on every possible improvement.
13. Distinguish blocking findings from suggestions.
14. Preserve user workspace.
15. Never modify the PR during read-only review.
