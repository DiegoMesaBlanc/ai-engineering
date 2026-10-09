---
name: pull-request
description: Prepare concise, accurate, copy-ready Pull Request or Merge Request titles, descriptions, testing summaries, and reviewer notes without creating or publishing the PR.
---

# Pull Request Preparation

## Purpose

Prepare a high-quality Pull Request / Merge Request proposal for manual
creation by the user.

This Skill generates text.
It does not create or publish PRs/MRs.

## Core Principles

- Describe actual changes.
- Explain why they were made.
- Link requirements when reliable references exist.
- Report actual validation evidence.
- Highlight significant risks.
- Follow repository conventions.
- Avoid unnecessary verbosity.

## Context Retrieval

Use the cheapest reliable source first:

1. local Git status, log, and diff
2. repository PR template and contribution guidelines
3. supplied task and design references
4. authorized read-only provider integrations

Do not require MCP when local files and Git provide sufficient evidence.

Do not inspect the entire repository when a focused set of files is enough.

## Analysis

Determine:

- changed files and modules
- functional behavior
- architectural impact
- new or removed dependencies
- database or configuration changes
- testing performed
- validation not performed
- potential risks
- relevant breaking changes

Compare the implementation with task acceptance criteria when available.

Only include design or architecture notes when they add useful information.

## Output

Produce:

### Title

A concise title that follows repository conventions.

### Description

Summary:
Explain what changed and why.

Task:
Reference the actual task when available.

Changes:
Summarize meaningful implementation details.

Design:
Include relevant UI/design decisions when applicable.

Architecture:
Explain significant architectural impact, not routine details.

Testing:
List validations actually performed.

Risks:
Describe meaningful risks or limitations.

Reviewer Notes:
Point out important decisions or areas requiring attention.

Checklist:
Use the project's PR template when present.

## Validation Honesty

Distinguish among:

- executed successfully
- executed with failures
- not executed
- unavailable

Never infer successful execution from the existence of a test file.

## Strict Read-Only Policy

Do not:

- create a PR/MR
- call provider write operations
- publish text or comments
- push or merge branches
- approve a PR
- update a task

Return copy-ready content for manual use.
