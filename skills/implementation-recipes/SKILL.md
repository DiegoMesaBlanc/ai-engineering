---
name: implementation-recipes
description: Practical implementation recipes for the supported stacks so agents build maintainable, professional code rather than the minimum that merely works.
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

- follow the project's existing conventions and style
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
2. Reuse existing abstractions, utilities, and patterns (see
   `architecture-advisor`).
3. Identify the cross-cutting concerns the change touches (validation,
   errors, auth, config, logging, testing).
4. Implement in the smallest cohesive unit that satisfies the design.
5. Validate with the project's formatter, linter, type checker, and tests.
6. Review the result against the agreed design and conventions.

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

## Frontend Recipes

### React and Next.js

- Match the detected router (App Router or Pages Router) and rendering
  model (server/client components).
- Place components, hooks, services, adapters, and types by cohesion,
  following the project's organization.
- Keep server and client boundaries explicit; do not leak server-only code
  to the client.
- Manage state with the project's existing solution; avoid introducing a
  new one without justification.
- Handle loading, empty, and error states in data access.
- Follow the framework's data-fetching and caching conventions.

### Angular

- Follow the standalone-component or NgModule organization actually used.
- Use the framework's dependency injection and services; keep components
  thin.
- Respect RxJS conventions and unsubscribe/cleanup patterns.
- Use the detected state, routing, and form libraries consistently.

### Vue / Nuxt

- Use the detected composition/options style and component conventions.
- Place composables, stores, and services by cohesion.
- Respect Nuxt server/client boundaries when present.

## Backend Recipes

### Node.js (Express, Fastify, NestJS)

- Keep routing, controllers, services, and data access separated per the
  project's structure.
- Validate inputs at the boundary; centralize error handling.
- Use the detected validation, ORM, and configuration libraries.
- For NestJS, respect modules/providers/controllers; for Express/Fastify,
  keep handlers thin and logic in services.

### Python (Django, FastAPI, Flask)

- Follow framework conventions: Django apps and models, FastAPI routers and
  dependencies, Flask blueprints.
- Keep business logic out of views and routers.
- Use the project's validation and serialization approach.
- Apply migrations and schema changes through the project's tooling.

### Java (Spring Boot)

- Respect package and module boundaries; keep controllers, services, and
  repositories separated.
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
