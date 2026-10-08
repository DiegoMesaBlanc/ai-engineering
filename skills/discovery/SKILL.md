---
name: discovery
description: Analyze a software project before implementation. Determine whether the project is new or existing, inspect its repository, detect the technology stack, architecture, conventions, testing setup, and relevant capabilities, then identify only the information and Skills required for the current task.
---

# Discovery

## Purpose

Discovery is the mandatory first stage before implementing non-trivial software changes.

The objective is to understand the project before making architectural or implementation decisions.

## Mandatory behavior

Before implementing a task:

1. Determine whether the project is new or existing.
2. Inspect the repository structure.
3. Inspect relevant configuration files.
4. Detect the programming language.
5. Detect the framework.
6. Detect framework and runtime versions when possible.
7. Detect package manager and build system.
8. Detect database technology when applicable.
9. Detect ORM/ODM when applicable.
10. Detect state management when applicable.
11. Detect testing frameworks.
12. Detect E2E tooling.
13. Detect linting and formatting configuration.
14. Detect CI/CD configuration.
15. Detect containerization and infrastructure configuration.
16. Detect the current architecture.
17. Detect existing project conventions.
18. Detect existing abstractions and design patterns when reasonably identifiable.
19. Determine which Skills are relevant to the task.
20. Load only the Skills that are relevant.

## Existing projects

Do not ask the user to provide information that can be reliably detected from the repository.

Inspect the repository first.

Prefer evidence from:

- package.json
- package-lock.json
- pnpm-lock.yaml
- yarn.lock
- pom.xml
- build.gradle
- pyproject.toml
- requirements.txt
- poetry.lock
- angular.json
- vite.config.\*
- tsconfig.json
- Dockerfile
- docker-compose files
- CI/CD configuration
- source structure
- test configuration
- README files
- architecture documentation

## New projects

For new projects, determine the minimum information required before implementation.

Ask only questions that materially affect:

- architecture
- technology selection
- data model
- security
- deployment
- testing strategy
- major functional behavior

Do not ask questions whose answers can be safely decided using established project defaults.

## Human interaction

Human interaction may be conducted in Spanish or English.

Do not translate source code requirements into non-English identifiers.

## Output

Before implementation, provide a concise discovery summary:

PROJECT STATE
STACK
ARCHITECTURE
CONVENTIONS
TESTING
INFRASTRUCTURE
RELEVANT SKILLS
UNKNOWN DECISIONS

Do not produce unnecessary detail.

## Important constraint

Discovery is not implementation.

Do not modify application code during discovery unless explicitly requested.
