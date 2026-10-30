---
name: implementation-recipes
description: Implementation recipes for the supported stacks — arrow functions and function style, type and contract placement, file cohesion, unit folders, test co-location, error handling, and boundary validation. Load when writing, changing, or reviewing code in any supported stack.
---

# Implementation Recipes

## Purpose

Explain *how* to implement, not only how to describe architecture, for the
supported ecosystems. Architecture decisions belong to
`architecture-advisor`; this Skill turns an agreed design into concrete,
maintainable implementation guidance.

## Non-Negotiable Quality Baseline

Quality is not proportional to task size. A one-line change and a large
feature are both held to professional standards:

- the **structural rules** of `AGENTS.md`: arrow functions, every contract
  in a **dedicated file**, no pass-through re-export, and their precedence
  over a detected project convention. They are the canonical wording and are
  not restated here
- keep functions and modules cohesive and single-purpose
- name things clearly and consistently
- handle errors explicitly; never swallow them silently
- validate input at boundaries
- never log or commit secrets
- write or update tests for changed behavior
- keep type safety where the language supports it
- leave no debug code, dead code, or temporary hacks behind

Proportionality applies to planning rigor and test depth, never to the
quality of the delivered code. Do not implement a task in a junior way
because it looks simple.

## General Procedure

1. Detect the stack, versions, and existing conventions (see
   `stack-detection`, `project-analysis`).
2. **Consult `references/official-documentation.md`** for the detected stack to retrieve authoritative guidance on framework conventions, APIs, and best practices.
3. Reuse existing abstractions, utilities, and patterns (see
   `architecture-advisor`).
4. Identify the cross-cutting concerns the change touches (validation,
   errors, auth, config, logging, testing).
5. Implement in the smallest cohesive unit that satisfies the design.
6. Validate with the project's formatter, linter, type checker, and tests.
7. Review the result against the agreed design and conventions.

**Official documentation consultation:** Step 2 is mandatory after stack detection. Consult only the sections relevant to the detected stack and current task. Do not load entire documentation sites.

**Structural rules are verified, not installed in the target project.** A
target project receives only its standard `eslint.config.*` and Prettier
configuration, with the core rules `func-style` and `no-restricted-syntax`
expressing the arrow function convention and
`@typescript-eslint/consistent-type-imports` covering type-only imports when
`typescript-eslint` is already part of the setup. The structural policy —
types separated from runtime behavior, contract ownership and grouping,
pass-through re-exports, and unit folders — is verified inside this system by
`workflows/output-compliance.md` over the change set and applied during
review. No custom rule file, plugin, or checker script is created in the target
project, and no dependency is added for it. The canonical configuration
guidance is `references/linter-rules.md`.

## Folder Architecture and Contract Placement

Follow the **Folder Architecture Map** from `architecture-advisor`. Place contracts/types in the folder of the owning layer. In a feature-driven tree, feature-level contracts go in `types/` (or the ecosystem equivalent), and a unit-local contract goes in the unit's folder as `<unit>.types.ts`; a contract never sits at the feature root or directly inside `components/`, `hooks/`, or `services/`. **Rule: one interface/type per file in its owning layer's folder.** Exception: when multiple interfaces/types are **exclusively consumed by the same component/service/logical unit**, they are grouped in a single file (e.g., `ComponentTypes.ts`, `ServiceTypes.ts`, or co-located with the unit). Do not create separate files for types that always travel together. Respect language idiom.

**Complexity-based structure decision** (when no coherent architecture exists):

| Complexity | Recommended structure |
|---|---|
| Small, single domain | Layer-first: `src/api/<domain>/`, `src/services/<domain>/`, `src/server/<domain>/`, `src/shared/`, `src/components/`, `src/hooks/` |
| Medium, multiple domains | Feature-first: `src/app/`, `src/shared/`, and `src/features/<feature>/` with responsibility folders (`components/`, `hooks/`, `services/`, `types/`, `context/`), one folder per unit inside its responsibility folder, and an `index.ts` public API for the feature (or the ecosystem equivalent). See **Feature-Driven Folder Architecture** in `skills/architecture-advisor/SKILL.md` and the per-ecosystem trees in `references/folder-architectures.md` |
| Large, multiple teams | Feature modules with explicit public APIs and boundaries; packages only when justified |

