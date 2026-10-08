# /review-pr

Perform a context-aware Pull Request / Merge Request review.

---

## Goal

Review the actual PR implementation as a Senior Engineer, Staff Engineer,
Tech Lead, or Architect according to the complexity of the change.

Do not review only the diff.

---

## Behavior

1. Detect Git provider.
2. Identify the PR/MR.
3. Retrieve PR metadata.
4. Retrieve source branch.
5. Retrieve target branch.
6. Retrieve exact source commit.
7. Retrieve associated task.
8. Retrieve associated design.
9. Inspect current local Git state.
10. Preserve the user's active workspace.
11. Create an isolated review worktree when possible.
12. Checkout the exact PR source commit.
13. Analyze the repository.
14. Analyze architecture.
15. Analyze the PR diff.
16. Analyze the task requirements.
17. Analyze the design.
18. Analyze testing.
19. Analyze security.
20. Analyze performance.
21. Analyze integration impact.
22. Run appropriate validations.
23. Perform synthetic merge analysis when possible.
24. Generate findings.
25. Present review summary.

---

## Review Modes

### Read Only

Default mode.

Analyze and report findings.

Do not modify the PR.
Do not publish comments.

---

### Review and Comment

Analyze the PR and ask for approval before publishing findings.

Example:

Found:

2 BLOCKING
1 IMPORTANT
2 SUGGESTIONS

Publish these findings to the PR?

---

### Review and Fix

Only when explicitly requested.

Flow:

Review
→ Findings
→ Human Approval
→ Fix
→ Tests
→ Re-review
→ Commit

Do not push changes automatically.

---

## Review Context

Use:

- Task Context
- Design Analysis
- Project Analysis
- Stack Detection
- Architecture Advisor
- Complexity Advisor
- Testing Strategy
- Engineering Review

Load only the Skills relevant to the PR.

---

## Findings

Classify findings as:

BLOCKING
IMPORTANT
SUGGESTION
QUESTION
PRAISE

Optional priorities:

P0 Critical
P1 High
P2 Medium
P3 Low

Each finding must explain:

- what is wrong
- why it matters
- evidence
- recommendation

Avoid vague comments.

---

## Final Output

PR REVIEW

Provider:
...

PR:
...

Reviewed commit:
...

Target:
...

Verdict:
...

Blocking:
...

Important:
...

Suggestions:
...

Architecture:
...

Correctness:
...

Security:
...

Performance:
...

Testing:
...

Design:
...

Integration:
...

Positive observations:
...

Recommended actions:
...

Validation:
...

---

## Safety

Never:

- destroy user changes
- replace the active branch
- review a stale source commit when a newer PR commit exists
- modify code during read-only review
- publish comments without authorization
- approve automatically
- merge automatically

Prefer an isolated Git worktree.