---
name: architecture-advisor
description: Evaluate architecture and design options and decide where code, types, contracts, and files belong — file cohesion, one primary unit per file, type-per-file placement, folder architecture, reuse, module conventions, and design-pattern selection. Load when deciding where new or changed code goes, when selecting design patterns from a task's forces, or when reviewing file, type, contract, and pattern implementation.
---

# Architecture Advisor

## Purpose

Recommend architecture changes only when justified by the task.

## Analyze

Consider:

- current architecture
- task complexity
- affected boundaries
- coupling
- cohesion
- dependencies
- data flow
- domain boundaries
- testability
- scalability
- security
- operational complexity

## Possible architectural approaches

Examples:

- simple module
- layered architecture
- modular monolith
- MVC
- Clean Architecture
- Hexagonal Architecture
- DDD
- event-driven architecture
- CQRS
- microservices

These are candidates, not defaults.

## Rules

Do not replace a working architecture merely because another architecture is considered more sophisticated.

Prefer incremental changes.

Prefer existing project conventions.

Avoid introducing new layers without a concrete responsibility.

Avoid premature microservices.

Avoid CQRS unless the problem justifies separate read/write models.

Avoid event sourcing unless historical state reconstruction is a real requirement.

## Recommendation format

CURRENT ARCHITECTURE

TASK IMPACT

OPTIONS

RECOMMENDATION

TRADE-OFFS

COMPLEXITY IMPACT

HUMAN DECISION REQUIRED

## Human approval

Require approval before implementing significant architectural changes.

## Architecture-First Engineering

Evaluate architecture, reuse, and code organization before writing code,
for every framework, language, and application type. This evaluation is part
of the canonical task flow; do not implement first and justify later.

### Evidence-Based Detection

Inspect the real project before proposing anything:

- the structure, entry points, and organization by domain, module, or
  responsibility
- imports, dependencies, and the boundaries between modules
- the components, hooks, contexts, services, controllers, models,
  interfaces, and adapters relevant to the detected stack
- configuration files, testing conventions, and framework rules
- existing architecture documentation and the patterns used by code that
  already works

Classify the result explicitly:

- **Detected with evidence**: the organization is confirmed by
  implementation, imports, boundaries, and configuration.
- **Inferred with uncertainty**: evidence suggests a structure but does not
  confirm it; state the uncertainty and what would confirm it.
- **No identifiable architecture**: no coherent organization is present.

Finding `components`, `services`, or `lib` directories does not, by itself,
establish an architecture. Base every claim on implementation evidence, not
directory names alone.

### Separate the Decisions

Distinguish, and decide independently:

- **Architectural style**: layering, Clean Architecture, hexagonal or
  ports-and-adapters, event-driven, or another applicable approach.
- **Organization strategy**: by feature/domain, by technical layer, or a
  justified combination.
- **Language and framework conventions**: the idiomatic structure of the
  detected ecosystem.

Feature organization is not exclusive to React. Java and Python can also be
organized by module or capability, using packages, modules, and conventions
appropriate to their ecosystems. Do not impose hexagonal, Clean, feature,
microservice, or any other option on every project. Justify the level of
separation from context and real needs.

#### Architecture Alternatives by Supported Ecosystem

Use this catalog as a set of alternatives to evaluate after detecting the
actual language, framework, and repository. It is not a mandatory mapping or
a complete list of every possible architecture. Include only alternatives
that fit the project and task, and use framework conventions as evidence for
organization, not as proof of an application architecture.

The **unit of organization**, the public API of a module, and the unit
folder for each ecosystem are defined in `references/folder-architectures.md`;
this table holds the organizational alternatives to consider.

