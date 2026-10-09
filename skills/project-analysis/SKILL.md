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

Use implementation evidence.

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
