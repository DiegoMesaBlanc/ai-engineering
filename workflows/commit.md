# Commit Workflow

## Purpose

Generate and validate a commit proposal from the current change set using
project-native quality gates, without pushing or deploying. Commits are
created only after all gates pass **and the user explicitly approves the
proposed commit groups and messages in the current conversation, after
reviewing the presented change set**. This workflow is the canonical procedure
for local commit generation; `commands/commit.md` is the OpenCode adapter that
delegates here.

Invoking `/commit` authorizes preparing and validating the proposal, not
creating commits. There is no implicit or standing commit authorization.

The request that started the work is not commit approval either. A prompt,
task, ticket, or argument that says "commit", "haz el commit", or anything
equivalent still receives the proposal and the approval question first,
because approval cannot exist before the change set the user has to review.

This workflow is **not** a code architecture audit; it validates project
correctness (format, lint, typecheck, tests, build, security) at the level
of a typical Husky pre-commit hook. It does not evaluate architecture,
patterns, or over-engineering.

## Scope and Classification

1. Resolve the project root.
2. Classify the repository:
   - Software project with application source code: all gates apply.
   - Documentation-only knowledge repository: only Git hygiene and commit
     message rules apply; no formatter, linter, typecheck, test, build, or
     security gates.
3. If the working tree is clean (no staged, unstaged, or untracked files):
   report `NOTHING TO COMMIT` and exit.

## Phase 1 — Inspect and Build Scope Manifest

- Run `git status --porcelain=v1`.
- Run `git diff --name-status -M HEAD`.
- Run `git diff --unified=0 HEAD`.
- Run `git ls-files --others --exclude-standard`.
- Record the exact scope: paths, branch, commit, changed line ranges.
- Exclude by default: `node_modules`, `dist`, `.next`, `build`, `coverage`,
  lockfiles, binaries, minified assets. Migrations and config files are
  included.
- Deleted files: record as context only.
- **The approved scope manifest is required (M-90).** It is the list that is recorded at
  alignment before the first file write, in `workflows/task-to-code.md`, and it is read
  from the task checkpoint.
  A task with a change set and no recorded manifest is `BLOCKED`: the proposal stops
  and the gap is reported, never inferred from the diff. An analysis-only task has an
  empty manifest, so any changed file is a `SCOPE-001` `FAIL` (M-91).
- Run `scope_file_check` (`SCOPE-001`, tracked and untracked files) against the
  manifest, and `out_of_scope_commit_check` over `base..HEAD` (`COMMIT-002` in
  `workflows/output-compliance.md`). A proposed commit or pull request carries only the
  commits of the approved scope: a commit that touches a path the manifest does not
  name is a `FAIL` that blocks the proposal (M-87). The base is the revision the
  approved scope was measured against, not the first commit of the branch.
- **Teardown gate (M-93).** Run `teardown_gate_check` (`TEARDOWN-001`) over the task
  checkpoint. A step recorded `NOT DONE` that the user has neither resolved nor accepted
  in writing blocks the commit proposal until the user resolves it or accepts it in
  writing, and it is listed in the proposal with its reason.
- **Approval citations (M-92).** Run `approval_citation_check` (`APPROVAL-001`) over the
  proposal, the decision records, and the checkpoint. An approval claim without the
  quoted user message is `NOT VERIFIED` and blocks the proposal.

If `--staged` is passed, limit scope to `git diff --name-only --cached`.

If explicit paths are passed, use those as the scope.

## Phase 2 — Gates Delegated to Project Validation

The formatter, linter, type-check, test, and build gates (Phases 4 to 8) are
defined once, in `workflows/project-validation.md`. This workflow does not
restate them. Run that workflow over the scoped change set:

- its Phase 2 detects the tooling and records each tool's lifecycle state;
- its Phase 5 verifies the mandatory formatter and linter;
- its Phase 7 executes the checks and verifies that the linter actually runs.

A mandatory tool that is missing is `SETUP_REQUIRED` and blocks the commit,
as `workflows/project-validation.md` defines. Do not install tools or modify
configuration without explicit user authorization.

**Structural compliance is a blocking gate of this workflow (M-08).** Run the
structural compliance check of `workflows/output-compliance.md` over the scoped
change set, as the project's validation requires: every detector whose "Applies
when" row the change set meets, with its `file:line` output. A `FAIL` in the
change set blocks the commit exactly as a failing linter does, and it is never
downgraded to a warning in Phase 10. A `FINDING` (a violation that already existed
in the base revision) is listed in the proposal and does not block. The check is
not optional for a software project with source code: a commit gate that skips it
lets a structural defect reach the branch unseen.

## Phase 3 — Secrets Scan

Scan the diff (staged + unstaged + untracked in scope) for common secret
patterns: API keys, tokens, passwords, private keys, connection strings.