| Detected ecosystem | Architectural and organization alternatives to consider when relevant |
| --- | --- |
| JavaScript/TypeScript with React or Next.js | Existing route or component organization; feature/domain modules; layered or modular application; server/client boundaries in Next.js. |
| Angular or Ionic Angular | Existing standalone-component or NgModule organization; feature modules; layered or domain-oriented modules; Ionic UI composition. |
| Vue, Nuxt, or Ionic Vue | Existing route/component organization; feature/domain modules; layered or modular application; Nuxt server/client boundaries when present. |
| Node.js with Express or Fastify | Simple route/module organization; layered application; modular monolith; ports-and-adapters where external boundaries justify it. |
| Node.js with NestJS | Existing module/provider/controller organization; feature modules; modular monolith; layered or domain-oriented boundaries. |
| Python with Django | Django app and model/view conventions; modular monolith; domain-oriented app boundaries where useful. |
| Python with FastAPI or Flask | Router/blueprint and package organization; simple layered modules; feature/domain packages; ports-and-adapters when integrations justify it. |
| Java with Spring Boot | Existing package and Spring module conventions; layered application; modular monolith; domain-oriented modules or Spring Modulith where boundaries justify it. |
| Kotlin with Spring Boot or Quarkus | Existing package and module conventions; layered application; feature modules; modular monolith |
| React Native with Expo or bare React Native | Existing navigation and platform conventions; feature/domain modules; shared versus platform-specific code; modular application. |
| SQL/database code | Schema and migration organization; bounded ownership by schema/module; procedural or event-driven patterns only when supported by workload needs. |
| Svelte / SvelteKit | Route and component organization; feature folders; layered application when the app is small |
| Solid | Component and route organization; feature folders; the ecosystem allows several components per file |
| Go | Package organization; layered packages, feature packages, or ports-and-adapters with an internal domain package |
| Rust | Crate and module organization; layered crate, feature modules, or domain/application/adapters modules |
| C# / .NET | Project and folder organization; layered folders, feature folders, or Clean Architecture projects |
| PHP | Composer package organization; layered folders, feature folders, or domain-oriented modules |
| Ruby | Rails app or gem organization; engine or module boundaries for multiple bounded contexts |

For ecosystems not listed, derive alternatives from the detected language,
framework, official documentation, and project conventions. Do not add a
technology-specific architecture Skill solely to extend this catalog.

### Types, Interfaces, and Contracts

**Rule: one interface/type per file in its owning layer's folder.** Exception: when multiple interfaces/types are **exclusively consumed by the same component/service/logical unit**, they are grouped in a single file (e.g., `ComponentTypes.ts`, `ServiceTypes.ts`, or co-located with the unit). Do not create separate files for types that always travel together. Each type lives in a **dedicated file** in the folder of the layer or module that owns it, never mixed with runtime behavior. The canonical wording is in `AGENTS.md`.

- **Contract ownership**: the type/contract lives in the folder of the layer or module that owns it. Consumers import from the contract module, never from an implementation module solely for types.
- **Shared by actual reuse**: a type is extracted to `types.ts`/`contracts.ts` when it is **imported/referenced from another file**. Mere export without external reuse is a WARN/observation, not a violation.
- **When a type is reused across files**, it belongs in the module's `types.ts`/`contracts.ts`, not inside the runtime implementation file. `lib/`, `util/`, `services/`, and similar folders with runtime code count as implementation folders.
- **Schema + inferred type pair**: Zod/Joi/Yup schemas and their `infer` types stay together in `schemas.ts`/`contracts.ts`; validators, converters, and clients go in a separate file.
- **Import direction**: type-only imports follow the same dependency direction as runtime imports. No presentation → infrastructure type imports when a contract module exists.
- **Language/framework conventions prevail**: do not impose the same physical structure on TypeScript, Java, Python, Go, Rust, etc. (see ecosystem matrix below).

Any exception must follow the project's conventions and be justified. The
architecture decision gate is the **Human Decision Gates** section in
`AGENTS.md`: when no coherent architecture is detected, when conventions
conflict, or when a new architecture must be chosen, present the options
and wait for the user's decision before implementing.

### Pass-Through Re-Exports

A module that consumes a contract does not re-export it. The owning file is
the single public path of the contract. This holds in every supported
ecosystem, in whatever form it forwards a contract.

- A component that imports its props from `./types` does not add
  `export type { XProps } from "./types";`. Consumers import from `./types`.
- A service that imports contracts from `./contracts` does not re-export
  them. Consumers import from `./contracts`.
- The same rule applies to the other forms of forwarding: a Go type alias
  (`type T = pkg.T`), a Rust `pub use` of a sibling, a Python `__all__`
  entry that only re-exposes an imported name, a Kotlin `typealias`, a PHP
  `class_alias()`, a C#/.NET `TypeForwardedTo`, a Ruby constant alias.
