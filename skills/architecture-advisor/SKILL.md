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
