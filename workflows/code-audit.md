# Code Audit Workflow

## Purpose

Perform a read-only, evidence-based, senior/staff whole-cycle audit of a generated software solution. It verifies the code is correct, maintainable, appropriately designed (no over-engineering), safe, and evidenced — not merely "it works."

This is **not**:
- `/system-check` (audits the ai-engineering system docs).
- `/validate` (runs formatter, linter, tests, build).
- `/review-pr` (reviews a colleague's PR for copy-ready feedback).

It is an independent, on-demand audit of the generated code itself.

## Senior/Staff Bar

A clean PASS requires more than "it runs":

- **Correct & robust**: edge cases handled, errors explicit, boundaries validated, invalid states unrepresentable.
- **Maintainable**: clear naming, cohesive modules, single responsibility, typed, no dead/debug code; **types/contracts owned by the correct module; imports follow allowed direction**.
- **Appropriately designed**: architecture/patterns justified, no unnecessary abstraction/fragmentation (R-42), contracts before implementation.
- **Safe**: auth, input validation, secrets, supply chain, least privilege.
- **Evidenced**: tests/tools executed or honestly reported; no fabricated findings.

## Applicability-Gated Dimensions

Each dimension maps to an existing Skill. Only applicable dimensions are audited. Every dimension is reported as `PASS` / `WARN` / `FAIL` / `N/A` with an explicit N/A reason.

| # | Dimension | Responsible Skill(s) | Applies when |
|---|---|---|---|
| 1 | **Architecture & folder structure** — style, layering, boundaries, dependency direction, placement, **folder map, type/contract ownership**, cohesion/coupling | `architecture-advisor`, `project-analysis` | Always |
| 2 | **Patterns & complexity** — justified patterns, decision priority, no over-engineering/fragmentation (R-42) | `complexity-advisor` | Always |
| 3 | **Code quality & separation** — naming, SRP, separation of concerns, duplication, dead/debug code, errors, boundary validation, typing, docs, config, logging | `implementation-recipes`, `engineering-review` | Always |
| 4 | **API & contracts** — schemas, error model, versioning, auth, idempotency, no leakage | `api-contract-design` | Boundary/integration changes |
| 5 | **Data & database** — integrity, safe migrations, parameterized queries, indexes, N+1, transactions | `database-engineering` | Schema/query changes |
| 6 | **Security** — authn/authz, input validation, encoding, secrets, supply chain, OWASP basics | `best-practices` | Always |
| 7 | **Testing & validation** — adequacy, behavior over implementation, edge cases, regression, determinism | `testing-strategy`, `test-driven-development` | Always |
| 8 | **Performance** — CWV, bundles, caching; query plans, N+1, concurrency | `performance`, `core-web-vitals` | Web/hot paths |
| 9 | **Accessibility** — WCAG 2.2 A/AA, keyboard, semantics, ARIA, focus, contrast | `accessibility` | UI/web/mobile |
| 10 | **SEO** — meta, canonical, robots, sitemap, structured data, hreflang | `seo` | Public web pages |
| 11 | **Internationalization** — externalized strings, plurals, locale formatting, RTL | `internationalization` | User-facing text |
| 12 | **Frontend/UI** — intentional design, responsive, fidelity | `frontend-design`, `design-analysis` | UI work |
| 13 | **Mobile** — navigation, touch targets, safe areas, keyboard, offline | `design-analysis`, `implementation-recipes` | Mobile work |
| 14 | **Organization boundaries** — package/module boundaries, dependency direction | `repository-strategy` | Multi-package |

## Procedure

### Phase 1 — Resolve and Classify

- Determine the project root (prefer `AI_ENGINEERING_HOME`, then repo root, then `$HOME/Documents/ai-engineering`).
- Classify the repository: software project with application source code, or documentation-only knowledge repository.
- If documentation-only, report `NOT VERIFIED` for code dimensions and exit.

### Phase 2 — Identify Scope

- If `$ARGUMENTS` provides paths, use those.
- If `$ARGUMENTS` includes `--all`, audit the entire project (all application source files).
- Else inspect Git status + diff; audit only uncommitted changes:
  - Staged + unstaged tracked changes (`git diff HEAD`).
  - Untracked files (`git ls-files --others --exclude-standard`).
  - If the working tree is clean: report `NOTHING TO AUDIT` and exit (no fallback to whole project).
- **Default scope is the change set only; declaration sites outside the diff are not audited.**
- Build a **Scope Manifest** recording:
  - `git status --porcelain=v1`
  - `git diff --name-status -M HEAD`
  - `git diff --unified=0 HEAD`
  - `git ls-files --others --exclude-standard`
- Exclude by default: `node_modules`, `dist`, `.next`, `build`, `coverage`, lockfiles, binaries, minified assets.
  - Migrations and config files are included.
- Deleted files: record as context only (no code audit).
- Record the exact scope (paths, branch, commit, changed line ranges).

### Phase 3 — Detect Stack and Architecture

- Run `stack-detection` and `project-analysis` to identify:
  - language, runtime, framework, version, package manager, build system.
  - existing architecture style and conventions (evidence-based, not directory names).
  - app type(s): web frontend, backend API, mobile, CLI, library, data pipeline.
- Record the detection with confidence level.

### Phase 4 — Build Applicability Matrix

For each of the 14 dimensions:
- Check the "Applies when" condition.
- If not applicable, mark `N/A` with a concrete reason (e.g., "no database touched", "not a web-facing surface").
- If applicable, queue the dimension for audit.

### Phase 5 — Load Skills and Audit

For each applicable dimension:
- Load the responsible Skill(s) only (bounded context).
- Audit against the Skill's checks and the senior/staff bar.
- Collect findings with `file:line`, evidence class (REAL/SIMULATED/STATIC), severity (P0–P3), confidence (High/Medium/Low), impact, and recommended correction.
- Discard unproven findings (no invented line numbers, no fabricated test results).

**Dimension 1 — Architecture & folder structure: explicit checks**

1. **Folder Architecture check**
   - Build the Folder Architecture Map from the project (detected) or the approved proposal (if available): layers/modules, responsibility→folder, allowed dependency directions, contract/type location per module, co-location rules, public module API.
   - Map declarations to folders/layers **within the audited scope**.
   - Verify allowed import directions; flag layer-skipping imports (e.g., UI → service implementation for types), mixed-responsibility folders, cross-feature private imports, global `types.ts`/`utils.ts` dumping grounds, circular layer dependencies.
   - Do not impose a universal template; audit conformance to the detected/declared structure. If no structure exists, report "no identifiable architecture" and evaluate whether new code created a coherent one.
   - Language-specific idioms: Go consumer-side interfaces, Java one-public-type-per-file, Python Protocols, Ruby RBS/Sorbet.

2. **Type and Contract Ownership check**
   - Enumerate contract/type declarations **in the audited scope** using language-aware patterns:
     - TS/JS: `interface`, `type`, `enum`, `class` (contract-like), `import type`.
     - Java/Kotlin: `interface`, `record`, `enum`, `class`, `data class`.
     - Python: `class` with `Protocol`/`TypedDict`/`dataclass`, `TypeAlias`.
     - C#: `interface`, `record`, `class`, `struct`.
     - Go: `type ... interface`, `type ... struct`.
     - Rust: `trait`, `struct`, `enum`, `type`.
     - PHP: `interface`, `class`, `enum`.
     - Ruby: Sorbet `sig`, RBS files.
     - SQL: schema/table types (N/A for code interfaces).
   - Map consumers via imports/references/implementations **within the audited scope**.
   - Follow declaration sites outside the change set **only as read-only context** to judge import correctness; do not expand the audit scope.
   - Flag: shared contract in implementation folder; consumer importing contract from implementation folder solely for types; duplicated/divergent contracts; type-only cycles; language-idiom violations (e.g., forcing provider-side interface in Go).
   - **Do not flag**: single-consumer props/local view models (co-location per `architecture-advisor`); Go consumer-side interfaces; nested/private helper types; Ruby RBS signatures.
   - **A type used by ≥2 consumers or forming part of a module's public API is shared**, even if it lives in the same module as its implementation; `lib/`, `util/`, `services/`, and similar folders with runtime code count as implementation folders. Such types must be in a dedicated `types.ts`/`contracts.ts`.
   - Evidence: declaration `folder/file:line` + each consumer `folder/file:line` + import statement.
   - Severity: P1 if cycle/layer violation with build/runtime risk; P2 for layering/maintainability; P3 for style/consistency. Tag findings with `TYPE_OWNERSHIP` or `FOLDER_ARCHITECTURE`.

3. **File Cohesion and Responsibility Separation check**
   - Detect files that mix multiple responsibilities: contracts/types, static data/catalogs, configuration, behavior/formatters, wiring.
   - Patterns: exported contracts in the same file as static data (i18n catalogs, config maps), runtime behavior, or wiring; locale metadata + translations + formatters in one file; large static data inline with logic.
   - i18n-specific: inline translation dictionaries in config/logic files; catalogs must be in dedicated `messages/` or `messages.ts` files separate from config and formatters.
   - Exempt: small single-responsibility files (types-only, config-only, behavior-only, wiring-only, generated files); single-consumer props/local view models per `architecture-advisor`.
   - Severity: P2 for mixing contracts with data/behavior; P3 for config + behavior or minor mixing. Tag findings with `FILE_COHESION`.
   - Evidence: file with responsibility breakdown + reason to change analysis + consumer import paths.

### Phase 6 — Validate Findings Against Context

Before finalizing each finding:
- Read the surrounding implementation and relevant callers.
- Inspect applicable tests and error-handling behavior.
- Verify whether the alleged issue is already fixed or intentional.
- Ensure the finding was introduced by the audited scope.
- **If the declaration site is outside the audited scope (e.g., a shared type declared in a prior commit), do not report the finding; pre-existing issues outside the diff are out of scope.**
- Avoid duplicate findings for the same root cause.

### Phase 7 — Report

Produce the report in this order:

#### 1. Scope & Context

- Project root, classification.
- Audited scope (paths, branch, commit).
- Detected stack + architecture with confidence.
- **Scope Manifest** (files + changed line ranges, exclusions declared).
- **Folder Architecture Map** (layers/modules, responsibility→folder, allowed dependency directions, contract/type location, co-location rules, public module API).
- **Out of scope**: pre-existing issues outside the diff, excluded files, and declaration sites outside the audited change set.

#### 2. Dimension Matrix

| Dimension | Status | N/A Reason (if applicable) |
|---|---|---|
| Architecture & folder structure | PASS/WARN/FAIL/N/A | *e.g., "change does not touch architecture"* |
| ... | ... | ... |

**Note**: Dimension 1 evaluates conformance of the changed code to the Folder Architecture Map; if the change set does not allow assessment of global architecture, report `N/A` with reason.

#### 3. Findings

Ordered by severity (P0 → P3), then confidence.

For each finding:
- `[P0–P3] Short title`
- `File:line` (or narrowest range)
- **Evidence:** REAL / SIMULATED / STATIC + concrete code path, requirement, or command output.
- **Impact:** concrete failure mode or risk.
- **Recommendation:** actionable correction.
- **Confidence:** High / Medium / Low.

#### 4. Verification Gaps

Checks that could not be performed and why (e.g., "Lighthouse not available, CWV findings are hypotheses").

#### 5. Verdict

One of:
- `PASS` — all applicable dimensions PASS, no P0/P1 findings.
- `PASS WITH FINDINGS` — no P0, at most one P1, rest P2/P3.
- `FAIL` — any P0, or multiple P1s.
- `NOT VERIFIED` — critical evidence unavailable.

### Phase 8 — Correction Handoff

- Applying fixes is **not** part of this workflow.
- If the user wants corrections, hand off to a separate authorized Task-to-Code flow (branch policy, alignment, implementation, validation).
- This workflow only produces the audit report.

## Evidence Classes

- **REAL** — executed against the local project (tests ran, linter output, browser trace).
- **SIMULATED** — fixture/mock-based, including simulated approvals.
- **STATIC** — inspection of code, config, docs without execution.

A static check does not prove runtime behavior. A passing build does not prove the whole workflow. When a check cannot be executed, record `BLOCKED` or `NOT_VERIFIED` with the concrete reason.

## Read-only Policy

Never:
- edit or fix source files
- create commits, branches, or PRs
- push or publish anything
- modify the working tree
- install dependencies
- claim a test or check passed without evidence

The human user performs any corrections manually.