- A **pass-through re-export** gives one contract two public paths. It
  duplicates the contract surface, makes the owner ambiguous, and turns
  every rename into a coordination problem. It is removed.
- A barrel (`index.ts`) that aggregates **several** modules is a public API
  surface and may re-export them. A barrel that re-exports from a single
  source is a pass-through.
- Java is the only supported ecosystem with no forwarding mechanism: an
  import makes a type available by its simple name only inside its own
  compilation unit, and the language has no type alias. The rule does not
  apply there.
- A forwarding mechanism used with a documented purpose — an assembly
  migration in C#/.NET, a conditional class alias in PHP — is not a
  pass-through.

The canonical wording is in `AGENTS.md`.

### Separation of Responsibilities

Identify and correctly place the responsibilities relevant to the stack.

For React/Next.js, for example, evaluate components, hooks,
contexts/providers, services, API/adapters, types, utilities, and
configuration, and explain why each element belongs to its module.

For other detected stacks, identify the corresponding framework concepts
(such as Angular services and injectors, Vue composables and plugins,
Spring controllers and services, or Python routers and modules) without
translating them into React terminology or prescribing matching directories.

Do not create empty folders, do not require every such directory when it
does not apply, and do not merge distinct responsibilities into a generic
folder for convenience.

### Placement and Cohesion

Decide where components, hooks, services, configuration, types, and
interfaces belong.

- Group cohesive elements when it is reasonable.
- Separate elements with distinct responsibilities or that need reuse.
- Avoid monolithic files and unnecessary fragmentation.
- Do not impose a universal folder structure or a fixed number of files.

### File Cohesion and Responsibility Separation

A file should have one primary responsibility and one primary reason to
change. Separate these concerns into distinct files when they coexist:

- **Contracts/Types** (public API surface): interfaces, types, enums shared
  across modules or forming the module's public API. **Rule: one per file in the owning layer's folder.** Exception: multiple types with the **same exclusive consumer** are grouped in one file.
- **Static Data / Catalogs** (declarative data): message catalogs, locale
  metadata, configuration maps, lookup tables.
- **Configuration** (settings): constants, environment-derived values,
  feature flags.
- **Behavior** (pure functions, services): formatters, calculators,
  accessors, business logic.
- **Wiring / Composition** (glue): re-exports, factory functions,
  dependency injection setup.

Do not bundle shared contracts, catalogs, and behavior in the same file.

**Ownership determines folder/module, not file bundling.** A type
owned by a module goes in that module's folder, but in its own
`types.ts`/`contracts.ts` file — not inside the implementation file
that contains runtime behavior. Types with the same exclusive consumer
may share a file.

**Operational test**: if a file would change for unrelated reasons
(translations change vs. formatting logic vs. locale config), or if a
consumer cannot import the contract without pulling in data/runtime,
split it. This is not fragmentation (R-42); it is correct separation.

### Primary Unit per File and Stack Applicability

A file must not become a dumping ground for independent units. Where the
detected ecosystem supports it, keep one primary component, class, module,
or other unit per file. The rule is not universal: React official
documentation and Next.js do not mandate it, Solid and Kotlin explicitly
allow multiple related declarations, and several ecosystems have no
one-unit-per-file rule. Apply only the rules supported by the ecosystem's
official documentation or a recognized authoritative style guide, using the
applicability matrix in `skills/implementation-recipes/SKILL.md`.

- A file exporting several independent components with their own state,
  behavior, or lifecycle is a dumping ground and a File Cohesion finding,
  not a cohesive unit.
- Small private, non-exported helpers may remain with the unit they serve.
- Angular follows its official "one concept per file" rule, including the
  documented exception for small related components or directives; do not
  reduce it to a mechanical one-component rule.
- Barrels (`index.ts`) are a JS/TS module convention, optional and
  justified only by multiple public exports or consumers, a stable module
  boundary, a substantially populated folder, or an existing project
  convention. Never add one solely to re-export a single file. Do not
  invent barrel equivalents in ecosystems that do not use them.

### Folder Architecture and Contracts

The folder structure is **detected first**. If a coherent architecture exists, it is preserved and documented. If no clear architecture is detected, or the existing structure is incoherent (e.g., a flat `lib/` mixing unrelated responsibilities), **a project-appropriate structure is decided based on complexity** and documented in a **Folder Architecture Map**:

