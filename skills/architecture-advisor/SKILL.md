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
