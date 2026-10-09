---
description: Review a colleague's Pull Request or Merge Request and generate evidence-based, copy-ready feedback by file and line without modifying code or publishing comments.
agent: orchestrator
---

Review this Pull Request / Merge Request:

$ARGUMENTS

The workflow is strictly read-only.

## Workflow

1. Identify the PR URL, provider, source/target commit, and author when available.
2. Retrieve the task and relevant design context when available.
3. Inspect current local Git state before checkout.
4. Preserve user changes; use an isolated worktree if local source inspection is required.
5. Inspect the exact PR source commit and relevant target context.
6. Analyze the diff and only the surrounding repository context needed to validate findings.
7. Evaluate correctness, requirements, architecture, design patterns, security, performance, maintainability, accessibility, testing, and integration risk.
8. Run safe, relevant existing validations when helpful.
9. Use compact review memory when previous findings exist.
10. Generate feedback grouped by file and line.

## Output

### REVIEW SUMMARY

- PR:
- Author:
- Source commit:
- Target:
- Scope:
- Overall assessment:

For every actionable finding include:

- Priority: P0 / P1 / P2 / P3
- Type: BLOCKING / IMPORTANT / SUGGESTION / QUESTION
- Category
- File and exact line/range when available
- Evidence and impact
- Confidence

#### Comment to copy

<Concise, respectful, actionable review comment>

#### Why it matters

<Explanation for the human reviewer>

#### Recommendation

<Optional implementation direction>

Do not invent line numbers or test results. If the source is inaccessible, identify the missing artifact and continue only with reliable available context. If no actionable issues are found, state that explicitly.

## Strict Read-Only Policy

Never edit source code, fix findings, publish comments, approve/reject, merge, push, create a PR, or update tasks. The user decides which comments to publish manually.
