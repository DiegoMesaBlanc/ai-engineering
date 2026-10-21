---
name: stack-detection
description: Detect the programming language, framework, runtime, versions, libraries, database, ORM, testing tools, build system, and infrastructure used by a software project.
---

# Stack Detection

## Purpose

Determine the actual technology stack from repository evidence.

## Supported primary ecosystems

Frontend:

- React
- Angular
- Vue
- Next.js
- Svelte / SvelteKit
- Solid

Backend:

- Node.js
- Python
- Java
- Kotlin
- Go
- Rust
- C# / .NET
- PHP
- Ruby

Mobile:

- React Native
- Ionic

Databases:

- PostgreSQL
- MySQL
- Oracle
- MongoDB

TypeScript is preferred wherever the selected ecosystem supports it.

## Target Version

Detection reports the version found. It does not choose the version the
project will use.

- The **target version** comes from the source analysis, the ecosystem's
  official documentation, or a stated default. Record which of the three it
  came from.
- When the analysis states a version and the environment has another, report
  the difference. The project is created for the target version; the
  environment mismatch is reported, never silently resolved.
- Consult the official documentation **for the target version**, not only for
  the version installed. The consultation is recorded: source, version, and
  the decisions it informed.
- Report a version that could not be determined as not determined, not as the
  installed one.

## Detection rules

Never infer the stack solely from the user's description when repository evidence is available.

Inspect configuration and dependency files.

Determine:

- language
- runtime
- framework
- framework version
- package manager
- build system
- state management
- UI/component library
- styling system
- testing framework
- E2E framework
- formatter
- linter
- formatting and linting commands
- database
- ORM/ODM
- API framework
- CI/CD platform
- containerization
- infrastructure

## Framework-specific detection

For every detected framework, identify when available:

- framework and runtime version
- project entry points and configuration
- routing and rendering model
- state management and data access
- UI libraries and styling conventions
- build and testing configuration

Use repository evidence. Do not infer features solely from a framework's
version.

### React

- Detect React version and project entry points.
- Determine whether React is used directly or through a framework such as
  Next.js.
- Identify the existing routing, state management, UI, and testing
  conventions.

### Next.js

- Detect Next.js version and configuration.
- Identify App Router (`app/`, `src/app/`) or Pages Router
  (`pages/`, `src/pages/`) from repository structure.
- Identify Server and Client Components, Route Handlers, and other
  Next.js-specific features only when present.
- Preserve existing rendering, data-fetching, and routing conventions.

### Angular

- Detect Angular version, Angular CLI configuration, and project entry
  points.
- Identify standalone components or NgModules when present.
- Identify routing, RxJS, state management, UI libraries, and testing
  conventions.

### Vue

- Detect Vue version, project entry points, and build configuration.
- Identify Vue Router, Pinia, and single-file components when present.
- Detect Nuxt or other Vue-based frameworks when present.

### Node.js

- Detect Node.js runtime requirements and package manager.
- Identify backend frameworks such as Express, NestJS, or Fastify when
  present.
- Detect module system, API structure, validation, and testing conventions.

### Python

- Detect Python version requirements and package management using
  available project files.
- Identify frameworks such as Django, FastAPI, or Flask when present.
- Detect application entry points, dependency management, and testing
  conventions.

### Java

- Detect Java version and build system, such as Maven or Gradle.
- Identify frameworks such as Spring Boot when present.
- Detect module boundaries, dependency management, and testing conventions.

### React Native

- Detect React Native version and project entry points.
- Identify Expo or bare React Native configuration.
- Detect navigation, state management, native modules, and testing setup
  when present.

### Ionic

- Detect Ionic version and its React, Angular, or Vue integration.
- Identify Capacitor or Cordova when present.
- Detect routing, native plugins, styling, and testing conventions.

### Svelte and SvelteKit

- Detect Svelte and SvelteKit versions and the runes mode in use.
- Identify the adapter, the build tool, and the routing convention.
- Detect the test runner and the component testing library.

### Solid

- Detect Solid version, the start and entry files, and the router in use.
- Identify the build tool, the server-side rendering setup, and the test
  runner.
