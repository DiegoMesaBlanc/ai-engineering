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
    - **Dumping ground check**: flag `lib/`, `utils/`, `services/` folders that contain domain-specific code (API clients, contracts, storage, i18n catalogs) mixed with generic utilities. These are anti-patterns, not conventions to preserve.
    - **Primary unit per file check**: where the detected ecosystem supports it, flag files that export multiple independent components, classes, modules, or other units with their own behavior. **Rule: one interface/type per file in its owning layer's folder.** Exception: multiple types with the **same exclusive consumer** are grouped in one file. Use the applicability matrix in `skills/implementation-recipes/SKILL.md`; never impose React/Angular conventions on Solid, Kotlin, Python, Go, Rust, C#/.NET, or Ruby, which do not support a one-unit-per-file rule. Small private non-exported helpers are exempt. Tag `FILE_COHESION`; severity P2 for independent exported units, P3 for minor related helpers.
    - **Unit folder and test co-location check**: for units with multiple related artifacts (implementation, test, styles, stories, subcomponents), verify they live in a dedicated unit folder (or the ecosystem's equivalent package/module) with the test next to the unit, where the ecosystem supports it. Flag `Component.tsx` + `Component.test.tsx` + `Component/` as siblings, or a test detached from the unit against the ecosystem convention. Tag `FOLDER_ARCHITECTURE`; severity P2 when the unit has multiple artifacts, P3 for a single trivial unit.
    - **Barrel check**: a barrel (`index.ts`) is optional. Flag a barrel that only re-exports one file or lacks a real public-API role as P3 `FOLDER_ARCHITECTURE`. Do not require barrels by default or invent equivalents in ecosystems that do not use them.
    - Do not impose a universal template; audit conformance to the detected/declared structure. If no structure exists, report "no identifiable architecture" and evaluate whether new code created a coherent one per the complexity-based decision (small→layer-first, medium→feature-first).
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
    - Flag: shared contract in implementation folder; consumer importing contract from implementation folder solely for types; duplicated/divergent contracts; type-only cycles; language-idiom violations (e.g., forcing provider-side interface in Go). **Pass-through re-export**: flag a module that re-exports a contract it only consumes — a component re-exporting the props it imports from `./types`, a service re-exporting the contracts it imports from `./contracts`. The owning file is the single public path. A barrel that aggregates several modules is exempt. Severity: P2. Tag `TYPE_OWNERSHIP`. **Evidence required:** run the `TYPE-001`, `TYPE-002`, and `TYPE-003` detection in `workflows/output-compliance.md` over the audited scope and report the matched `file:line`.
    - **Do not flag**: Go consumer-side interfaces; nested/private helper types; Ruby RBS signatures.
    - **A type is shared when it is imported/referenced from another file**. It must then live in a dedicated `types.ts`/`contracts.ts`/`schemas.ts`, not in a file that also exports runtime behavior. Mere export without external reuse is WARN/observation, not a defect.
    - **Schema + inferred type pair**: Zod/Joi/Yup schemas and their `infer` types must stay together in `schemas.ts`/`contracts.ts`; validators, converters, and HTTP clients must be in a separate file.
    - Evidence: declaration `folder/file:line` + each consumer `folder/file:line` + import statement.
    - Severity: P1 if cycle/layer violation with build/runtime risk; P2 for layering/maintainability (shared type in implementation file); P3 for style/consistency (exported but not reused). Tag findings with `TYPE_OWNERSHIP` or `FOLDER_ARCHITECTURE`.

3. **File Cohesion and Responsibility Separation check**
    - Detect files that mix multiple responsibilities: contracts/types, static data/catalogs, configuration, behavior/formatters, wiring.
    - Patterns: exported contracts in the same file as static data (i18n catalogs, config maps), runtime behavior, or wiring; locale metadata + translations + formatters in one file; large static data inline with logic; schemas + inferred types + validators + converters + HTTP client in one file.
    - Multi-unit files: several exported components, classes, or modules with independent state, behavior, or lifecycle in one file, where the detected ecosystem supports a primary-unit-per-file rule. Do not flag multiple related declarations where the ecosystem explicitly allows them (e.g., Solid, Kotlin).
    - i18n-specific: inline translation dictionaries in config/logic files; catalogs must be in dedicated `messages/` or `messages.ts` files separate from config and formatters.
    - API-specific: Zod schemas + inferred types + validation functions + converters + fetch client in one file → must separate `contracts.ts` (schemas + types) from `client.ts` (validators, converters, fetch).
    - **TypeScript/JavaScript function style**: **prefer arrow functions** (`const fn = () => {}`) in `.ts`, `.tsx`, `.js`, `.jsx` files. Flag `function` keyword when an arrow function would work equally well. Allow `function` when there is a legitimate reason: generators (`function*`), recursive functions needing self-reference by name, functions requiring `this` binding that cannot use arrow-class-field syntax, use of `arguments` object, or `new.target`. Severity: P3. Tag: `TS_FUNCTION_STYLE`. **Evidence required:** run the `FUNC-001` detection in `workflows/output-compliance.md` over the audited scope and report the matched `file:line`.
    - Exempt: small single-responsibility files (types-only, config-only, behavior-only, wiring-only, generated files).
    - Severity: P2 for mixing contracts with data/behavior, schema+validators+client, or multiple independent exported units where the ecosystem supports one unit per file; P3 for config + behavior or minor mixing. Tag findings with `FILE_COHESION`.
    - Evidence: file with responsibility breakdown + reason to change analysis + consumer import paths.

### Phase 6 — Validate Findings Against Context

Apply the canonical procedure in `skills/engineering-review/SKILL.md`
("Validate Findings Against Context"). It is not restated here.

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