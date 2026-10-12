# Official Documentation Index

## Purpose

Provide authoritative, version-aware documentation references for every
technology supported by the AI Engineering System.

This index contains URLs and selection guidance, not copies of the
documentation.

Consult the smallest relevant source on demand.

## Global Rules

1. Detect the actual stack, runtime, and installed framework versions.
2. Prefer official documentation maintained by the technology owner.
3. Select documentation compatible with the detected version.
4. Do not assume that latest-version features exist in older projects.
5. Inspect deprecations and migration guidance when changing APIs.
6. Use project source code, type definitions, tests, and lockfiles as
   complementary evidence.
7. Never claim to have read a page that was not retrieved.
8. Do not browse or load every reference for every task.
9. Do not require MCP for public documentation that can be retrieved
   through existing tools.
10. Preserve project conventions unless there is a justified reason to
    change them.

## Frontend

### React

- Official documentation: https://react.dev/
- Learn: https://react.dev/learn
- API reference: https://react.dev/reference/react
- Rules of React: https://react.dev/reference/rules

### Angular

- Official documentation: https://angular.dev/docs
- API reference: https://angular.dev/api/
- Style guide: https://angular.dev/style-guide
- Agent Skills: https://angular.dev/ai/agent-skills

Detect the installed version before using Signals, Signal Forms, SSR,
routing, or other version-sensitive APIs.

### Vue

- Official guide: https://vuejs.org/guide/
- API reference: https://vuejs.org/api/
- TypeScript with Vue: https://vuejs.org/guide/typescript/overview.html
- Vue Router: https://router.vuejs.org/
- Pinia: https://pinia.vuejs.org/

### Next.js

- Official documentation: https://nextjs.org/docs
- App Router: https://nextjs.org/docs/app
- Pages Router: https://nextjs.org/docs/pages
- Upgrade guidance: https://nextjs.org/docs/app/guides/upgrading

Detect App Router or Pages Router from the actual repository.

Do not create a dedicated Next.js Skill solely to repeat documentation
links or framework detection rules.

### TypeScript

- Official documentation: https://www.typescriptlang.org/docs/
- TSConfig reference: https://www.typescriptlang.org/tsconfig/

### JavaScript

- MDN JavaScript guide: https://developer.mozilla.org/docs/Web/JavaScript/Guide
- MDN JavaScript reference: https://developer.mozilla.org/docs/Web/JavaScript/Reference

Use official runtime and framework documentation when a question concerns
a runtime-specific or framework-specific behavior.

## Backend

### Node.js

- Official documentation: https://nodejs.org/docs/
- API reference: https://nodejs.org/docs/latest/api/

### Express

- Official documentation: https://expressjs.com/en/

### NestJS

- Official documentation: https://docs.nestjs.com/

### Python

- Official documentation: https://docs.python.org/3/
- Language reference: https://docs.python.org/3/reference/

### Django

- Official documentation: https://docs.djangoproject.com/

Select the documentation version matching the installed Django version.

### FastAPI

- Official documentation: https://fastapi.tiangolo.com/

### Java

- Official Java documentation: https://docs.oracle.com/en/java/
- Java learning resources: https://dev.java/learn/

### Spring

- Official documentation: https://docs.spring.io/spring-framework/reference/
- Spring Boot: https://docs.spring.io/spring-boot/index.html

Detect the Java version, build system, and Spring version before selecting
guidance.

## Mobile

### React Native

- Official documentation: https://reactnative.dev/docs/getting-started
- Architecture: https://reactnative.dev/architecture/landing-page
- Upgrading: https://reactnative.dev/docs/upgrading

### Ionic

- Official documentation: https://ionicframework.com/docs
- Components: https://ionicframework.com/docs/components
- Angular integration: https://ionicframework.com/docs/angular/overview
- React integration: https://ionicframework.com/docs/react/overview
- Vue integration: https://ionicframework.com/docs/vue/overview

### Capacitor

- Official documentation: https://capacitorjs.com/docs

## Databases

### PostgreSQL

- Official documentation: https://www.postgresql.org/docs/
- SQL commands: https://www.postgresql.org/docs/current/sql-commands.html

### MySQL

- Official documentation: https://dev.mysql.com/doc/
- Reference manual: https://dev.mysql.com/doc/refman/en/

### Oracle Database

- Official documentation: https://docs.oracle.com/en/database/
- SQL language reference: https://docs.oracle.com/en/database/oracle/oracle-database/

Select documentation for the actual Oracle Database version when known.

### MongoDB

- Official documentation: https://www.mongodb.com/docs/
- Manual: https://www.mongodb.com/docs/manual/

## UI, Styling, and Components

### Tailwind CSS

- Official documentation: https://tailwindcss.com/docs

### Material UI

- Official documentation: https://mui.com/material-ui/getting-started/

### PrimeNG

- Official documentation: https://primeng.org/

Use the UI library already present in the project unless a change is
explicitly justified.

## Testing and Validation

### Vitest

- Official documentation: https://vitest.dev/guide/

### Jest

- Official documentation: https://jestjs.io/docs/getting-started

### Playwright

- Official documentation: https://playwright.dev/docs/intro

### Cypress

- Official documentation: https://docs.cypress.io/

### Testing Library

- Official documentation: https://testing-library.com/docs/

### ESLint

- Official documentation: https://eslint.org/docs/latest/

### Prettier

- Official documentation: https://prettier.io/docs/

## Security

### OWASP

- OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/
- Application Security Verification Standard: https://owasp.org/www-project-application-security-verification-standard/

Use security references relevant to the application's actual threat model
and technology.

## Selection Workflow

For every task:

1. Detect the technology and version.
2. Identify the specific technical question or implementation concern.
3. Select one or more relevant documentation references.
4. Retrieve only the sections needed to resolve the question.
5. Compare the guidance against existing code and project conventions.
6. Implement the smallest suitable change.
7. Validate the result with available project tooling.

## External Skills

Documentation sources and Skills are separate resources.

A documentation URL does not need to become a Skill.

An external Skill must provide actionable knowledge or workflow beyond
what can be achieved by consulting documentation alone.

Evaluate every external Skill for source, license, maintenance,
compatibility, token cost, and policy conflicts before importing it.

Load external Skills only when the detected stack and task justify them.

## Unavailable Sources

If documentation cannot be accessed:

- report the limitation when material
- use reliable local evidence where possible
- avoid inventing version-specific behavior
- ask for additional context only when necessary