- **Layers/modules** and their responsibilities.
- **Responsibility → folder** mapping (components, hooks, services, domain, adapters, contracts/types, configuration).
- **Allowed dependency directions** between layers/modules.
- **Contract/type location per module**: shared contracts live in the owning layer's folder, **one per file**; types with the **same exclusive consumer** are grouped in one file.
- **Co-location rules** and **public module API** (barrels only when justified).
- **Unit-level grouping and test co-location**: where the ecosystem supports it, a unit with multiple related artifacts lives in its own folder with its test.

The type/contract placement rule derives from the folder map: **the type lives in the folder of the layer that owns the contract**.

**Complexity-based structure decision** (when no coherent architecture exists):

| Complexity | Recommended structure |
|---|---|
| Small, single domain | Layer-first: `src/api/<domain>/`, `src/services/<domain>/`, `src/server/<domain>/`, `src/shared/`, `src/components/`, `src/hooks/` |
| Medium, multiple domains | Feature-first: `src/features/<feature>/{api,domain,server,components,hooks,contracts}` + `src/shared/` |
| Large, multiple teams | Feature modules with explicit public APIs and boundaries; packages only when justified |

**Architectural style alternatives** (select based on domain complexity, team structure, deployment needs). The **concrete folder tree of each style, per ecosystem, is defined in `references/folder-architectures.md`**; that is where the user sees the folders:

| Style | When to use | Structure hint |
|---|---|---|
| Layered | Simple domains, single team, low coupling | `src/{api,services,domain,infrastructure}/` |
| Feature-first | Multiple domains, independent features | `src/features/<feature>/{api,domain,ui,contracts}/` |
| Hexagonal / Ports & Adapters | Multiple external integrations, testability critical | `src/{domain,application,adapters}/` with ports in domain |
| DDD (Domain-Driven Design) | Complex domain logic, ubiquitous language | `src/{domain,application,infrastructure,interfaces}/` with aggregates, entities, value objects |
| Modular Monolith | Multiple bounded contexts, single deployable | `src/modules/<context>/{domain,application,api}/` + `src/shared/` |
| Event-driven | Asynchronous integration between bounded contexts, queues or sagas required | `src/{producer,consumer,contracts,messaging}/` with events in `contracts/` |

The structure hints are TypeScript-shaped. Adapt them to the detected
ecosystem using the conventions catalog above: a Django project uses apps,
a Spring project uses packages, a Go project uses packages, a Rails project
uses engines or modules, and a .NET project uses projects or folders.

Microservices are a **deployment** decision, not a code-organization
default. Do not offer them for a new project without evidence that
independent deployment is a stated requirement.

**Evolution guidance**: As the project grows (e.g., adding auth, payments, notifications), **re-evaluate at each significant boundary change**. Indicators to migrate:
- Multiple domains with independent lifecycles → Feature-first
- Multiple external systems (DB, APIs, queues) → Hexagonal
- Complex domain rules with invariants → DDD
- Team ownership boundaries → Modular monolith

Migration follows R-42: only when a concrete threshold is met (reuse, size/complexity, required change, testability). Document the decision in the Folder Architecture Map.

Framework-specific route handlers (`app/api/...`, `routes/...`) remain in the framework's conventional location; they are thin and delegate to the domain/service layer.

**Universal responsibility rules** (apply regardless of chosen structure):

- `lib/` and `utils/` are for **generic, domain-agnostic utilities only**. Domain services, API clients, contracts, storage, and i18n catalogs do not belong there.
- **API has its own folder**: contracts/schemas and HTTP client together (`api/contracts.ts`, `api/client.ts` or feature `api/`), separate from services and storage.
- **Services/domain logic** in `services/` or `domain/`; **persistence** in `server/`, `storage/`, or `infrastructure/`.
- **Cross-cutting modules** (i18n, auth, config) live in `shared/` or a dedicated top-level folder, not flat in `lib/`.
- Route handlers delegate to services; they do not contain business logic.

A mixed `lib/` with unrelated responsibilities is an anti-pattern, not a convention to preserve. New code uses the correct folders; existing code moves only when touched (R-42).

### Feature-Driven Folder Architecture

In a feature-driven architecture the code is grouped by business capability,
not by technical kind. This is the target shape for a modern web
application:

