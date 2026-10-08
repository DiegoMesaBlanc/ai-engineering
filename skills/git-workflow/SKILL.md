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
