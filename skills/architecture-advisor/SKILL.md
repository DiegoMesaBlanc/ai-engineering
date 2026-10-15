---
name: architecture-advisor
description: Evaluate architecture and design options for a software task while preserving the existing architecture when appropriate and avoiding unnecessary complexity.
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
| React Native with Expo or bare React Native | Existing navigation and platform conventions; feature/domain modules; shared versus platform-specific code; modular application. |
| SQL/database code | Schema and migration organization; bounded ownership by schema/module; procedural or event-driven patterns only when supported by workload needs. |

For ecosystems not listed, derive alternatives from the detected language,
framework, official documentation, and project conventions. Do not add a
technology-specific architecture Skill solely to extend this catalog.

### Types, Interfaces, and Contracts

Default rule:

- Separate reusable types, interfaces, and contracts from the
  implementations that consume them.
- Organize them by feature, domain, or module and group them by cohesion.
- Avoid a single global file that accumulates unrelated types, and avoid
  business interfaces hidden inside monolithic components.
- Respect language conventions; do not impose the same physical structure
  on TypeScript, Java, and Python.

Any exception must follow the project's conventions and be justified.

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

### Module Conventions and Reuse

Evaluate public exports, barrels, import aliases such as `@/`, and module
conventions.

- Reuse the existing conventions when they are viable.
- Do not introduce barrels, aliases, or new module conventions
  automatically; justify their utility and compatibility first.

### Design Patterns

Select a design pattern only when it solves a real need.

Prefer existing abstractions and framework capabilities over a new pattern.

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

- If the project has a coherent architecture, continue it unless there is
  an important incompatibility.
- If no clear architecture is detected, conventions conflict, or the task
  requires choosing a new architecture, present the diagnosis and a
  recommendation with concrete alternatives and trade-offs.
- In that case, ask the user whether to preserve the existing organization
  or adopt the recommended alternative, adapting the options to the
  language and the project.
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
documentation; use recognized architectural methodologies to guide
decisions rather than copying universal templates.

### Autonomy and Decision Gates

Resolve routine technical details autonomously.

Consult the user only for an important product or architecture decision that
cannot be reasonably inferred from evidence and existing conventions.

### Post-Implementation Architectural Validation

After implementation, compare the real architecture with the proposal and
the approved decision, in addition to the functional tests.

Verify:

- structure, responsibilities, cohesion, coupling, and dependency direction
- components, hooks, services, configuration, types, and interfaces are
  placed consistently with the project's organization
- cohesive elements are grouped and distinct responsibilities are separated
- existing abstractions and conventions were reused where appropriate
- types, interfaces, and contracts are separated from their consumers
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
