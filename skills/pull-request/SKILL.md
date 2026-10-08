---
name: pull-request
description: Create and prepare Pull Requests or Merge Requests across Git providers using validated repository changes, task context, design context, testing results, and project conventions.
---

# Pull Request

## Purpose

Create high-quality Pull Requests or Merge Requests from validated
repository changes.

The skill is provider-agnostic.

Do not assume GitHub, GitLab, Azure Repos, or Bitbucket.

The Git provider must be detected or explicitly selected.

---

# Core Principle

A Pull Request is the delivery boundary between implementation and review.

The PR must communicate:

1. What changed.
2. Why it changed.
3. How it was implemented.
4. Which task or requirement it addresses.
5. What was tested.
6. What risks remain.
7. What reviewers should pay attention to.

Do not create a PR merely because code was committed.

The implementation must be validated first.

---

# Provider Detection

Determine the Git provider using available repository metadata.

Possible providers include:

- GitHub
- GitLab
- Azure Repos
- Bitbucket
- Other compatible Git providers

Prefer repository configuration and remote URLs over assumptions.

Do not hardcode provider-specific behavior in the core workflow.

Use the appropriate provider integration when available.

---

# Preconditions

Before creating a PR:

1. Inspect repository status.
2. Inspect current branch.
3. Inspect target branch.
4. Inspect commits that belong to the change.
5. Inspect staged and unstaged changes.
6. Check for unrelated changes.
7. Validate the implementation.
8. Check tests.
9. Check typecheck when available.
10. Check lint when available.
11. Check build when appropriate.
12. Check security validation when configured.

Do not create a PR containing unrelated changes.

Do not commit secrets.

---

# Task Context

When the change is associated with a task, retrieve and analyze:

- task identifier
- title
- description
- acceptance criteria
- priority
- comments
- dependencies
- related issues
- attachments
- design references
- technical references

The PR must reflect the actual task.

Do not invent requirements.

---

# Design Context

When a design reference exists:

- retrieve the design context
- analyze relevant screens/components
- identify affected UI behavior
- identify responsive requirements
- identify important states
- identify accessibility considerations
- identify existing project components that should be reused

Do not mention a design source that was not actually available.

The PR should describe relevant design implementation decisions when
useful for reviewers.

---

# Change Analysis

Analyze:

- files changed
- lines changed
- affected modules
- affected features
- architecture impact
- new dependencies
- removed dependencies
- public APIs
- database changes
- configuration changes
- infrastructure changes
- test changes

Determine whether the change is:

- isolated
- cross-module
- architectural
- breaking
- potentially risky

---

# Commit Analysis

Inspect the commits included in the PR.

Prefer logical and meaningful commits.

Detect:

- unrelated commits
- accidental files
- generated files
- debugging code
- temporary changes
- secrets
- meaningless commits

Respect project-specific commit conventions.

Do not rewrite history automatically.

---

# PR Title

Generate a concise title following the repository's existing convention.

When no convention exists, prefer a Conventional Commit style title when
appropriate.

Examples:

feat(checkout): add provider selection

fix(auth): prevent duplicate refresh requests

refactor(payments): simplify provider resolution

Do not mention AI in the title.

---

# PR Description

Generate a structured description when the provider supports it.

Recommended structure:

## Summary

Explain what changed and why.

## Task

Reference the associated task or work item when available.

## Changes

Summarize the implementation.

## Design

Summarize relevant design decisions.

## Architecture

Describe architectural impact.

## Testing

List validations performed.

## Risks

Identify relevant risks.

## Breaking Changes

Explicitly identify breaking changes.

## Reviewer Notes

Highlight areas where reviewers should pay attention.

---

# Validation

Never claim that validation succeeded without evidence.

Possible validations:

- unit tests
- integration tests
- E2E tests
- typecheck
- lint
- build
- security checks
- dependency checks

Respect the existing project tooling.

Do not introduce a new testing framework merely to create a PR.

---

# Review Readiness

Before creating the PR, produce:

PR READY

Title:
...

Target:
...

Source:
...

Task:
...

Summary:
...

Tests:
...

Typecheck:
...

Lint:
...

Build:
...

Security:
...

Potential Risks:
...

Unrelated Changes:
...

Suggested Reviewers:
...

The user must approve PR creation unless an explicit automation policy
has been enabled.

---

# Pull Request Creation

After approval:

1. Ensure the source branch is correct.
2. Ensure the target branch is correct.
3. Ensure the repository provider is correct.
4. Ensure the latest validated commit is the source commit.
5. Create the PR/MR using the provider integration.
6. Return the PR URL and identifier.

Do not silently change the target branch.

Do not automatically merge the PR.

Do not automatically approve the PR.

---

# Reviewer Suggestions

Suggest reviewers only when evidence exists.

Possible signals:

- CODEOWNERS
- repository ownership rules
- file ownership
- previous reviewers
- task ownership
- module ownership

Do not invent reviewers.

---

# Labels

Suggest labels only when supported by repository conventions or provider
metadata.

Do not create arbitrary labels.

---

# Post-Creation

After successful creation report:

PR CREATED

Provider:
...

PR:
...

Source:
...

Target:
...

Commit:
...

URL:
...

Reviewers:
...

Do not automatically merge.

---

# Safety

Never:

- expose credentials
- commit secrets
- force push
- rewrite history
- change unrelated branches
- merge without explicit authorization
- approve your own PR automatically
- claim validation that was not executed

---

# Scope Control

The Pull Request skill does not perform broad refactoring.

If unrelated improvements are discovered:

- report them separately
- do not automatically include them

Prefer a separate task or PR.