| Ecosystem | Contract/type location (rule: one per file; group if same exclusive consumer) | Import direction |
|---|---|---|
| TS/JS (React/Next.js) | Feature `types/<domain>.types.ts` or `src/shared/types/<domain>.ts` (one type per export; group same-consumer types) | UI → feature/shared contract; never UI → `services` for types |
| Java/Kotlin (Spring) | `domain/` or `application/` package (one public type per file; group same-consumer types) | `api → application → domain`; no UI → infra |
| Python (FastAPI/Flask/Django) | `domain/contracts.py` or `domain/types.py` (one class per module preferred; group same-consumer types) | router → application/domain; not → infra |
| C#/.NET | `Domain/` or `Application/` (one public type per file; group same-consumer types) | `Api → Application → Domain` |
| Go | structs in `internal/domain`; **interfaces in consumer package** | no cycles; no provider-side interface |
| Rust | traits near type or `contracts/`; DTOs in common crate | respect orphan rules |
| PHP | `Domain/` or `Contracts/` (one class per file, PSR-4; group same-consumer types) | PSR-4, one type per file |
| Ruby | Sorbet/RBS in `sig/` or `rbi/` | no invented interfaces |

Do not impose a universal template; preserve the project's detected conventions **when coherent**. A mixed `lib/` is not a convention to preserve.

### Unit style: functions or classes

Clean Architecture and Hexagonal are a **dependency rule**, not a syntax
rule. The unit is a class only when the ecosystem or the DI container makes
it the idiom. Do not mix the two styles inside one layer.

| Ecosystem | Default unit | Use a class when |
|---|---|---|
| TypeScript / JavaScript (Node, Express, Fastify) | function or factory + `interface` port | the unit holds state or lifecycle, the project already uses classes, or a DI container instantiates it |
| NestJS, Angular | class | always; the framework instantiates and injects it |
| Java, Kotlin, C#, PHP | class + interface | always; it is the language idiom |
| Go | struct + interface at the consumer | a struct carries state; the language has no class |
| Rust | struct + trait | a struct carries state; the language has no class |
| Python | function or class + `Protocol`/ABC | state, framework, or project convention |
| Ruby | class or module | state or framework; duck typing otherwise |

Rules:

