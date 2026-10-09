# /review-pr

Review an existing Pull Request / Merge Request and generate actionable
feedback for a human reviewer.

This command is strictly read-only.

## Input

Accept:

- a PR/MR URL
- provider and PR identifier
- a local repository path and PR identifier
- a source branch and target branch
- a supplied diff or source files

Infer available context before asking the user.

## Workflow

1. Identify the PR and provider.
2. Retrieve the source and target commits when accessible.
3. Retrieve task context when available.
4. Retrieve design context when available.
5. Inspect the current local Git state before any checkout.
6. Preserve the user's working tree.
7. Use an isolated worktree when local source inspection is necessary.
8. Analyze repository architecture and relevant conventions.
9. Analyze the complete relevant diff.
10. Inspect related existing code and abstractions.
11. Evaluate correctness, requirements, architecture, security,
    performance, maintainability, accessibility, and testing.
12. Run safe, relevant validations when appropriate.
13. Check previous review findings when available.
14. Generate copy-ready feedback grouped by file and line.
15. Provide a concise overall assessment.

## Review Output

Start with:

### REVIEW SUMMARY

PR:
Author:
Source commit:
Target:
Scope:
Overall assessment:

Then report each actionable finding.

### FINDING

Priority: P0 / P1 / P2 / P3
Type: BLOCKING / IMPORTANT / SUGGESTION / QUESTION
Category:
File:
Line or range:
Evidence:
Impact:
Confidence:

#### Comment to copy

<Exact proposed review comment>

#### Recommendation

<Optional explanation for the human reviewer>

Use source-commit line numbers and anchor feedback to changed lines whenever
possible.

If a concern is outside the diff, explain the connection to the change and
identify an appropriate changed-line anchor when possible.

## Review Language

Use the team's existing PR language when it can be determined.

Otherwise, provide explanations in the user's preferred language and
copy-ready comments in English by default.

## Safety

Never:

- publish comments or reviews
- approve or reject the PR through the provider
- modify source code
- push changes
- merge the PR
- update tasks
- change branches in the user's primary worktree
- delete user files or changes

When a local checkout is needed, use an isolated worktree or another safe,
read-only approach.

## No-Finding Result

If no actionable issues are found, say so explicitly.

Do not invent findings merely to produce comments.