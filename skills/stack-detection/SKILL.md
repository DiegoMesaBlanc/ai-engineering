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

React:

Inspect React version and relevant ecosystem libraries.

Next.js:

Inspect Next.js version and relevant ecosystem libraries.

Angular:

Inspect Angular version, RxJS, state management, UI libraries and build configuration.

Vue:

Inspect Vue version, router, state management and build configuration.

Node.js:

Detect framework and runtime version.

Python:

Detect framework, package manager and project layout.

Java:

Detect build system, Java version and framework.

React Native:

Detect React Native version, Expo or native build setup.

Ionic:

Detect Ionic, Angular/Vue/React integration and Capacitor when present.

## Output

Return a concise normalized stack profile.

Do not install dependencies merely to detect the stack.
