---
description: Review a colleague's Pull Request or Merge Request and generate evidence-based, copy-ready feedback grounded in the reviewed revision without modifying code or publishing comments.
agent: reviewer
---

Review this Pull Request / Merge Request:

$ARGUMENTS

## Supported Inputs

The primary input is a Pull Request / Merge Request URL or identifier.

Optional additional references may include:

- task URL or identifier
- design URL
- repository URL or local repository path
- supplied diff or source files

Examples:

/review-pr https://github.com/company/app/pull/142

/review-pr pr=https://dev.azure.com/company/project/_git/app/pullrequest/142 task=https://dev.azure.com/company/project/_workitems/edit/12345

/review-pr pr=https://gitlab.com/company/app/-/merge_requests/142 design=https://example.com/design/456

Resolve each reference independently.

Prefer task and design references already linked to the PR. Use additional
references supplied by the user when necessary.

Review only the relevant code and supporting context. Do not publish review
comments or modify the repository.

The workflow is strictly read-only and copy-only.

## Workflow

1. Identify the PR URL, provider, source/target commit, and author when available.
2. Retrieve the task and relevant design context when available.
3. Inspect current local Git state before checkout.
4. Preserve user changes; use an isolated worktree if local source inspection is required.
5. Inspect the exact PR source revision and base; never substitute a stale branch or unrelated diff.
6. Analyze the diff and only the surrounding repository context needed to validate findings.
7. Select relevant review dimensions for the affected code and stack.
8. Run safe, relevant existing validations when helpful and record actual results.
9. Verify every finding against the surrounding implementation and the reviewed revision before reporting it.
10. Use compact review memory when previous findings exist.
11. Generate the copy-ready report required by `workflows/pull-request-review.md`.

Follow the evidence requirements, severity definitions, and required report
format defined in `workflows/pull-request-review.md`.

## Output

Produce the report defined by the workflow, including the review context,
the findings ordered by severity, the verification gaps, the non-blocking
observations, and the summary recommendation.

Do not restate the report format, the evidence rules, or the verdict list
here. The workflow is the authoritative source; a copy in this adapter
drifts from it.

## Strict Read-Only Policy

Never edit source code, fix findings, publish comments, approve/reject,
merge, push, create a PR, or update tasks. The user decides which comments
to publish manually.
