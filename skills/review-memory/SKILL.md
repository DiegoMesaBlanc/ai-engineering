---
name: review-memory
description: Maintain compact, local-only review state across PR revisions to identify resolved findings, unresolved issues, regressions, and new changes without repeating the entire previous review.
---

# Review Memory

## Purpose

Preserve a compact record of previous review iterations.

The memory is local and optional.

It must never publish review content or modify a remote repository.

## Storage

Use a local application-state directory outside:

- the project repository
- the ai-engineering source repository
- tracked project files

Use a structure similar to:

reviews/
provider/
repository/
pr-identifier.md

Do not assume that local state is available on another machine.

If persistence is unavailable, return the review normally without it.

## Store Only Useful Metadata

Store:

- provider and repository identifier
- PR identifier
- last reviewed source commit
- relevant target commit
- finding identifier
- file and approximate location
- finding category and priority
- concise finding summary
- status: open, resolved, changed, or superseded
- last review iteration

Avoid storing:

- entire diffs
- full source files
- credentials or tokens
- full task descriptions
- unrelated repository information
- complete repeated review reports

Store only enough information to recognize previous findings.

## Review Iteration

When a new PR commit is available:

1. Load only the memory for the requested PR.
2. Identify the previous reviewed commit.
3. Inspect the new changes.
4. Match existing findings against the current code.
5. Mark findings resolved when evidence supports resolution.
6. Keep unresolved findings active.
7. Detect regressions and newly introduced issues.
8. Avoid repeating unchanged findings unnecessarily.
9. Update the compact local review state when permitted.

Never mark a finding resolved only because its line number changed.

## Finding Identity

Identify findings using a combination of:

- category
- affected behavior
- file or component
- concise normalized description

Line numbers alone are insufficient because code changes can move findings.

## Privacy and Retention

Keep review memory local.

Do not upload memory to external services.

Do not include private implementation details unless necessary to identify
a finding.

Allow local review memory to be deleted without affecting the repository.

## Token Efficiency

Never reload the complete previous review merely to recover finding status.

Load the compact memory and retrieve additional context only when needed.

## Human Control

Memory does not make review decisions automatically.

The reviewer must reassess unresolved findings against the current code.
