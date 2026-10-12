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

Produce the required report:

### Review context

- PR or change identifier.
- Reviewed branch/revision and base, when known.
- Task or acceptance criteria used.
- Scope and evidence sources.
- Checks actually executed.

### Findings

Order findings by severity, then confidence. For each:

- `[Severity] Short title` (Critical / High / Medium / Low)
- `File:line` verified against the reviewed revision
- **Evidence:** verified code path, requirement, or execution result.
- **Impact:** concrete failure mode.
- **Recommendation:** actionable correction.
- **Confidence:** High / Medium / Low.

If no actionable defects are found, state that explicitly. Do not claim
the change is bug-free.

### Verification gaps

Record checks that could not be performed and why.

### Non-blocking observations

Include only observations that do not meet the bar for a confirmed defect.

### Summary

Choose one evidence-supported recommendation:

- Request changes.
- Review comments / follow-up recommended.
- No actionable findings identified.

Do not say "Approved" or imply that a remote PR state was changed.

## Evidence Rules

Do not invent line numbers, test results, runtime behavior, requirements,
or design specifications.

Verify line numbers against the reviewed revision; inspect the file when
the diff lacks context.

Separate confirmed defects from concerns that require human verification.

Do not report stylistic preferences as defects unless an explicit project
rule or requirement supports them.

## Strict Read-Only Policy

Never edit source code, fix findings, publish comments, approve/reject,
merge, push, create a PR, or update tasks. The user decides which comments
to publish manually.