- A **port** is an `interface` (TypeScript, Java, Kotlin, C#, PHP), a
  `Protocol` or ABC (Python), a trait (Rust), or an interface declared by the
  consumer (Go). It never leaks the framework.
- An **adapter** is a factory function or a class; both satisfy the port.
- The **composition root** is the only place that constructs concrete
  adapters.
- Arrow functions remain the default for standalone functions; class methods
  are exempt from the arrow rule.
- A project that adopts classes does so for the whole layer, not file by
  file: two styles in one layer is a finding.

### Unit Organization and Test Co-location

When a unit has more than one related artifact — implementation, test,
styles, stories, fixtures, or subcomponents — use a folder per unit and put
the test next to the unit where the ecosystem supports it. Do not leave
`X.tsx`, `X.test.tsx`, and `X/` as siblings.

**Primary unit per file — canonical applicability matrix:**

| Ecosystem | Primary unit per file | Test location | Authority / applicability |
|---|---|---|---|
| React / Next.js | One React component per file when the project accepts Airbnb + ESLint `react/no-multi-comp`; multiple stateless/pure components allowed. React documentation and Next.js do not mandate this. | Vitest/Jest next to the unit or project convention | Airbnb, ESLint; Next.js is unopinionated |
| TypeScript / JavaScript backend (Node.js, Express, NestJS, Fastify) | One unit per file is common but not mandated | `foo.test.ts` next to `foo.ts`; API-surface tests in `tests/integration/` | Vitest guide (co-located `sum.test.js` example), project convention |
| Angular | One concept per file; official exception for small related components/directives | `*.spec.ts` in the same directory | Angular Style Guide |
| Vue | One component per `.vue`; at most one `<template>`/`<script>` | Vitest next to the unit | Vue SFC specification |
| Svelte / SvelteKit | A `.svelte` file is one component | Vitest or project convention | Svelte documentation |
| Solid | Multiple components per file are explicitly allowed; do not enforce one per file | Vitest or project convention | Solid documentation |
| React Native | One component per file is the convention; related pure helpers may share | Jest/Vitest next to the unit | React Native documentation; Jest preset |
| Ionic (Angular) | Follows Angular's "one concept per file" | `*.spec.ts` in the same directory | Angular Style Guide; Ionic adds no layout rule |
| Ionic (React) | Follows React's file convention | Vitest/Jest next to the unit | React conventions; Ionic adds no layout rule |
| Java | At most one public top-level class/interface referenced by other compilation units per file; one public type per file is the common convention | Mirrored `src/test/java` package | Java Language Specification §7.6 |
| Kotlin | Multiple related declarations per file are allowed and encouraged; do not enforce one per file | Mirrored test source set | Kotlin coding conventions |
| Python | No one-class-per-file rule | `test_*.py` co-located or in a tests tree | PEP 8, pytest |
| Go | No one-type-per-file rule; package per directory | `_test.go` in the same package | Go documentation |
| Rust | No one-type-per-file rule | `#[cfg(test)]` module or `tests/` | Rust Book, Cargo |
| C#/.NET | No one-class-per-file mandate | Mirrored test project | .NET documentation |
| PHP | One autoloaded class per file (PSR-4) | Mirrored tests directory | PSR-4, PHPUnit |
| Ruby | No one-class-per-file mandate | `spec/` mirroring `lib/` | RSpec |

**Layout:** component folders for front-end units; Angular's conventional
`name.component.ts`/`.html`/`.css`/`.spec.ts` set; mirrored test source sets
for Java/Kotlin; `test_*.py` for Python; `_test.go` for Go; `tests/` for
Rust; mirrored test projects for C#/.NET; and mirrored test directories for
PHP/Ruby.

**Two rules apply in every ecosystem:**

1. A test case lives next to the unit it tests (co-locating ecosystems) or in
   the mirrored test tree (mirroring ecosystems). It is never placed at the
   root of the source tree or of a layer folder: when no single unit owns it,
   it belongs to an explicit integration tree (`tests/integration/`) or next
   to the module it exercises.
2. Shared test support — setup, fixtures, factories, helpers, test servers —
   lives in an explicitly named support location: `testing/` or
   `test/support/` in TypeScript and JavaScript, `spec/support/` in Ruby,
   `conftest.py` in Python, `src/testFixtures` in JVM projects, the test
   project in .NET, and `tests/common/` in Rust. A support location never
   contains test cases. In TypeScript and JavaScript a `test/` or `tests/`
   folder directly under `src/` is not a support location.

**Barrels:** `index.ts` is optional and is not part of the default layout.
Add one only for multiple public exports or consumers, a stable module
boundary, a substantially populated folder, or an existing project
convention. Never add one solely to re-export a single file. Barrels are a
JS/TS module convention; do not invent equivalents in other ecosystems.

Consult `references/official-documentation.md` ("Project Structure and Test
Co-location") and the project's coherent conventions before choosing the
layout. A single trivial file with no test, styles, or subcomponents may
remain a single file. Do not impose a rule the detected ecosystem does not
support.

## Cross-Cutting Concerns

Apply these consistently regardless of stack:

- **Configuration**: read from the project's existing config and
  environment mechanism; no hardcoded environment values or secrets.
- **Validation**: validate at trust boundaries; reject invalid input early.
- **Errors**: use the project's error model; return meaningful, safe
  errors; do not leak internals.
- **Logging and observability**: use the project's logger; structured where
  available; never log secrets or personal data.
- **Security**: apply least privilege, input handling, and auth through the
  existing mechanisms.
- **Testing**: test changed behavior at the level that protects it.

## Internal Module Layout (File-Level)

A module (feature, shared lib, domain) should split its internal
responsibilities across files, not bundle them in one file. Use this
pattern as the default; adapt to the project's existing conventions.

| Responsibility | File(s) | Notes |
|---|---|---|
| Contracts / Types (public API) | `types/` / `contracts/` / `domain/` (one file per type, or one file per consumer group) | Shared interfaces, types, enums, DTOs. |
| Static Data / Catalogs | `messages/`, `catalogs/`, `data/` | Message catalogs, locale data, lookup tables. Per-locale files enable lazy loading. |
| Configuration | `config.ts` / `constants.ts` / `settings.ts` | Locale metadata, feature flags, constants. |
| Behavior / Formatters | `format.ts` / `utils.ts` / `helpers.ts` / `services/` | Pure functions, formatters, accessors, business logic. |
| Public API (barrel, optional) | `index.ts` | Re-exports only; use only when multiple entry points or consumers justify it, never to re-export a single file. |

**i18n-specific layout** (per locale files recommended for lazy loading):

```text
src/shared/i18n/
  types.ts           # Locale, LocaleMeta, PaymentSelectorTranslations
  config.ts          # SUPPORTED_LOCALES, LOCALE_META, getLocaleMeta, ...
  messages/
    en.ts
    es.ts
    ar.ts
  format.ts          # formatExpiry, formatCurrency, formatNumber, ...
  index.ts           # Optional public API: getTranslations, formatters, types
```

If per-locale lazy loading is not needed, a single `messages.ts` is
acceptable; catalogs must still be separate from config and formatters.

Do not mix contracts, catalogs, config, and formatters in one file
(e.g., a monolithic `i18n.ts`). Split when the file would change for
unrelated reasons or a consumer cannot import the contract without
pulling in data/runtime.

**API module recipe** (contracts + client + service + storage separation):

```text
src/api/<domain>/           # or src/features/<feature>/api/
  contracts.ts              # Zod schemas + inferred types (shared)
  client.ts                 # HTTP calls, validation, converters
src/services/<domain>/      # or src/features/<feature>/domain/
  service.ts                # Business logic, domain rules
  types.ts                  # Domain types if not in contracts
src/server/<domain>/        # or src/features/<feature>/server/
  storage.ts                # Persistence, DB access (server-only)
```

- Route handlers (`app/api/...`, `routes/...`) are thin; they import from `client`/`service`, not from implementation files directly.
- UI imports contracts from `contracts.ts`, never from `service.ts` or `storage.ts`.
- Schema + inferred type pair stays in `contracts.ts`; validators/converters in `client.ts`.
- Shared constants (`SUPPORTED_*`) go in `config.ts`/`constants.ts`, not in service or contract files.

## Frontend Recipes

### React and Next.js

- Match the detected router (App Router or Pages Router) and rendering
  model (server/client components).
- Follow the project's **Folder Architecture Map**: a feature holds `components/`, `hooks/`, `services/`, `types/`, and `context/` when it owns local state, plus `index.ts` or the ecosystem equivalent; shared code lives in `src/shared/{components,hooks,lib,types}`.
- **Contracts**: feature-level types in `types/<domain>.types.ts` or `src/shared/types/<domain>.ts`; props/view models co-located in the unit folder, never at the feature root.
- **Imports**: UI imports contracts from feature/shared types, never from `services` or `lib` implementation modules.
- Keep server and client boundaries explicit; do not leak server-only code to the client.
- Manage state with the project's existing solution; avoid introducing a new one without justification.
- Handle loading, empty, and error states in data access.
- Follow the framework's data-fetching and caching conventions.
- **Function style**: use **arrow functions** (`const fn = () => {}`) for components, hooks, utilities, and callbacks. The `function` keyword is reserved for the legitimate reasons listed in `AGENTS.md` (a generator, a `this` binding that arrow syntax cannot provide, the `arguments` object, or `new.target`). This applies to `.ts`, `.tsx`, `.js`, and `.jsx`, and it governs the functions the task **creates**: an existing declaration is never converted as a side effect, editing its body leaves the declaration unchanged, and converting it is a task of its own. A lint violation that pre-existed the change set is a finding, not a blocker.
- **Component style**: Prefer `const Component = ({ props }) => {}` over `function Component({ props }) {}`.
- **Hook style**: `const useHook = () => {}` not `function useHook() {}`.
- **Callback style**: `const handleClick = () => {}` not `function handleClick() {}`.

### Angular

- Follow the standalone-component or NgModule organization actually used.
- **Contracts**: shared types in feature `types/` or `models/`; component inputs/outputs co-located in the unit folder. Use a barrel only when justified.
- **Imports**: components import from feature/shared types, not from service implementation files.
- Place HTTP calls and data access in `data-access/`, and UI in `components/`.
- Use the framework's dependency injection and services; keep components thin.
- Respect RxJS conventions and unsubscribe/cleanup patterns.
- Use the detected state, routing, and form libraries consistently.

### Vue / Nuxt

- Use the detected composition/options style and component conventions.
- **Contracts**: shared types in feature `types/`; props co-located in the unit folder.
- **Imports**: components import from feature/shared types, not from service files.
- Place composables in `composables/`, stores in `stores/`, HTTP calls in `services/`, and UI in `components/`.
- Respect Nuxt server/client boundaries when present.

### Svelte / SvelteKit

- One component per `.svelte` file; a component folder holds the component, its
  test, styles, and stories.
- SvelteKit routes live in `src/routes/`; shared code lives in `src/lib/` with
  the `$lib` alias, not in deep relative paths.
- **Contracts**: shared types in feature `types/`; component props co-located in
  the unit folder; never declare contracts inside `<script>` when another unit
  consumes them.
- Server-only code stays in `+page.server.ts`, `+layout.server.ts`, or
  `+server.ts`, and is never imported by client code.
- Validate at form actions and endpoints, not inside presentational components.

### Solid / SolidStart

- One component per file is the norm; Solid allows several related components in
  one file, so do not enforce a one-component rule as a framework mandate.
- A feature holds `components/`, `hooks/`, `services/`, `types/`, and
  `context/` when it owns local state, plus `index.ts`.
- Prefer signals and stores already used by the project; do not duplicate state
  in a second mechanism.
- **Contracts**: shared types in `types/`; props co-located in the unit folder.
- Keep server functions (`"use server"`) out of client component modules.

## Backend Recipes

### Node.js (Express, Fastify, NestJS)

- Follow the project's **Folder Architecture Map**: `src/routes`, `src/controllers`, `src/services`, `src/domain`, `src/infrastructure`, `src/shared/types`.
- **Contracts**: shared DTOs/contracts in `src/domain` or `src/shared/types`; route/request types co-located with handlers.
- **Imports**: controllers import contracts from domain/shared, not from service implementation.
- Validate inputs at the boundary; centralize error handling.
- Use the detected validation, ORM, and configuration libraries.
- For NestJS, respect modules/providers/controllers; for Express/Fastify, keep handlers thin and logic in services.

### Python (Django, FastAPI, Flask)

- Follow framework conventions: Django apps and models, FastAPI routers and dependencies, Flask blueprints.
- **Contracts**: shared `Protocol`/`TypedDict`/`dataclass` in `domain/contracts.py` or `domain/types.py`; request/response models co-located with routers.
- **Imports**: routers/views import from `domain`, not from service implementation modules.
- Keep business logic out of views and routers.
- Use the project's validation and serialization approach.
- Apply migrations and schema changes through the project's tooling.

### Java (Spring Boot)

- Respect package and module boundaries: `api`, `application`, `domain`, `infrastructure`.
- **Contracts**: interfaces/DTOs in `domain` or `application`; Java public types conventionally one per file (JLS §7.6); private/nested types may stay.
- **Imports**: controllers import from `domain`/`application`, not from `infrastructure` implementation.
- Use dependency injection and configuration conventions.
- Handle exceptions centrally and consistently.

### Kotlin (Spring Boot)

- Keep the same package boundaries as Java: `api`, `application`, `domain`,
  `infrastructure`; multiple closely related declarations per file are
  idiomatic and are not a defect.
- **Contracts**: data classes and DTOs in `domain` or `application`; keep
  framework types out of the domain; make nullability explicit at boundaries.
- **Tests**: mirrored `src/test/kotlin` package, next to the production package.
- Respect the project's coroutine scopes and dispatchers; never launch from
  `GlobalScope`; prefer constructor injection and immutable state.
- Handle errors with the project's exception or result model; do not swallow
  them in `runCatching` without a decision.

### Go

- One package per directory; `internal/` for code that must not be imported
  outside the module; the package's exported identifiers are its public API and
  carry doc comments.
- **Tests**: `_test.go` in the same package directory; table-driven cases and
  `t.Run` subtests; `gofmt`, `go vet`, and `go test ./...` must pass.
- Return errors instead of panicking; wrap with `%w`; pass `context.Context` as
  the first parameter on I/O boundaries; keep package-level mutable state out.
- Define small interfaces at the consumer, not at the implementation; do not
  create an interface for a single implementation without a test seam.

### Rust

- One module per unit; `pub(crate)` by default and `pub use` in `lib.rs` or
  `mod.rs` for the module's public API.
- **Tests**: unit tests in an inline `#[cfg(test)]` module, integration tests in
  `tests/`; a test file never lives inside `src/` as a separate file.
- Propagate errors with `?` and the project's error type; no `unwrap` or
  `expect` on a path that can fail in production code.
- Borrow at boundaries instead of cloning by default; keep global mutable state
  out; validate untrusted input at the edge.

### C# / .NET

- One project per bounded context; folders `Api/`, `Services/`,
  `Infrastructure/`, `Domain/`; one public type per file when the project adopts
  StyleCop `SA1402`.
- **Tests**: a separate `*.Tests` project mirroring the production namespace;
  a test type never lives in the production project.
- Enable nullable reference types; use `async`/`await` throughout and never
  block with `.Result` or `.Wait()`.
- Constructor injection and interfaces at the consumer; no service locator; keep
  persistence attributes out of the domain where the project allows it.

### PHP

- PSR-4: one autoloaded class per file, namespace mirroring the path;
  **tests** in `tests/`, mirroring `src/` (PHPUnit).
- Constructor injection and typed properties/parameters; no static service
  locator or global container access.
- Validate at the boundary; use domain exception classes; never suppress errors
  with `@`.
- Follow the detected framework's conventions (Laravel, Symfony) instead of
  imposing a second structure.

### Ruby

- `lib/` holds the code and `spec/` mirrors it; one class per file by
  convention, with module namespaces matching the path.
- **Tests**: `spec/` mirroring `lib/`, RSpec; a spec never lives under `lib/`.
- Service objects with keyword arguments; avoid monkey patching core classes;
  use `# frozen_string_literal: true`.
- Custom error classes and explicit rescue; never `rescue nil`; follow Rails
  autoloading and `app/` conventions when Rails is detected.

## Data Access

- Use the existing ORM/ODM or driver and its conventions.
- Keep queries parameterized; never build queries by string concatenation.
- Keep persistence concerns out of presentation and domain logic.
- Coordinate schema changes through the project's migration tooling; when a
  change touches the schema, migrations, queries, indexes, or transactions,
  load `database-engineering` for the safe, backward-compatible procedure.

## Mobile

- Follow the detected navigation, state, and platform conventions.
- Separate shared logic from platform-specific code.
- Respect offline, permission, and lifecycle patterns of the platform.
- Use the platform's safe-area insets, keyboard handling, and touch targets;
  never reuse desktop interaction patterns unchanged.
- Keep startup and first interaction fast on mid-range devices.
- Load `skills/design-analysis/SKILL.md` for mobile-specific analysis and
  `skills/internationalization/SKILL.md` when locales or RTL apply.

### React Native

- A feature holds `components/`, `hooks/`, `services/`, `types/`, and
  `context/` when it owns state, plus its public API; `App.tsx` is the entry
  point and `android/`/`ios/` stay at the repository root.
- Platform-specific files use the React Native suffix convention
  (`*.ios.tsx`, `*.android.tsx`); shared logic lives in a platform-agnostic
  module instead of being duplicated per platform.
- Native modules and device APIs sit behind a service adapter, never called
  directly from a component.
- **Tests** sit next to the unit; shared support lives in `testing/`.

### Ionic and Capacitor

- The layout follows the detected web framework: Angular's
  `components/`/`data-access/`/`services/`/`models/` or React's
  `components/`/`hooks/`/`services/`/`types/`.
- Capacitor plugins and native APIs sit behind a service adapter; a component
  never imports `@capacitor/*` directly.
- `android/` and `ios/` stay at the repository root, outside the web source
  tree.
- **Tests** sit next to the unit, following the web framework's convention.

## Relation to Other Components

- `architecture-advisor` decides structure, boundaries, and patterns.
- `api-contract-design` defines interfaces; this Skill implements both
  sides.
- `database-engineering` governs schema, migration, query, and transaction
  changes.
- `test-driven-development` and `testing-strategy` guide tests.
- `engineering-review` verifies the result.

## Output

### IMPLEMENTATION RECIPE

- Detected stack and versions
- Existing conventions and abstractions reused
- Files and modules to add or change, with their responsibility
- Cross-cutting concerns addressed
- Testing approach
- Validation commands
- Risks and deviations

## Safety

- Never introduce a dependency, pattern, or abstraction without the
  required assessment and authorization.
- Never weaken quality for speed on a simple task.
- Never leave the codebase less consistent or less testable than found.
