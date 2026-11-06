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
2. the intended source and target branches
3. repository PR template and contribution guidelines
4. supplied task and design references
5. authorized read-only provider integrations

Do not require MCP when local files and Git provide sufficient evidence.

Do not inspect the entire repository when a focused set of files is enough.

## Analysis

Determine:

- changed files and modules
- unrelated changes that must not be included
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

Every count, name, and claim about the change set in the description (files
changed, a removed dependency, an advisory, a version) is derived from the
evidence it cites: the diff or the tool output, and re-derived before it is reported. A statement
copied from memory or from an earlier draft is not evidence (M-65).

## Responding to Review Feedback

When the user receives review comments on their own change, prepare copy-ready
replies. Do not publish them and do not change code without the user's request.

For each comment:

1. **Verify before agreeing.** Read the cited code and the surrounding callers,
   then reproduce or test the claim when it is behavioral. Accept it only when
   the evidence supports it.
2. **Classify the response**: accepted and fixed, accepted and deferred (with a
   reason and a follow-up), declined with evidence, or needs clarification.
3. **Push back with evidence** when the comment is wrong. Cite the test, the
   requirement, or the code that shows the actual behavior.
4. **Avoid performative agreement.** Do not reply "good catch" or "you're
   right" before the claim has been checked. Reply with what was verified.
5. **Keep the scope.** A request for an unrelated change is a new item, not an
   implicit requirement of this PR.

Output one block per comment: the comment reference, the verdict, the evidence,
and the reply text.

## Strict Read-Only Policy

Do not:

- create a PR/MR
- call provider write operations
- publish text or comments
- push or merge branches
- approve a PR
- update a task

Return copy-ready content for manual use.
