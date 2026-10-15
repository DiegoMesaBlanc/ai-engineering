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

### Inspect Before Proposing

Inspect the real project architecture and conventions before proposing
changes:

- feature or module organization
- separation of responsibilities and layers
- existing reusable abstractions
- dependencies and their direction
- component and module boundaries

Use implementation evidence, not directory names alone.

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

Inspect existing dependencies, available tools, and styling conventions
before choosing a solution.

If the project needs a new styling system, compare relevant alternatives and
consult the user when the decision significantly affects architecture,
dependencies, or the visual experience.

### Autonomy and Decision Gates

Resolve routine technical details autonomously.

Consult the user only for an important product or architecture decision that
cannot be reasonably inferred from evidence and existing conventions.

### Post-Implementation Architectural Validation

After implementation, contrast the result with the proposed design and the
project's conventions, in addition to the functional tests.

Verify:

- components, hooks, services, configuration, types, and interfaces are
  placed consistently with the project's organization
- cohesive elements are grouped and distinct responsibilities are separated
- existing abstractions and conventions were reused where appropriate
- no unnecessary abstraction, pattern, or fragmentation was introduced
- public exports, barrels, aliases, and module conventions remain consistent

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
