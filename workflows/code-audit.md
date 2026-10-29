# Code Audit Workflow

## Purpose

Perform a read-only, evidence-based, senior/staff whole-cycle audit of a generated software solution. It verifies the code is correct, maintainable, appropriately designed (no over-engineering), safe, and evidenced — not merely "it works."

This is **not**:
- `/system-check` (audits the ai-engineering system docs).
- `/validate` (runs formatter, linter, tests, build).
- `/review-pr` (reviews a colleague's PR for copy-ready feedback).

It is an independent, on-demand audit of the generated code itself.

The same procedure serves two inputs. Foreign code is audited against the
dimensions alone. A deliverable **we** produced is audited in **Independent
Mode**, which adds the independence contract, the acceptance judgment, and
claim falsification around the same dimensions. There is one flow, not two.

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
| 2 | **Patterns & complexity** — patterns selected from design forces, justified, composed, stack-idiomatic, no over-engineering/fragmentation (R-42) | `complexity-advisor`, `architecture-advisor` | Always |
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

## Independent Mode — Our Own Deliverable

Use this mode when the deliverable was produced by us or by an agent in our
session and must be judged from outside the context that produced it. The
question it answers is not "is this code good?" but **"did we actually
deliver what was asked, and is it right?"**

It is **not a third scope mode**. The scope is still chosen in Phase 2 (last
commit, `--all`, or explicit paths or a range). Independent Mode adds the
independence contract, the acceptance judgment, and claim falsification
around the same dimensions.

Invoke it through the `external-auditor` agent (`agents/external-auditor.md`):
as the Auditor mode in a **new session**, or as a fresh subagent. Selecting
that mode inside the session that implemented the deliverable does **not**
create independence, because the history persists.

### Independence Contract

All seven rules are mandatory.

| # | Rule | How it is satisfied |
|---|---|---|
| I-1 | **Separate session** | The audit runs in a context that did not produce the deliverable: a fresh subagent or a new session. |
| I-2 | **Cold input** | The auditor receives the requirement and the artifact, never the implementation reasoning. |
| I-3 | **Read-only** | No edits, fixes, commits, branches, pushes, or remote writes. |
| I-4 | **Adversarial stance** | The auditor assumes the deliverable is wrong until evidence proves otherwise. Absence of findings requires executed checks. |
| I-5 | **Re-execution** | Every claim in the delivery report is re-run. A claimed PASS is a hypothesis, never evidence. |
| I-6 | **Frozen scope** | The scope manifest is recorded before inspection and does not change after seeing the code. |
| I-7 | **Declared limit** | Independence is contextual, not epistemic. The same model keeps the same blind spots. State this in the report. |

**Allowed input:** the artifact root and scope; the original task or
requirement wording and its acceptance criteria; the design artifact when the
task had one; the canonical rules (`AGENTS.md`, Skills, references).

**Denied input** (never received, never read when discoverable): the
implementation conversation or session history; the task plan, architecture
proposal, file responsibility map, Folder Architecture Map, or pattern
selection record; the implementer's final report, validation claims, or PR
description; the implementer's explanation of why a choice was made. If the
context already holds that material, stop and request a fresh session.

### Phase I0 — Independence Gate

1. Confirm the session did not produce the deliverable.
2. Build the **Input Manifest**: what was received and what was withheld.
3. If the context contains implementation history, report
   `NOT INDEPENDENT — FRESH SESSION REQUIRED` and stop.
4. Record the client, the session, the model, and the date.

### Phase I1 — Requirement and Acceptance Matrix

Run after Phase 2 and before the dimension audit.

1. Rebuild the requirement from its original wording, without the
   implementation plan. Extract acceptance criteria, constraints, non-goals,
   and explicit decisions.
2. Build the **Acceptance Matrix**: one row per criterion, with the evidence
   that would prove it and how it will be observed.
3. Mark a criterion `AMBIGUOUS` when the requirement does not settle it, and
   say so instead of inventing an interpretation.
4. Never derive criteria from the implementation. That makes the deliverable
   judge itself.

### Phase I2 — Deterministic Re-execution

Run the project's own gates before any semantic judgment, following
`workflows/project-validation.md` (Phases 2, 5, and 7) and the structural
checks of `workflows/output-compliance.md` (`FUNC-001`, `TYPE-001` to
`TYPE-003`, `FILE-001`, `STRUCT-001`, `STRUCT-006`, `STRUCT-007`) in the form
the detected ecosystem supports. Record for every check the command, the exit
code, and the relevant output. A check that cannot run is `NOT VERIFIED` with
the missing tool named. **A skipped check is never a pass.**

### Phase I3 — Claim Falsification

For every claim in the delivery report, produce one row:

| Claim | How it was re-checked | Result |
|---|---|---|
| "tests pass" | command and exit code | `CONFIRMED` / `FALSIFIED` / `NOT VERIFIED` |
| "lint clean" | command and exit code | … |
| "requirement met" | acceptance matrix | … |

A `FALSIFIED` claim is a P1 finding regardless of the code quality: the
delivery was misrepresented.

### Independent Mode Verdict

The Phase 7 verdict applies, with two additions:

- Any acceptance criterion `FAIL` makes the verdict `FAIL`.
- A criterion left silently `NOT VERIFIED` makes the verdict
  `NOT VERIFIED`. `PASS` requires a passed Phase I0 and every criterion
  `PASS`.

The report also states the independence declaration (client, session, model,
date, Input Manifest) and the I-7 limit.

## Procedure

### Phase 1 — Resolve and Classify

- Determine the project root (prefer `AI_ENGINEERING_HOME`, then repo root, then `$HOME/Documents/ai-engineering`).
- Classify the repository: software project with application source code, or documentation-only knowledge repository.
- If documentation-only, report `NOT VERIFIED` for code dimensions and exit.

### Phase 2 — Identify Scope

There are exactly two modes. Do not add a third.

1. **Default — the last commit.** Audit the change introduced by `HEAD`:
   - `git rev-parse --verify HEAD`; if it fails (no commits), report
     `NOTHING TO AUDIT` and exit.
   - `git show --no-patch --format='%H %P%n%s'` to record the commit, its
     parents, and its subject.
   - `git diff --name-status -M HEAD^ HEAD` and
     `git diff --unified=0 HEAD^ HEAD` for a normal commit; for a root commit
     use `git show --name-status --format= --root HEAD` and
     `git show --format= --unified=0 --root HEAD`; for a merge commit diff
     against the first parent (`HEAD^1`).
   - Record the commit SHA and the changed line ranges.
2. **`--all` — the whole project.** Audit every application source file in the
   current working tree, including uncommitted changes.

- **The audited scope is never expanded.** Declaration sites outside it are
  read-only context: read them with `git show <sha>:<path>` in commit mode, or
  from the working tree in `--all`; they are not audited and their issues are
  out of scope.
- **Never check out a commit or modify the working tree.**
- Build a **Scope Manifest** recording the mode (`last commit` or `project`),
  the `--name-status` and `--unified=0` output in commit mode or the file list
  in `--all`, and the exclusions.
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
    - Verify allowed import directions; flag layer-skipping imports (e.g., UI → service implementation for types), mixed-responsibility folders, cross-feature private imports, global `types.ts`/`utils.ts` dumping grounds, circular layer dependencies. **Evidence required:** run `LAYER-001` in `workflows/output-compliance.md` over the audited scope and report the matched `file:line`.
    - **Dumping ground check**: flag `lib/`, `utils/`, `services/` folders that contain domain-specific code (API clients, contracts, storage, i18n catalogs) mixed with generic utilities. These are anti-patterns, not conventions to preserve.
    - **Feature-driven structure check**: in a feature-driven architecture, verify the tree has `app/`, `shared/`, and `features/<feature>/`; that the feature root holds only its responsibility folders and its public API, with no source file at the root; that `components/`, `hooks/`, `services/`, `types/`, and `context/` (or the ecosystem equivalent in `references/folder-architectures.md`) each own their artifacts; that a contract never sits at the feature root or directly inside `components/`, `hooks/`, or `services/`; that each unit with its own test, styles, or types has its own folder inside its responsibility folder; that every feature exposes a public API (`index.ts` or the ecosystem equivalent); and that **no import reaches past another feature's public API** into its internals. A feature imports from `app/` and `shared/` only. Tag `FOLDER_ARCHITECTURE`; severity P2.
    - **Primary unit per file check**: where the detected ecosystem supports it, flag files that export multiple independent components, classes, modules, or other units with their own behavior. **Rule: one interface/type per file in its owning layer's folder.** Exception: multiple types with the **same exclusive consumer** are grouped in one file. Use the applicability matrix in `skills/implementation-recipes/SKILL.md`; never impose React/Angular conventions on Solid, Kotlin, Python, Go, Rust, C#/.NET, or Ruby, which do not support a one-unit-per-file rule. Small private non-exported helpers are exempt. Tag `FILE_COHESION`; severity P2 for independent exported units, P3 for minor related helpers.
    - **Unit folder and test co-location check**: for units with multiple related artifacts (implementation, test, styles, stories, subcomponents), verify they live in a dedicated unit folder (or the ecosystem's equivalent package/module) with the test next to the unit, where the ecosystem supports it. Flag `Component.tsx` + `Component.test.tsx` + `Component/` as siblings, a test at the root of the source tree or of a layer folder, a test detached from the unit against the ecosystem convention, or a support location that holds test cases. Tag `FOLDER_ARCHITECTURE`; severity P2 when the unit has multiple artifacts, P2 for a test at the source root or a support location with test cases, P3 for a single trivial unit.
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

**Dimension 2 — Patterns & complexity: explicit checks**

1. **Selection record check**
    - Compare the implemented patterns against the recorded selection from `references/design-patterns.md`.
    - A change that introduces or changes a pattern without a selection record is a finding: `PATTERN_UNJUSTIFIED`.
    - A pattern in the code with no recorded signal and force is a finding: `PATTERN_UNJUSTIFIED`.
    - A recorded pattern with no real consumer is a finding: `PATTERN_UNUSED`.
2. **Signal check**
    - Every selected pattern must cite at least one signal from the catalog's taxonomy.
    - A pattern justified only by its name, by “best practice”, or by a framework fashion is not evidence.
    - A count that matches a preferred number rather than the set of **design forces** is a finding.
3. **Existing mechanism check**
    - Verify that an existing abstraction, framework capability, or language construct was not duplicated by a new pattern.
    - A new pattern that repeats an existing mechanism is `PATTERN_DUPLICATE`.
4. **Composition check**
    - Verify that patterns which resolve the same force are not both present unless they are a recognized composition.
    - Verify that patterns which must compose (Strategy + Factory, Composite + Visitor, Command + Memento, Repository + Unit of Work, Outbox + Idempotent Receiver) are wired correctly.
5. **Stack translation check**
    - Verify that the implementation uses the detected stack's idiom, not a class-shaped GoF diagram forced onto functions, protocols, traits, or modules.
    - Verify that the idiom is consistent with `references/design-patterns.md`, `skills/implementation-recipes/SKILL.md`, and the stack's official documentation.
6. **Count check**
    - Recompute the count from the selected set. It may be 0, 2, 32, or 100+; it is never capped and never padded.
    - A count that cannot be reconstructed from signals and forces is a finding.
7. **Scope check**
    - Audit only the change set. A pre-existing pattern outside the diff is context, not a finding (R-42).
    - A pattern introduced or changed by the task is in scope.

**Dimension 3 — Code quality and separation: explicit checks**

1. **SOLID check** — apply the five criteria in
   `skills/engineering-review/SKILL.md` ("SOLID"): single responsibility,
   open/closed, Liskov substitution, interface segregation, and dependency
   inversion. Report with the tags `SOLID_SRP`, `SOLID_OCP`, `SOLID_LSP`,
   `SOLID_ISP`, and `SOLID_DIP`, severity P2, `file:line` evidence, and the
   concrete reason to change or the dependency edge. Never report a principle
   without that evidence.
2. **Cohesion and coupling check** — a unit with unrelated responsibilities,
   a folder that changes for unrelated reasons, or a module that reaches into
   another module's internals is `FILE_COHESION` or `FOLDER_ARCHITECTURE`.
3. **Boundary validation and error model** — untrusted input validated at the
   edge, domain errors distinct from transport errors, no leaked internals,
   no swallowed failures. A violation is `BOUNDARY` or `ERROR_MODEL`, P2.
4. **Language idiom check** — the design uses the detected stack's idiom
   (Go consumer interfaces, Python protocols, TS structural contracts,
   Java/Kotlin/C#/PHP interfaces, Ruby duck typing) instead of a class-shaped
   diagram forced onto the language. A forced idiom is `SOLID_*` or
   `FILE_COHESION`, P3.

### Phase 6 — Validate Findings Against Context

Apply the canonical procedure in `skills/engineering-review/SKILL.md`
("Validate Findings Against Context"). It is not restated here.

### Phase 7 — Report

The report is **advisory**: it proposes improvements. It never applies a change
and never blocks a commit by itself.

Produce the report in this order:

#### 1. Scope & Context

- Project root, classification.
- Audited scope: `last commit <sha>` or `whole project`.
- Detected stack + architecture with confidence.
- **Scope Manifest** (mode, files + changed line ranges or project file list,
  exclusions declared).
- **Folder Architecture Map** (layers/modules, responsibility→folder, allowed dependency directions, contract/type location, co-location rules, public module API).
- **Out of scope**: files and declaration sites outside the audited scope,
  excluded files, and issues pre-existing outside it.
- **Independent Mode only**: the independence declaration and Input Manifest
  (Phase I0), the Acceptance Matrix (Phase I1), the deterministic results
  (Phase I2), and the claim falsification table (Phase I3), placed before the
  dimension matrix.

#### 2. Dimension Matrix

| Dimension | Status | N/A Reason (if applicable) |
|---|---|---|
| Architecture & folder structure | PASS/WARN/FAIL/N/A | *e.g., "change does not touch architecture"* |
| ... | ... | ... |

**Note**: Dimension 1 evaluates conformance of the audited scope to the Folder Architecture Map; if the scope does not allow assessment of global architecture, report `N/A` with reason.

#### 3. Suggested improvements

Ordered by severity (P0 → P3), then confidence. Each item is a recommendation,
not an applied change.

For each suggestion:
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