```
src/
├── app/            # global configuration: providers, routes, global state
├── shared/         # generic and reusable, bound to no business capability
└── features/
    └── <feature>/
        ├── components/      # UI units owned by the feature
        │   └── <unit>/      # implementation + test + styles
        ├── hooks/           # behavioral units
        ├── services/        # API and transport calls
        ├── types/           # feature-level contracts
        ├── context/         # feature-local state, when the feature owns it
        └── index.ts         # the public API of the feature
```

**The feature root holds no source file.** The feature root contains only its
responsibility folders and its public API. A component, a hook, a service, or
a contract at the feature root is a misplaced artifact; it belongs to the
folder that owns its responsibility. The artifact's kind decides the folder:
UI goes to `components/`, behavior to `hooks/`, transport to `services/`,
feature-level contracts to `types/`, and local state to `context/`.

**Components and contracts never share a folder.** A contract consumed by
more than one unit lives in `types/`; a contract consumed by exactly one unit
lives in that unit's folder as `<unit>.types.ts`. It never sits at the
feature root and never directly inside `components/`, `hooks/`, or
`services/`. The full per-ecosystem translation, with a concrete tree for
every supported framework and language, is in
`references/folder-architectures.md` ("Feature responsibility folders per
ecosystem").

**One folder per unit inside its responsibility folder.** A unit that owns an
implementation and a test gets its own folder inside `components/`,
`hooks/`, or `services/`, with the test next to the implementation. A
responsibility folder never holds a flat pile of implementation files and
tests mixed together; that pile is the symptom of a missing decision about
which unit owns what.

**The feature exposes a public API.** `features/<feature>/index.ts` (or the
ecosystem equivalent) is the only entry point other parts of the application
may use. It exports what
they are allowed to use and nothing else; the rest of the feature stays
encapsulated.

This is a different kind of barrel from the convenience re-export described
above, and the two are not in conflict: a convenience barrel re-exports a
single file and is optional, while the public API of a feature is the
encapsulation boundary between capabilities and is required in this
architecture.

**The golden dependency rule.** A feature may import from `app/` and from
`shared/`. It must never import the internals of another feature. When two
features need to interact, they go through the other feature's public API,
or through the global state in `app/`. An import that reaches past the
public API is an architecture violation, not a convenience.

**Promotion to `shared/`.** A unit that starts inside a feature and later
needs a second consumer is **promoted**: it moves to `shared/`, in the
folder that matches its kind, and the feature that owned it imports it from
there. A unit is not copied into `shared/` to avoid the move; the feature
keeps a single owner.

**The system decides.** The folder layout is decided, not left to chance:
for each unit, the owner, the folder, and the public surface are recorded
before the files are created.

### Unit Folders and Co-located Tests

When a unit has more than one related artifact — implementation, test,
styles, stories, fixtures, or subcomponents — group them in a dedicated
folder named after the unit, and keep the test next to the unit it tests
where the ecosystem supports it.

- Do not leave `Component.tsx`, `Component.test.tsx`, and `Component/` as
  siblings. A unit folder plus the implementation and test outside it is a
  mixed structure and a finding, not a convention to preserve.
- A single trivial file with no test, styles, stories, or subcomponents may
  remain a single file. Create the folder when the unit has multiple related
  artifacts.
- The exact layout follows the ecosystem's official convention and the
  project's coherent conventions; the invariant is grouping by unit and
  co-locating tests when the ecosystem permits.
- Ecosystem examples:
  - React/Next.js: `Component/` containing `Component.tsx`,
    `Component.test.tsx`, styles, stories, and subcomponents. A barrel is
    optional, not part of the default layout. Solid may keep multiple
    related components in one file and still use a component folder for
    tests and assets.
  - Vue: `Component/` containing `Component.vue`, `Component.test.ts`,
    styles, and stories. A barrel is optional.
  - Svelte/SvelteKit: `Component/` containing `Component.svelte`,
    `Component.test.ts`, styles, and stories. A barrel is optional.
  - Angular: a component folder with `name.component.ts`,
    `name.component.html`, `name.component.css`, and `name.component.spec.ts`.
  - Java/Kotlin: one package per unit; tests in the mirrored test source
    set and package.
  - Python: `module.py` with `test_module.py` co-located or in a mirrored
    `tests/` tree, per project convention.
  - Go: package directory with `unit_test.go` in the same package.
  - Rust: module directory with a `#[cfg(test)]` module or a crate-level
    `tests/` directory.
  - C#/.NET: folder per unit; tests in the mirrored test project.
  - PHP: PSR-4 directory per class; tests in a mirrored directory.
  - Ruby: `lib/unit.rb` with `spec/unit_spec.rb` mirroring `lib/`.
- If the existing project deviates and the task touches the unit, migrate
  that unit within the touched scope (R-42). Do not reorganize untouched
  units.
- Consult `references/official-documentation.md` ("Project Structure and
  Test Co-location") before deciding the layout for the detected stack.

### Module Conventions and Reuse

Evaluate public exports, barrels, import aliases such as `@/`, and module
conventions.

- Reuse the existing conventions when they are viable.
- Do not introduce barrels, aliases, or new module conventions
  automatically; justify their utility and compatibility first.

### Fragmentation Gate

Extracting components, modules, or types; introducing barrels or aliases;
or reorganizing working code requires the same justification as
`skills/complexity-advisor/SKILL.md` (Refactoring and Abstraction
Justification): at least one threshold must be met with evidence
(reuse, size/complexity, required change, testability). If no threshold is
met, do not fragment. Record the observation as an Engineering Insight
instead of implementing.

### Design Patterns

Apply the canonical procedure in `references/design-patterns.md`.

- Extract the task's **design forces** and cite the evidence.
- Inventory existing patterns and framework mechanisms before adding one.
- Select every candidate whose force is present and not already resolved;
  reject every candidate whose force is absent.
- Translate each selected pattern to the detected stack's idiom.
- Derive the count; never target it. A task may need 0, 2, 32, or 100+.
- Record the selection and the rejected candidates.

A pattern without a signal is a defect.

### Dependencies, Tooling, and Styling

Inspect what already exists first. Reuse a compatible solution whenever it
is reasonable.

If no styling solution exists and the task requires one, compare relevant
alternatives and consult the user when the choice has significant
consequences for design, dependencies, or the visual experience.

Do not install new dependencies without the approval required by the
existing policies.

### Mandatory Decision Before Implementing

Establish an explicit checkpoint before writing code:

- If the project has a **coherent** architecture (detected structure with clear boundaries and consistent import directions), continue it unless there is
  an important incompatibility.
- If no clear architecture is detected, the detected structure is incoherent (e.g., flat `lib/` mixing domains), or conventions conflict, present the diagnosis and a
  recommendation with concrete alternatives and trade-offs, **including the complexity-based decision** from the table above.
- In that case, ask the user whether to preserve the existing organization
  or adopt the recommended alternative, adapting the options to the
  language and the project.
- **New project**: when there is no existing organization, the question is
  not "preserve or replace" but "which architectural style". Present the
  applicable styles from the table above with their trade-offs and ask which
  one to use. A recommendation is not a decision, and the structure is not
  created before the answer. Use `workflows/project-initialization.md`.
- Wait for the answer before implementing. Do not silently turn a
  recommendation into an approved decision.
- Do not ask about routine decisions that can be inferred from the code.
- If the user specifies an architecture, respect it and explain any
  significant incompatibility before implementing it.

The proposal must include a sufficiently concrete target structure, the
location of relevant files, the contracts and types, the direction of
dependencies, and the pending decisions. Stating "architecture evaluated"
is not sufficient.

### Technical References

Use these sources to inform decisions, not as universal templates:

- **Official Documentation Index**: `references/official-documentation.md` — authoritative, version-aware documentation references for every supported technology. **Consult this first** for the detected stack.
- Feature-Sliced Design:
  https://feature-sliced.design/docs/get-started/overview
- Hexagonal Architecture:
  https://alistaircockburn.com/Articles/Hexagonal-Architecture
- Azure architecture styles:
  https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/
- Spring Modulith:
  https://docs.spring.io/spring-modulith/reference/fundamentals.html
- FastAPI larger applications:
  https://fastapi.tiangolo.com/tutorial/bigger-applications/

Prioritize proven repository conventions and official language/framework
documentation (via `references/official-documentation.md`); use recognized
architectural methodologies to guide decisions rather than copying universal
templates.

### Autonomy and Decision Gates

Resolve routine technical details autonomously.

Consult the user only for an important product or architecture decision that
cannot be reasonably inferred from evidence and existing conventions.

### Decision Records

When a decision is significant, hard to reverse, or selects between
meaningful alternatives with real trade-offs, record it briefly: context,
decision, alternatives, and consequences. Reuse the project's existing
convention for storing such records; do not create a new documentation
location or a diagram unless it adds value.

Do not record trivial or easily reversible decisions, and never fabricate
rationale. Progressive over-documentation is a defect, not thoroughness.

### Post-Implementation Architectural Validation

After implementation, compare the real architecture with the proposal and
the approved decision, in addition to the functional tests.

Verify:

- structure, responsibilities, cohesion, coupling, and dependency direction
- components, hooks, services, configuration, types, and interfaces are
  placed consistently with the project's organization
- cohesive elements are grouped and distinct responsibilities are separated
- existing abstractions and conventions were reused where appropriate
- types, interfaces, and contracts are owned by the correct module; consumers import from the contract, not from an implementation module; language idiom respected
- no unnecessary abstraction, pattern, or fragmentation was introduced
- public exports, barrels, aliases, and module conventions remain consistent
- relevant configuration is consistent with the organization

Each important conclusion must include concrete evidence: files, modules,
imports, configuration, or executed commands. If a property cannot be
verified automatically, say so.

Distinguish between:

- documentation consistency of `ai-engineering`
- compliance with the flow before implementing
- architectural quality of the resulting code

Do not claim the agent's behavior is proven if only the instructions were
verified.

Report deviations and their impact. Do not silently accept a result that
diverges from the agreed design.

## Frontend UI Organization

Distinguish application architecture from frontend component organization
and UI design methodologies.

Application architecture candidates may include:

- Layered
- MVC
- Clean Architecture
- Hexagonal Architecture
- Modular Monolith
- Domain-Driven Design

Frontend organization candidates may include:

- Feature-based
- Domain-based
- Route-based
- Component-based
- Hybrid organization

UI organization and design-system methodologies may include:

- Atomic Design
- design systems
- composition-based component architecture
- shared component libraries

Atomic Design is a UI design and component-organization methodology, not
a replacement for the application's architectural style.

Use it only when it helps the project establish reusable UI components,
consistent design patterns, and a useful hierarchy.

Do not reorganize an existing application around Atomic Design merely
because the methodology is available.

Prefer the current structure when it works well.

Evaluate migration cost, reuse, component boundaries, coupling, and the
project's actual design-system needs before recommending a change.

## Microfrontend Architecture

Microfrontends are a frontend composition architecture in which a product
is assembled from independently developed and potentially independently
deployed frontend applications.

They are not a default choice for large frontends.

### Evaluate Microfrontends When

- independently owned frontend domains have meaningful deployment needs
- teams require independent release cycles
- bounded product areas can be developed and tested independently
- a staged migration from a legacy or different frontend technology is
  justified
- independent delivery provides a concrete organizational or technical
  benefit

### Evaluate the Costs

Inspect:

- application composition
- routing and navigation
- authentication boundaries
- communication between frontend applications
- shared dependencies and version compatibility
- design-system consistency
- state ownership
- bundle size and runtime performance
- deployment coordination
- observability
- testing complexity
- failure isolation

### Prefer a Modular Frontend When

- a single team owns the product
- independent deployment is not required
- feature boundaries are sufficient
- existing routing and component composition already solve the problem
- operational complexity would outweigh the benefits

A large application does not automatically require microfrontends.

Multiple routes do not automatically require microfrontends.

Multiple frontend teams alone are not sufficient evidence; evaluate their
ownership and deployment requirements.

### Technology Candidates

Depending on the actual project, evaluate:

- Module Federation
- single-spa
- another supported frontend composition mechanism

Do not select a technology before deciding whether microfrontends are
actually justified.

### Relationship to Repository Strategy

Microfrontends and monorepos are separate decisions.

A project may use:

- a monorepo with a modular frontend
- a monorepo containing multiple microfrontends
- separate repositories for independently deployed microfrontends

Evaluate both decisions independently.

### Approval

Introducing microfrontends or changing the composition architecture
requires human approval.

Report a simpler alternative whenever one can satisfy the requirements.
