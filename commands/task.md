---
description: Start a task, understand its requirements and design, ask for the branch name, and implement only after the task branch is ready.
agent: orchestrator
---

Start or continue the engineering task specified by:

$ARGUMENTS

Follow the central Engineering Orchestrator and Task-to-Code workflow.

## Mandatory branch policy

Before modifying application code:

1. Detect the repository and project state.
2. Retrieve and understand the task when possible.
3. Inspect current Git status and identify the correct base branch.
4. Ask for the exact branch name if one has not been provided.
5. Validate the name and check whether the branch already exists.
6. Preserve uncommitted work and prepare the requested branch or isolated worktree.
7. Verify the working location before implementation.

Ask exactly:

"What branch name would you like me to use for this task?"

Do not guess the name.

Do not begin implementation before the branch is ready.

Do not discard, stash, reset, or overwrite user changes silently.

If the branch already exists, stop and ask whether to use it or choose another name.

## Normal workflow

After the branch is ready:

1. Analyze acceptance criteria.
2. Analyze the relevant design when available.
3. Analyze repository and stack.
4. Evaluate architecture, repository topology, complexity, and patterns.
5. Present meaningful decisions when approval is required.
6. Implement following current conventions.
7. Validate and review.
8. Present the Git diff and commit proposal.

## PR policy

Generate copy-ready PR title and description when requested.

Do not create or publish a PR/MR.

Do not push, merge, publish comments, or update a remote task.
