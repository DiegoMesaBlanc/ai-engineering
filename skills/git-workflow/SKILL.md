---
name: git-workflow
description: Manage Git changes safely, review diffs, organize atomic commits, and generate accurate conventional commit messages based on the actual changes.
---

# Git Workflow

## Purpose

Maintain a clean, understandable Git history.

## Before modifying Git state

Inspect:

- current branch
- working tree status
- staged changes
- unstaged changes
- recent commits
- repository contribution conventions

Never assume the working tree is clean.

## Protect user changes

Never:

- discard unrelated user changes
- reset files without approval
- force push without explicit approval
- rewrite history without explicit approval
- commit secrets
- commit environment credentials

## Commit strategy

Prefer small, atomic commits.

One commit should represent one logical change.

Do not create commits merely because a file changed.

Group related changes.

Separate unrelated changes.

## Commit message

Use Conventional Commits when the project does not define another convention.

Format:

<type>(<scope>): <description>

Examples:

feat(auth): add refresh token validation

fix(payments): handle provider timeout

refactor(users): extract validation service

test(payments): add provider integration tests

docs(api): document payment endpoint

chore(ci): update test pipeline

## Message rules

- English only.
- Imperative mood.
- Concise.
- Describe the actual change.
- Do not mention AI.
- Do not claim tests or behavior that were not actually performed.
- Do not include unrelated work.

## Before commit

1. Inspect git status.
2. Inspect the diff.
3. Identify unrelated changes.
4. Verify relevant tests.
5. Verify type checking when available.
6. Verify linting when available.
7. Stage only the intended files.
8. Show the proposed commit message.
9. Create the commit only when authorized by the user or project workflow.

## Commit grouping

If changes contain multiple logical units, propose multiple commits.

Example:

Commit 1:
feat(payments): add payment provider abstraction

Commit 2:
test(payments): add provider integration tests

Commit 3:
docs(payments): document provider configuration

## Never

Never create meaningless commits such as:

- update files
- changes
- fixes
- work in progress

Never bundle unrelated changes into one commit.

## Task Branch Lifecycle

Every implementation task must have an explicit working branch.

### Before Implementation

1. Inspect the current branch.
2. Inspect working-tree status.
3. Identify the repository's default branch or the explicitly required
   base branch.
4. Read and understand the task.
5. Ask the user for the exact task branch name if it has not already
   been provided.

Use this question:

"What branch name would you like me to use for this task?"

Do not invent the branch name.

Do not modify application code until the user has provided the name and
the working branch is ready.

### Validate the Name

- Follow the project's existing branch naming conventions.
- Validate the proposed name using Git's branch-name validation when
  available.
- Do not silently rename the branch.
- Do not overwrite or reuse an existing branch without explicit
  authorization.

### Select the Base

Prefer the task's explicitly required base branch when available.

Otherwise, use the repository's detected default branch.

If the correct base cannot be established, ask before creating the branch.

### Preserve Existing Work

Inspect uncommitted changes before switching branches.

Never discard user changes.

If the working tree is dirty, preserve the primary workspace and prefer
an isolated worktree for the new task branch when safe.

If the task depends on uncommitted changes, ask how the user wants those
changes incorporated.

Never stash, reset, or overwrite user work silently.

### Create and Verify

After the name is provided:

1. Check whether the branch already exists.
2. Check the intended base.
3. Create the task branch, or an isolated worktree if needed.
4. Verify the active branch or worktree.
5. Confirm the working location.
6. Begin implementation only after verification.

### New Projects

For a new project that has no Git history, obtain the branch name before
generating the task implementation.

Initialize the repository and use the requested branch as the initial
working branch when no established base branch exists.

### Remote Operations

Creating a local task branch is part of the authorized task workflow once
the user provides its name.

Do not push the branch unless separately requested.

Do not create a PR/MR. Prepare copy-ready PR content only.
