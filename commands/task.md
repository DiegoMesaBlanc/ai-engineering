---
description: Start a development task, analyze its context, ask for the branch name, and implement only after the task branch is ready.
---

Start or continue the engineering task specified by:

$ARGUMENTS

Follow the central Engineering Orchestrator and Task-to-Code workflow.

## Mandatory branch policy

Before modifying application code:

1. Detect the repository and project state.
2. Retrieve and understand the task when possible.
3. Inspect current Git status and identify the correct base branch.
4. Ask the user for the exact branch name if it has not already been
   supplied.
5. Validate that the branch name is valid and not already in use.
6. Create the local task branch or an isolated worktree as appropriate.
7. Verify that work will take place on the requested branch.

Ask exactly:

"What branch name would you like me to use for this task?"

Do not guess the name.

Do not begin implementation before the branch is ready.

Do not discard, stash, reset, or overwrite existing user changes silently.

If the requested branch already exists, stop and ask whether the user
wants to use it or provide another name.

## Normal workflow

After the branch is ready:

1. Analyze task requirements and acceptance criteria.
2. Analyze the relevant design when available.
3. Analyze the repository and detect its stack.
4. Evaluate architecture, repository topology, complexity, and patterns.
5. Present meaningful decisions when human approval is required.
6. Implement using existing project conventions.
7. Test and validate the changes.
8. Review the implementation.
9. Present the Git diff and commit proposal.

## PR policy

Prepare copy-ready PR title and description when requested.

Do not create or publish the PR/MR.

Do not push, merge, publish comments, or update a task remotely.
