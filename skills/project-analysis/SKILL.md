---
name: project-analysis
description: Analyze an existing software repository to understand its structure, architecture, dependencies, conventions, testing, CI/CD, and technical risks before modifying it.
---

# Project Analysis

## Purpose

Build an evidence-based understanding of an existing project.

## Analyze

Inspect only what is relevant to the task.

Consider:

- repository structure
- source boundaries
- application entry points
- modules/features
- dependency graph
- configuration
- environment handling
- API boundaries
- database access
- state management
- authentication and authorization
- error handling
- logging
- testing
- CI/CD
- Docker
- infrastructure
- documentation

## Architecture detection

Identify the architecture actually present.

Possible examples include:

- layered architecture
- modular monolith
- MVC
- feature-based architecture
- Clean Architecture
- Hexagonal Architecture
- DDD-oriented structure
- microservices
- event-driven architecture

Do not classify an architecture solely from directory names.

Use implementation evidence:

- real structure, entry points, and organization by domain, module, or
  responsibility
- imports, dependencies, and the boundaries between modules
- the components, hooks, contexts, services, controllers, models,
  interfaces, and adapters relevant to the detected stack
- configuration files, testing conventions, and framework rules
- existing architecture documentation and the patterns used by working code

Report the confidence of the classification explicitly:

- detected with evidence
- inferred with uncertainty, stating what would confirm it
- no identifiable architecture

Finding `components`, `services`, or `lib` directories does not, by itself,
establish an architecture.

## Existing abstractions

Identify reusable abstractions before proposing new ones.

Examples:

- services
- repositories
- adapters
- factories
- facades
- domain objects
- validators
- API clients
- shared components
- hooks
- utilities

## Module and Import Conventions

Detect the conventions actually used, when present:

- feature-based or module-based organization
- public exports and barrel files
- import aliases such as `@/`
- shared versus feature-local code
- component, hook, service, and type placement

Report the conventions found. Do not introduce new conventions during
analysis.

**A convention that violates the structural policy is reported as a defect,
not as a convention to follow.** When the repository uses `function`
declarations, or mixes type declarations with runtime behavior in the same
file, report it explicitly as an existing defect in the detected code. The
structural policy takes **precedence over a detected project convention
that violates it**: the code written from now on follows the policy in
`AGENTS.md` and does not reproduce it.

## Folder Architecture Map

Detect and report the **Folder Architecture Map** when present:

- **Layers/modules** and their responsibilities.
- **Responsibility → folder** mapping (components, hooks, services, domain, adapters, contracts/types, configuration).
- **Allowed dependency directions** between layers/modules (evidence from imports).
- **Contract/type location per module**: where shared contracts/types live (**rule: one per file in the owning layer's folder**; types with the **same exclusive consumer** are grouped in one file).
- **Public module API** (barrels, explicit exports) and co-location conventions.
- **Unit-level grouping and test co-location**: whether a unit with multiple artifacts (implementation, test, styles, stories, subcomponents) lives in its own folder with its test, or whether `X`, `X.test`, and `X/` are siblings, where the ecosystem supports that grouping.
- **Language-specific idioms** observed (e.g., Go consumer-side interfaces, Java one-public-type-per-file, Python Protocols, Ruby RBS/Sorbet).

Do not classify from directory names alone; base on implementation evidence (imports, boundaries, configuration). If no coherent map exists, report "no identifiable folder architecture" and **assess project complexity** to recommend a structure per `architecture-advisor`:

- **Small, single domain** → layer-first (dedicated `api/`, `services/`, `server/`, `shared/` folders).
- **Medium, multiple domains** → feature-first (`src/features/<feature>/{api,domain,server,components,hooks,contracts}` + `src/shared/`).
- **Large, multiple teams** → feature modules with explicit public APIs and boundaries.

### Dumping Ground Detection

Inspect `lib/`, `utils/`, `services/`, and similar flat folders for **mixed responsibilities**:

- Domain services, API clients, contracts, storage, and i18n catalogs co-located in one folder.
- Files in `lib/` imported by only one feature/domain (not genuinely shared).
- No clear responsibility → folder mapping.

If found, report as **"legacy dumping ground; do not replicate for new code"**. New code must follow the correct structure per the complexity decision above.

## Internal Module Layout Detection

For each significant module (feature, shared lib, domain), detect and
report the internal file structure:

- Whether contracts/types are in a dedicated `types.ts`/`contracts.ts`
  or mixed with implementation.
- Whether static data/catalogs (i18n messages, config maps, lookup
  tables) are in separate files (`messages/`, `catalogs/`, `data/`) or
  inline with logic.
- Whether formatters, helpers, and behavior are in dedicated files
  (`format.ts`, `utils.ts`, `helpers.ts`, `services/`).
- Whether a public API barrel (`index.ts`) exists and is justified; barrels
  are optional and a single re-export is not sufficient justification.
- Whether tests are co-located with the unit they test, following the
  ecosystem convention (React/Vue/Svelte/Angular component folder,
  Java/Kotlin mirrored test package, Python `test_<module>.py`,
  Go `_test.go`, Rust `tests/`, etc.).
- Whether a unit with multiple artifacts uses a dedicated folder or leaves
  `X`, `X.test`, and `X/` as siblings, where the ecosystem supports it.
- Whether a file exports multiple independent components/units with their
  own behavior, and whether the detected ecosystem supports a primary-unit-
  per-file rule (see the applicability matrix in
  `skills/implementation-recipes/SKILL.md`).
- Language-specific conventions (e.g., Java one-public-type-per-file,
  Python `domain/contracts.py`, Go consumer-side interfaces).
- Whether schema + inferred types (Zod/Joi/Yup) are co-located with validators/converters (should be separated).

Record the observed conventions. Do not refactor automatically; use
this to align generation and audit with the project's actual patterns.
If the project has monolithic module files (e.g., `lib/i18n.ts` with
contracts, data, and behavior, or `lib/payment-api.ts` with schemas,
types, validators, and converters in one file), note it as the existing convention but
flag new code for separation per `architecture-advisor` File Cohesion
rules and the reuse-based type ownership rule.

## Output

Report:

- Current structure
- Current architecture
- Existing conventions
- Existing reusable abstractions
- Testing approach
- Important risks
- Relevant technical debt

Do not modify unrelated code.

## Repository Topology Detection

Determine whether the project is:

- a single application repository
- a monorepo
- a collection of separately managed repositories
- a frontend/backend multi-repository system
- a repository containing microfrontends
- a monorepo that contains several applications or packages

Use evidence from:

- workspace configuration
- application entry points
- package and dependency relationships
- CI/CD pipelines
- deployment configuration
- ownership rules
- repository boundaries

Report the evidence used for the classification.

## Frontend Composition Detection

Determine whether the frontend is:

- a single application
- a modular frontend
- a component library plus applications
- a microfrontend composition
- an unclear or partially migrated architecture

Inspect existing framework and build configuration before making
recommendations.

Do not infer microfrontends merely because multiple applications or
frameworks are present in the repository.
