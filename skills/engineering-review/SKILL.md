---
name: engineering-review
description: Review implemented changes for correctness, architecture, maintainability, typing, error handling, testing, security, performance, and unnecessary complexity.
---

# Engineering Review

## Review scope

Review the actual changes, not the entire repository unless required.

## Check

### Correctness

- Does the implementation satisfy the requirement?
- Are edge cases handled?
- Are errors handled correctly?

### Architecture

- Does the change respect the existing architecture and conventions?
- Were unnecessary abstractions introduced?
- Were existing abstractions reused?
- **Pattern selection**: verify the change against the recorded selection in
  `references/design-patterns.md`. Every pattern must have a signal and a
  **design forces** entry; no existing mechanism may be duplicated;
  compositions must be coherent; the count must be derived; the
  implementation must be idiomatic for the stack. A pattern without a signal
  is a finding.
- Perform the post-implementation architectural validation defined in
  `skills/architecture-advisor/SKILL.md`: contrast the result with the
  proposed design and the project's conventions, including placement,
  cohesion, reuse, module conventions, and unnecessary fragmentation.
- **Folder architecture & contract ownership**: verify conformance with the
  Folder Architecture Map (layers/modules, responsibility→folder, allowed
  dependency directions, contract/type location per module). Flag layer-skipping
  imports, contracts in implementation folders, mixed-responsibility folders,
  and global `types.ts`/`utils.ts` dumping grounds.
- **Primary unit per file**: where the detected ecosystem supports it, flag
  files that export several independent components, classes, or other units
  with their own behavior. Use the applicability matrix in
  `skills/implementation-recipes/SKILL.md`; do not impose a rule the
  ecosystem does not support. Small private non-exported helpers are exempt.
- **Unit folders and co-located tests**: when a unit has multiple related
  artifacts (implementation, test, styles, stories, subcomponents), its
  implementation and test should live in the unit folder where the
  ecosystem supports it; flag `X`, `X.test`, and `X/` siblings, or a test
  detached from the unit against the ecosystem convention. Do not apply the
  folder convention to ecosystems that organize tests differently.
- **Barrel check**: a barrel (`index.ts`) is optional. Flag a barrel that
  only re-exports one file or has no real public-API role as a P3
  observation; do not require barrels by default.
- **Dumping ground check**: flag `lib/`, `utils/`, `services/` folders that
  contain domain-specific code (API clients, contracts, storage, i18n catalogs)
  mixed with generic utilities. These are anti-patterns; new code must not
  extend them.
- **Type/contract ownership and import direction**: types live in the folder of
  the owning layer; consumers import from the contract, not from an
  implementation module; language idiom respected (Go consumer-side interfaces,
  Java one-public-type-per-file, Python Protocols, etc.).
- **Reuse-based type extraction**: a type is extracted to `types.ts`/
  `contracts.ts`/`schemas.ts` only when it is **actually imported from another
  file**. Mere export without external reuse is a WARN/observation, not a
  defect.
- **Schema + inferred type pair**: Zod/Joi/Yup schemas and their `infer` types
  must stay together in `schemas.ts`/`contracts.ts`; validators, converters,
  and HTTP clients must be in a separate file.
- **File cohesion and responsibility separation**: each changed file should
  have one primary responsibility. **Rule: one interface/type per file in its owning layer's folder.** Exception: multiple types with the **same exclusive consumer** are grouped in one file. Flag files mixing contracts/types with
  static data/catalogs, configuration, behavior, or wiring. i18n modules
  must separate contracts, catalogs (`messages/<locale>.ts`), config, and
  formatters. API modules must separate schemas + types (`contracts.ts`) from
  validators, converters, and clients (`client.ts`).
- **Pass-through re-export**: flag a module that re-exports a contract it
  only consumes — a component re-exporting the props it imports from
  `./types`, a service re-exporting the contracts it imports from
  `./contracts`. The owning file is the single public path; a barrel that
  aggregates several modules is exempt.
- **TypeScript/JavaScript function style**: prefer arrow functions; flag the `function` keyword when an arrow function would work equally well, in `.ts`, `.tsx`, `.js`, and `.jsx` files. Allow `function` for a generator, a `this` binding that arrow syntax cannot provide, the `arguments` object, or `new.target`, with the reason stated in a comment. Severity: P3. Tag: `TS_FUNCTION_STYLE`.
- **Unnecessary abstraction or fragmentation is a finding that must be
  resolved: revert the change or justify it with evidence against the
  thresholds in `skills/complexity-advisor/SKILL.md` (Refactoring and
  Abstraction Justification). A conditional PASS without resolution is not
  acceptable.**

### Code quality

- readability
- naming
- cohesion
- coupling
- duplication
- typing
- maintainability

### Testing

- appropriate unit tests
- integration tests where required
- E2E tests where required
- regression coverage

### Security

Look for relevant vulnerabilities based on the technology stack.

### Performance

Consider performance only when materially relevant.

For web-facing changes, use `skills/performance/SKILL.md` and
`skills/core-web-vitals/SKILL.md`, and `skills/accessibility/SKILL.md`,
`skills/seo/SKILL.md`, and `skills/best-practices/SKILL.md` for
accessibility, search, and web security. Keep field, lab, and hypothesis
evidence distinct, and never present a lab or Lighthouse score as WCAG
conformance or as a ranking prediction.

### Scope

Identify unrelated changes.

## Validate Findings Against Context

This is the canonical procedure for validating a finding before reporting
it. `workflows/code-audit.md` (Phase 6), `workflows/pull-request-review.md`
(Phase 10), and `workflows/task-to-code.md` (Phase 12) all use it; they do
not restate it.

Before finalizing a finding:

1. Read the surrounding implementation and relevant callers.
2. Inspect the applicable acceptance criteria, task requirements, and design
   references.
3. Check existing tests and relevant error-handling behavior.
4. Verify whether the alleged defect is already fixed elsewhere, or is
   intentional and consistent with the project.
5. Ensure the finding was introduced by the reviewed change. A declaration
   site outside the audited scope, such as a shared type declared in a prior
   commit, is out of scope.
6. Avoid duplicating findings that describe the same root cause.

If the evidence is insufficient, omit the finding or place the concern in a
separately marked section requiring human verification. Do not invent line
numbers, test results, runtime behavior, requirements, or design
specifications, and verify every line number against the reviewed revision.

## Output

Classify findings:

P0 Critical
P1 Important
P2 Recommended
P3 Nice-to-have

Do not modify code during review unless explicitly requested.
