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

- **use arrow functions in TypeScript and JavaScript**; the `function`
  keyword is reserved for a generator, a `this` binding that arrow syntax
  cannot provide, the `arguments` object, or `new.target`, and the reason
  is stated in a comment
- **keep every interface, type, enum, and contract in a dedicated file**
  in the folder of the layer or module that owns it, never in the same file
  as runtime behavior; one type per file is the default, and several types
  are grouped in one file only when they are consumed by the same
  component, service, or logical unit
- **never re-export a contract you only consume**: no **pass-through
  re-export** of the props or contracts a module imports from `./types` or
  `./contracts`; the owning file is the single public path, in every
  supported ecosystem and in every form it forwards a contract
- follow the project's existing conventions and style, **except** that the
  rules above take **precedence over a detected project convention that
  violates them**: a convention that produces `function` declarations or
  mixed type/runtime files is a defect to correct in the code you write,
  not a convention to copy
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

**Structural rules are installed, not only documented.** For a JavaScript or TypeScript project, the ESLint configuration must include the structural compliance rules from `references/linter-rules.md`: `func-style`, the `no-restricted-syntax` selector for function expressions, and the local `structural/no-mixed-types-and-runtime` rule, plus the contract consumer script. A linter that runs without these rules does not enforce the structural policy of this system.

## Folder Architecture and Contract Placement

Follow the **Folder Architecture Map** from `architecture-advisor`. Place contracts/types in the folder of the owning layer. **Rule: one interface/type per file in its owning layer's folder.** Exception: when multiple interfaces/types are **exclusively consumed by the same component/service/logical unit**, they are grouped in a single file (e.g., `ComponentTypes.ts`, `ServiceTypes.ts`, or co-located with the unit). Do not create separate files for types that always travel together. Respect language idiom.

**Complexity-based structure decision** (when no coherent architecture exists):

| Complexity | Recommended structure |
|---|---|
| Small, single domain | Layer-first: `src/api/<domain>/`, `src/services/<domain>/`, `src/server/<domain>/`, `src/shared/`, `src/components/`, `src/hooks/` |
| Medium, multiple domains | Feature-first: `src/app/`, `src/shared/`, and `src/features/<feature>/` with one folder per unit (implementation + its test) and an `index.ts` public API for the feature. See **Feature-Driven Folder Architecture** in `skills/architecture-advisor/SKILL.md` |
| Large, multiple teams | Feature modules with explicit public APIs and boundaries; packages only when justified |

| Ecosystem | Contract/type location (rule: one per file; group if same exclusive consumer) | Import direction |
|---|---|---|
| TS/JS (React/Next.js) | Feature `types.ts` or `src/shared/types/<domain>.ts` (one type per export; group same-consumer types) | UI → feature/shared contract; never UI → `services` for types |
| Java/Kotlin (Spring) | `domain/` or `application/` package (one public type per file; group same-consumer types) | `api → application → domain`; no UI → infra |
| Python (FastAPI/Flask/Django) | `domain/contracts.py` or `domain/types.py` (one class per module preferred; group same-consumer types) | router → application/domain; not → infra |
| C#/.NET | `Domain/` or `Application/` (one public type per file; group same-consumer types) | `Api → Application → Domain` |
| Go | structs in `internal/domain`; **interfaces in consumer package** | no cycles; no provider-side interface |
| Rust | traits near type or `contracts/`; DTOs in common crate | respect orphan rules |
| PHP | `Domain/` or `Contracts/` (one class per file, PSR-4; group same-consumer types) | PSR-4, one type per file |
| Ruby | Sorbet/RBS in `sig/` or `rbi/` | no invented interfaces |

Do not impose a universal template; preserve the project's detected conventions **when coherent**. A mixed `lib/` is not a convention to preserve.

### Unit Organization and Test Co-location

When a unit has more than one related artifact — implementation, test,
styles, stories, fixtures, or subcomponents — use a folder per unit and put
the test next to the unit where the ecosystem supports it. Do not leave
`X.tsx`, `X.test.tsx`, and `X/` as siblings.

**Primary unit per file — canonical applicability matrix:**

| Ecosystem | Primary unit per file | Test location | Authority / applicability |
|---|---|---|---|
| React / Next.js | One React component per file when the project accepts Airbnb + ESLint `react/no-multi-comp`; multiple stateless/pure components allowed. React documentation and Next.js do not mandate this. | Vitest/Jest next to the unit or project convention | Airbnb, ESLint; Next.js is unopinionated |
| Angular | One concept per file; official exception for small related components/directives | `*.spec.ts` in the same directory | Angular Style Guide |
| Vue | One component per `.vue`; at most one `<template>`/`<script>` | Vitest next to the unit | Vue SFC specification |
| Svelte / SvelteKit | A `.svelte` file is one component | Vitest or project convention | Svelte documentation |
| Solid | Multiple components per file are explicitly allowed; do not enforce one per file | Vitest or project convention | Solid documentation |
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
| Contracts / Types (public API) | `types.ts` / `contracts.ts` / `domain/*.ts` | Shared interfaces, types, enums, DTOs. |
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
- Follow the project's **Folder Architecture Map**: feature folders with `components/`, `hooks/`, `services/`, `types.ts`; shared code in `src/shared/{components,hooks,lib,types}`.
- **Contracts**: shared domain types in feature `types.ts` or `src/shared/types/<domain>.ts`; props/view models co-located in the component.
- **Imports**: UI imports contracts from feature/shared types, never from `services` or `lib` implementation modules.
- Keep server and client boundaries explicit; do not leak server-only code to the client.
- Manage state with the project's existing solution; avoid introducing a new one without justification.
- Handle loading, empty, and error states in data access.
- Follow the framework's data-fetching and caching conventions.
- **Function style**: use **arrow functions** (`const fn = () => {}`) for components, hooks, utilities, and callbacks. The `function` keyword is reserved for the legitimate reasons listed in `AGENTS.md` (a generator, a `this` binding that arrow syntax cannot provide, the `arguments` object, or `new.target`). This applies to `.ts`, `.tsx`, `.js`, and `.jsx`.
- **Component style**: Prefer `const Component = ({ props }) => {}` over `function Component({ props }) {}`.
- **Hook style**: `const useHook = () => {}` not `function useHook() {}`.
- **Callback style**: `const handleClick = () => {}` not `function handleClick() {}`.

### Angular

- Follow the standalone-component or NgModule organization actually used.
- **Contracts**: shared types in feature `types.ts` or a shared types module; component inputs/outputs co-located. Use a barrel only when justified.
- **Imports**: components import from feature/shared types, not from service implementation files.
- Use the framework's dependency injection and services; keep components thin.
- Respect RxJS conventions and unsubscribe/cleanup patterns.
- Use the detected state, routing, and form libraries consistently.

### Vue / Nuxt

- Use the detected composition/options style and component conventions.
- **Contracts**: shared types in feature `types.ts` or `composables/types.ts`; props co-located.
- **Imports**: components import from feature/shared types, not from service files.
- Place composables, stores, and services by cohesion.
- Respect Nuxt server/client boundaries when present.

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