```sh
grep -rnE "(sk_live_|sk_test_|AKIA[0-9A-Z]{16}|ghp_[A-Za-z0-9]{36}|-----BEGIN (RSA|EC|OPENSSH) PRIVATE KEY-----|(password|passwd|secret|token|api[_-]?key)[[:space:]]*[:=][[:space:]]*[\"'][^\"']{8,})" \
  <scoped-paths> \
  --exclude-dir=node_modules --exclude-dir=dist --exclude-dir=build \
  --exclude-dir=coverage --exclude-dir=.git
```

Each match is a candidate: confirm it is not a placeholder, an example, or a
test fixture. A real secret is reported with its location and type, redacted,
and **blocks** all commits.

If secrets are found: report location and type (redacted), **block** all
commits.

## Phase 4 — Format Check

Run the formatter check of `workflows/project-validation.md` (Phase 7) over
the scoped files. A failing check blocks commits. Do not auto-format; the user
must run the formatter separately.

When the project also has a whole-project script (for example
`npm run format:check`), run it too and attribute every failing file: a file in
the change set blocks the commit; a file outside it is a `BASELINE_FAILURE` of the
health probe (`workflows/project-validation.md`, Phase 8a), reported with its count
and first location, and does not block. A red whole-project run with no failing
file in the change set is never reported as the commit's gate failing, and never as
passing.

## Phase 5 — Lint

Run the linter of `workflows/project-validation.md` (Phases 5 and 7). A failing
lint blocks commits. A missing linter for a software project is
`SETUP_REQUIRED` and blocks commits.

## Phase 6 — Type Check

Run the type check of `workflows/project-validation.md` (Phase 7). A failure
blocks commits. A typecheck whose errors are all under `node_modules` is a
configuration failure under `workflows/project-validation.md` (Phase 8), and it
blocks commits in the same way.

## Phase 7 — Tests

Select the test depth as `workflows/project-validation.md` does (Phases 3 and
4). Unit tests run by default; integration and end-to-end suites run when the
request passes `--full` or the validation selects them. A failing test blocks
commits. A project with no test script reports `NOT_CONFIGURED` and continues.

## Phase 8 — Build / Compile

Run the build of `workflows/project-validation.md` (Phase 7) when a build is
configured. A failure blocks commits.

## Phase 9 — Security

If security audit commands are configured:

- `npm audit` / `yarn audit` / `pnpm audit` / `pip-audit` / `cargo audit`
  / `composer audit` / `bundler-audit` / `gradle dependencyCheck` / etc.
- SAST tools if installed (`semgrep`, `sonarqube`, `codeql`, etc.) — only
  if already present in the project; never install.

If a configured audit reports critical or high findings: report each one with
its fixed version when one exists. A finding does not block the commit; it is a
finding of the health probe (`workflows/project-validation.md`, Phase 8a) and is
listed in the commit proposal.

If no security tooling is configured: report `NOT_CONFIGURED` and continue.

## Phase 10 — AI Sanity Review

Run a lightweight review of the staged diff:

- Check for obvious correctness issues: broken imports, undefined
  references, missing error handling, hardcoded secrets not caught by
  Phase 3, missing tests for new logic, type assertions without
  verification.
- Do **not** evaluate architecture, design patterns, folder structure,
  or over-engineering.
- Severity: block only on clear P0/P1 correctness or security defects.
- Report P2/P3 as warnings; do not block.

This is distinct from `/code-audit` (which is a senior/staff whole-cycle
audit). The AI sanity review is a fast correctness gate.

## Phase 11 — Atomic Commit Grouping

Group the scoped changes into atomic commits:

- One logical change per commit.
- Separate unrelated changes.
- Every file in the change set is named in its approved scope manifest (M-71).
  Run `scope_file_check` (SCOPE-001 in `workflows/output-compliance.md`) against
  the manifest before grouping. A file it reports that the manifest does not name
  is removed from the proposal. When its diff is a formatter or linter rewrite
  alone, that rewrite is reverted for that file; any other change in it is
  preserved and reported to the user, never discarded.
- Do not create commits merely because a file changed.
- Prefer small, atomic commits.
- Stage only the intended files/hunks for each commit.
- Check the closure of every group with `commit_group_closure_check` (COMMIT-001 in
  `workflows/output-compliance.md`), passing the files of that group. Every relative
  import and every decision-record citation of a group must resolve inside the group
  or at HEAD. A group whose file depends on an uncommitted file of another group is
  not proposed as it stands: either the dependency moves into the group that needs
  it, or the dependency is committed first as its own group. Each group's closure
  result is shown in the Phase 13 proposal.

## Phase 12 — Commit Message Generation

For each commit group, generate a commit message:

- **Always in English.**
- Conventional Commits format unless the project defines another
  convention: `<type>(<scope>): <description>`.