- Solid allows several components per file; do not assume one component per
  file.

### Kotlin

- Detect the Kotlin version, the JVM target, and the build system.
- Identify Spring Boot or Quarkus when present, and the package structure.
- Detect the test framework and the test source set.

### Go

- Detect the Go version from the module directive, the module path, and the
  build tool.
- Identify the package layout, the HTTP router or framework, and the ORM or
  database layer.
- Detect the test style: standard `testing` with table-driven tests, or a
  third-party assertion library.

### Rust

- Detect the Rust edition and toolchain, the workspace layout, and the crate
  names.
- Identify the web framework or the binary entry point, and the async
  runtime.
- Detect the test style: inline `#[cfg(test)]` modules, or a `tests/`
  directory.

### C# and .NET

- Detect the target framework, the SDK version, and the project kind: web
  API, worker service, class library, or console application.
- Identify ASP.NET Core, Entity Framework Core, and the test framework.
- Detect the solution layout and whether the repository is a multi-project
  solution.

### PHP

- Detect the PHP version requirement and the Composer manifest.
- Identify the framework: Laravel, Symfony, or a plain application.
- Detect the autoloading standard in use, PSR-4 or PSR-0.
- Detect the test framework and the test directory convention.

### Ruby

- Detect the Ruby version from the gemfile or the version file, and the
  framework: Rails, Sinatra, or a plain gem.
- Identify the bundler configuration, the app or gem structure, and the
  test framework: RSpec or Minitest.

## Formatter and linter detection

Every software project must have a configured formatter and linter within
the scope defined in AGENTS.md, so detect them from repository evidence
before selecting tools. Documentation-only knowledge repositories are out
of scope for the mandatory setup.

Inspect, when present:

- JavaScript and TypeScript: ESLint configuration, Prettier configuration,
  and related scripts in package.json
- Angular: Angular ESLint and Prettier configuration
- Python: Ruff configuration in pyproject.toml or a dedicated file, plus
  other configured formatters or linters
- Java: build-plugin configuration for formatters and linters such as
  google-java-format and Checkstyle
- SQL: SQLFluff or an equivalent dialect-aware configuration
- other ecosystems: the ecosystem-standard formatter and linter

Determine the configured commands, the package manager or build system that
runs them, and whether relevant source files are actually covered.

If the mandatory formatter or linter is absent in an in-scope software
project, report it as `SETUP_REQUIRED`; do not record it as
`NOT_APPLICABLE`.

When a required formatter or linter is missing, select a compatible
open-source tool per the default choices in AGENTS.md, matching the detected
framework and its major version, and read the existing package manager and
configuration format (for example, the detected ESLint major determines flat
config versus legacy eslintrc). Prefer a compatible already-installed
dependency over a new one, and do not add a second tool that duplicates one
already present. Record detected, configured, installed, executed, and
approved states separately from the check-result statuses. Configuration,
conflict avoidance, monorepo scoping, and installation are defined once in
`workflows/project-validation.md` (Phase 2 and Phase 5); do not restate that
procedure here.

## Database detection

Identify the database and version from available configuration and
dependency evidence.

Also detect:

- driver or client library
- ORM or ODM
- schema and migration tooling
- test database strategy

Do not assume a database is in use merely because its name appears in
documentation.

## Dependency inspection

Determine from repository evidence, to support dependency decisions:

- dependency manifests and lockfiles
- installed direct dependencies and relevant transitive dependencies
- declared and resolved dependency versions
- package manager, workspace, and shared-versus-package boundaries
- existing internal utilities, abstractions, and native or standard-library
  APIs that could satisfy a requirement without a new package

This inspection supplies evidence only. The dependency decision and its
authorization gate are canonical in `workflows/dependency-assessment.md`;
do not duplicate that decision policy here.

## Detection constraints

- Do not load every framework's documentation by default.
- Do not install dependencies merely to detect the stack.
- Do not assume that a framework feature is used because it is available.
- Load specialized Skills only when the task requires them.

## Output

Return a concise normalized stack profile.

Do not install dependencies merely to detect the stack.
