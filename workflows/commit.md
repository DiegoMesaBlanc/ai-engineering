# Commit Workflow

## Purpose

Generate and validate a commit proposal from the current change set using
project-native quality gates, without pushing or deploying. Commits are
created only after all gates pass **and the user explicitly approves the
proposed commit groups and messages in the current conversation**. This
workflow is the canonical procedure for local commit generation;
`commands/commit.md` is the OpenCode adapter that delegates here.

Invoking `/commit` authorizes preparing and validating the proposal, not
creating commits. There is no implicit or standing commit authorization.

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

If `--staged` is passed, limit scope to `git diff --name-only --cached`.

If explicit paths are passed, use those as the scope.

## Phase 2 — Detect Tooling and Project Convention

Detect the actual stack, package manager, and available scripts:

- Lockfile → package manager (`npm`, `yarn`, `pnpm`, `pip`, `poetry`, `uv`,
  `cargo`, `maven`, `gradle`, etc.).
- `package.json` scripts: `format`, `format:check`, `lint`, `typecheck`,
  `test`, `test:unit`, `test:integration`, `test:e2e`, `build`,
  `audit`, `security`.
- Python: `ruff format --check`, `ruff check`, `mypy`, `pytest`, `pip-audit`.
- Java: `checkstyle`, `spotless`, `mvn test`, `mvn verify`.
- Go: `gofmt`, `go vet`, `go test`, `golangci-lint`.
- Rust: `cargo fmt --check`, `cargo clippy`, `cargo test`, `cargo audit`.
- C#/.NET: `dotnet format`, `dotnet build`, `dotnet test`.
- Ruby: `rubocop`, `bundle exec rspec`, `bundler-audit`.
- PHP: `php-cs-fixer`, `phpstan`, `phpunit`, `composer audit`.

Record each tool's lifecycle state: detected, configured, installed,
executed, approved — separately from check status.

If a mandatory tool (formatter, linter for software projects) is missing,
report `SETUP_REQUIRED` and do not proceed to commit. Do not install
tools or modify configuration without explicit user authorization.

## Phase 3 — Secrets Scan

Scan the diff (staged + unstaged + untracked in scope) for common secret
patterns: API keys, tokens, passwords, private keys, connection strings.
Use regex patterns; no external tool required.

If secrets are found: report location and type (redacted), **block** all
commits.

## Phase 4 — Format Check

If a formatter is configured and a non-mutating format check command
exists (e.g., `prettier --check`, `ruff format --check`, `cargo fmt
--check`, `dotnet format --verify-no-changes`, `gofmt -l`):

- Execute the format check on the scoped files.
- If the check fails: report files, **block** commits.
- Do not auto-format; the user must run the formatter separately.

## Phase 5 — Lint

If a linter is configured (mandatory for software projects):

- Execute the linter on the scoped files.
- If lint fails: report errors, **block** commits.
- If no linter is configured for a software project: report
  `SETUP_REQUIRED`, **block** commits.

## Phase 6 — Type Check

If a type checker is configured and applicable:

- Execute the type check command.
- If it fails: report errors, **block** commits.

## Phase 7 — Tests

Execute tests in this order:

1. Unit tests (project's `test:unit` or default `test` script).
2. Integration tests (if `test:integration` exists and `--full` is passed).
3. E2E tests (if `test:e2e` exists and `--full` is passed).

If any test fails: report failures, **block** commits.

If no test script exists, report `NOT_CONFIGURED` and continue.

## Phase 8 — Build / Compile

If a build or compile command is configured:

- Execute the build.
- If it fails: report errors, **block** commits.

## Phase 9 — Security

If security audit commands are configured:

- `npm audit` / `yarn audit` / `pnpm audit` / `pip-audit` / `cargo audit`
  / `composer audit` / `bundler-audit` / `gradle dependencyCheck` / etc.
- SAST tools if installed (`semgrep`, `sonarqube`, `codeql`, etc.) — only
  if already present in the project; never install.

If a configured audit fails with actionable findings: report, **block**
commits.

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
- Do not create commits merely because a file changed.
- Prefer small, atomic commits.
- Stage only the intended files/hunks for each commit.

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

Present the complete proposal:

- Scope Manifest: files, line ranges, exclusions.
- Tooling lifecycle states and gate results.
- Each proposed commit group: files/hunks, message, and rationale.
- Warnings, skipped checks, and anything that could not be verified.

Then ask explicitly:

> Do you approve creating these commits? Reply with an explicit approval
> (for example: "yes, create the commits").

Rules:

- Invoking `/commit` is authorization to prepare this proposal, not to
  create commits.
- Silence, a general request to implement the task, or a previous approval
  of the implementation is **not** commit approval.
- Do not run `git commit` or `git commit --amend` until the user replies
  affirmatively in the current conversation.
- If the user declines or requests changes, do not commit; adjust the
  proposal and ask again.
- If the approved scope, grouping, or message differs from what was
  presented, re-present the proposal and ask again before creating anything.
- If any gate failed in Phases 3–10, do not request approval to commit;
  report the failures and stop.

## Phase 14 — Create Approved Commits

Only after explicit approval in Phase 13, for each approved group:

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