- Imperative mood.
- Concise (subject ≤ 72 chars).
- Describe the actual change.
- Do not mention AI.
- Do not claim tests or behavior that were not actually performed.
- Do not include unrelated work.

If the project uses a different convention (e.g., Jira ticket prefix),
respect it but keep the message in English.

## Phase 13 — Proposal and Approval Gate

Do not create any commit in this phase.

Present the complete proposal, including the change set the user has to
review:

Every factual statement in the proposal — a file count, a removed or added
dependency, an advisory identifier, a version number, or a claim that an
artifact was deleted — is derived from the evidence it cites: the diff or the
tool output that produced it, and is re-derived from that evidence before it is shown. A
statement copied from memory, from an earlier draft, or from a previous report
is not evidence (M-65).

- Scope Manifest: files, line ranges, exclusions.
- The commits of `base..HEAD` outside the approved scope (`COMMIT-002`), and the
  rebase or written acceptance the user chose. An open `COMMIT-002` `FAIL` blocks the
  proposal (M-87).
- The structural compliance result of Phase 2: each detector with its `file:line`
  output, its `FAIL` count, and its `FINDING`s listed as pre-existing (M-08).
- The code-audit dimension table, when the change set has one: every applicable
  dimension with its verdict and evidence. A dimension that is `NOT VERIFIED`, a
  dimension that was not run, or a verdict `BLOCKED` is shown as blocking, and the
  proposal does not ask for commit approval until each of them is resolved (M-39, M-74).
  The proposal never reports a dimension as passed that the table leaves open.
- The changed values of existing behavior from `SCOPE-002` (M-79): each candidate is
  named in the manifest, or it is a blocking item for the approval question.
- The diff of each proposed group, so the actual adjustments can be reviewed.
- Tooling lifecycle states and gate results.
- Each proposed commit group: files/hunks, message, rationale, and its closure
  result (COMMIT-001: every import and citation resolves inside the group or at HEAD).
- The attribution of every failing gate file: in the change set, or `BASELINE_FAILURE`.
- Warnings, skipped checks, and anything that could not be verified.

Then ask explicitly:

> Do you approve creating these commits? Review the proposal above and reply
> with an explicit approval (for example: "yes, create the commits").

Rules:

- Invoking `/commit` is authorization to prepare this proposal, not to
  create commits.
- A prompt, task, ticket, argument, or earlier message that orders the commit
  ("commit", "haz el commit", "commit and push", or equivalent) is **not
  commit approval**. Do not create anything: present the proposal and ask.
- Approval only counts when it answers this question in the current
  conversation, after the proposal was presented. An approval expressed
  before the proposal exists is not approval: treat it as absent and ask
  again.
- The user must be able to review the adjustments. A proposal without the
  change set is not approvable, and an approval given without it is not
  valid.
- Silence, a timeout, a resumed checkpoint, an automated retry, a general
  request to implement the task, or a previous approval of the implementation
  is **not** commit approval.
- Do not run `git commit` or `git commit --amend` until the user replies
  affirmatively to this question in the current conversation.
- Approval covers exactly the presented groups, files, and messages.
- If the user declines or requests changes, do not commit; adjust the
  proposal and ask again.
- If the approved scope, grouping, or message differs from what was
  presented, re-present the proposal and ask again before creating anything.
- If any gate failed in Phases 2–10, or a structural `FAIL` or an open `COMMIT-002`
  `FAIL` is in the change set, do not request approval to commit; report the
  failures and stop.
- If `SCOPE-001`, `TEARDOWN-001`, or `APPROVAL-001` reports a `FAIL` or `NOT VERIFIED`
  (M-90 to M-93), or the approved scope manifest is missing, do not request approval to
  commit either: report the item with its `file:line` and stop.

## Phase 14 — Create Approved Commits

Only after the affirmative answer to the Phase 13 question, for each approved
group:

1. Stage exactly the approved files/hunks.
2. Verify no secrets in the staged diff.
3. Create the commit with the approved message.
4. Never push.

If the working tree or the staged diff changed since the proposal, stop,
re-run the affected gates, and re-present the proposal before creating any
commit.

If any gate fails at this point: **do not create any commits**; report the
failure.

## Phase 15 — Report

Produce a summary:

- Scope Manifest (files, line ranges, exclusions).
- Tooling lifecycle states.
- Gate results: command, status (PASS/FAIL/BLOCKED/SETUP_REQUIRED/
  NOT_CONFIGURED), duration.
- Approval status and the approved scope.
- Commits created (hash, message, files).
- Warnings from AI sanity review.
- Checks skipped and why.
- Next steps if blocked or not approved.

## Read-only Policy (for dry-run)

When `--dry-run` is passed:

- Execute all gates.
- Show proposed commit groups and messages.
- Stop after the Phase 13 proposal; do not request commit approval.
- Do not stage, commit, or modify the working tree.
- Report everything as if committing, but with "DRY RUN" prefix.