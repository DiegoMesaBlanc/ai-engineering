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

Backend:

- Node.js
- Python
- Java

Mobile:

- React Native
- Ionic

Databases:

- PostgreSQL
- MySQL
- Oracle
- MongoDB

TypeScript is preferred wherever the selected ecosystem supports it.

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

## Detection constraints

- Do not load every framework's documentation by default.
- Do not install dependencies merely to detect the stack.
- Do not assume that a framework feature is used because it is available.
- Load specialized Skills only when the task requires them.

## Output

Return a concise normalized stack profile.

Do not install dependencies merely to detect the stack.
