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

## Internationalization

### Unicode CLDR

- Project: https://cldr.unicode.org/

### ICU MessageFormat

- User guide: https://unicode-org.github.io/icu/userguide/format_parse/messages/

### ECMAScript Intl

- Reference: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl

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

### Angular ESLint

- Official documentation: https://github.com/angular-eslint/angular-eslint

### Ruff

- Official documentation: https://docs.astral.sh/ruff/

### google-java-format

- Official documentation: https://github.com/google/google-java-format

### Checkstyle

- Official documentation: https://checkstyle.org/

### SQLFluff

- Official documentation: https://sqlfluff.com/

Use the formatter and linter appropriate to the detected stack and the
project's existing configuration, as required by the Formatting and
Linting Policy in AGENTS.md.

## Security

### OWASP

- OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/
- Application Security Verification Standard: https://owasp.org/www-project-application-security-verification-standard/

### Dependency and Secret Scanning (open source)

- OWASP Dependency-Check: https://dependency-check.github.io/
- OSV-Scanner: https://google.github.io/osv-scanner/
- Trivy (vulnerabilities, misconfigurations, IaC): https://aquasecurity.github.io/trivy/
- gitleaks (secret scanning): https://github.com/gitleaks/gitleaks

Use security references relevant to the application's actual threat model
and technology, and prefer the project's existing or approved tooling.

## CI/CD and Delivery

### GitHub Actions

- Official documentation: https://docs.github.com/en/actions
- Workflows: https://docs.github.com/en/actions/using-workflows
- Security hardening: https://docs.github.com/en/actions/security-guides

### GitLab CI/CD

- Official documentation: https://docs.gitlab.com/ci/
- Pipeline security: https://docs.gitlab.com/ci/pipelines/

### Azure Pipelines

- Official documentation: https://learn.microsoft.com/en-us/azure/devops/pipelines/
- Security and secrets:
  https://learn.microsoft.com/en-us/azure/devops/pipelines/security/

### Bitbucket Pipelines

- Official documentation:
  https://support.atlassian.com/bitbucket-cloud/docs/configure-bitbucket-pipelinesyml/

Detect the platform actually used by the project and consult the
corresponding documentation. CI/CD and delivery decisions follow
`workflows/secure-delivery.md`.

## Architecture, Design Patterns & Clean Code (Foundational)

These are the authoritative sources that inform our internal rules. Consult when making architectural decisions, evaluating patterns, or establishing conventions.

### Clean Architecture & SOLID Principles

- **Robert C. Martin (Uncle Bob) - Clean Architecture**: https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
- **Clean Architecture Book**: https://www.oreilly.com/library/view/clean-architecture-a/9780134494272/
- **SOLID Principles**: https://en.wikipedia.org/wiki/SOLID
- **The Dependency Rule**: https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html#the-dependency-rule

### Domain-Driven Design (DDD)

- **Eric Evans - Domain-Driven Design (Blue Book)**: https://domainlanguage.com/ddd/
- **DDD Reference (Vaughn Vernon)**: https://domainlanguage.com/ddd/reference/
- **Implementing DDD (Vaughn Vernon)**: https://vaughnvernon.co/?page_id=168
- **DDD Tactical Patterns**: https://martinfowler.com/bliki/DomainDrivenDesign.html

### Hexagonal Architecture (Ports & Adapters)

- **Alistair Cockburn - Hexagonal Architecture**: https://alistaircockburn.com/Articles/Hexagonal-Architecture/
- **Ports and Adapters**: https://herbertograca.com/2017/11/16/hexagonal-architecture-ports-adapters/

### Feature-Sliced Design (Frontend Architecture)

- **Official Methodology**: https://feature-sliced.design/docs/get-started/overview
- **Layers & Slices**: https://feature-sliced.design/docs/reference/layers
- **Public API**: https://feature-sliced.design/docs/reference/public-api

### Modular Monolith & Bounded Contexts

- **Modular Monolith (Kamil Grzybek)**: https://github.com/kgrzybek/modular-monolith-with-ddd
- **Modular Monolith vs Microservices**: https://www.martinfowler.com/articles/modular-monolith.html
- **Context Mapping (DDD Strategic Design)**: https://martinfowler.com/bliki/BoundedContext.html

### Clean Code & Code Quality

- **Robert C. Martin - Clean Code**: https://www.oreilly.com/library/view/clean-code/9780136083238/
- **Clean Code Principles**: https://github.com/ryanmcdermott/clean-code-javascript
- **Code Smells & Refactoring (Martin Fowler)**: https://martinfowler.com/books/refactoring.html
- **Refactoring Catalog**: https://refactoring.com/catalog/

### Software Architecture Patterns

- **Patterns of Enterprise Application Architecture (Martin Fowler)**: https://martinfowler.com/books/eaa.html
- **Enterprise Integration Patterns**: https://www.enterpriseintegrationpatterns.com/
- **CQRS (Greg Young)**: https://martinfowler.com/bliki/CQRS.html
- **Event Sourcing**: https://martinfowler.com/eaaDev/EventSourcing.html

### Design Principles & Laws

- **Conway's Law**: https://www.melconway.com/Home/Conways_Law.html
- **Law of Demeter**: https://en.wikipedia.org/wiki/Law_of_Demeter
- **DRY / KISS / YAGNI**: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself
- **Twelve-Factor App**: https://12factor.net/

### SDLC & Engineering Practices

- **Continuous Delivery (Jez Humble, Dave Farley)**: https://continuousdelivery.com/
- **Trunk-Based Development**: https://trunkbaseddevelopment.com/
- **GitHub Flow / Git Flow**: https://nvie.com/posts/a-successful-git-branching-model/
- **Semantic Versioning**: https://semver.org/
- **Conventional Commits**: https://www.conventionalcommits.org/

### Testing Principles

- **Test Pyramid (Martin Fowler)**: https://martinfowler.com/bliki/TestPyramid.html
- **Testing Trophy (Kent C. Dodds)**: https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
- **Property-Based Testing**: https://github.com/leanprover-community/mathlib4/blob/master/docs/contribute/property-based-testing.md

## Project Structure and Test Co-location

These sources are used only for the properties they actually support.
Framework documentation may be unopinionated about file layout; a style
guide from one ecosystem is not evidence for a universal rule. Use the
canonical applicability matrix in
`skills/implementation-recipes/SKILL.md` before making a finding.

### Primary Unit per File

- **React — Thinking in React**: https://react.dev/learn/thinking-in-react
  — component decomposition by responsibility; does **not** mandate one
  component per file.
- **Next.js — Project structure**: https://nextjs.org/docs/app/getting-started/project-structure
  — documents folder conventions; does **not** mandate one component per
  file or a barrel per folder.
- **Airbnb React/JSX Style Guide**: https://airbnb.io/javascript/react/
  — “Only include one React component per file”, with an exception for
  multiple stateless/pure components. This is an authoritative community
  style guide, not React official documentation.
- **ESLint `react/no-multi-comp`**: https://github.com/jsx-eslint/eslint-plugin-react/blob/master/docs/rules/no-multi-comp.md
  — enforces one component per file by default; can be disabled.
- **Angular — Style Guide**: https://angular.dev/style-guide
  — “One concept per file”; one component, directive, or service per file
  is usual, but small related components/directives may share a file when
  they form one concept. Unit tests live in the same directory as the
  code under test.
- **Vue — Single-File Components**: https://vuejs.org/guide/scaling-up/sfc.html
  — a `.vue` file encapsulates one Vue component (template, logic, and
  styles); the SFC format is inherently one component per file.
- **Svelte — `.svelte` files**: https://svelte.dev/docs/svelte/svelte-files
  — components are written into `.svelte` files; the file is the component.
- **Solid — Components basics**: https://docs.solidjs.com/concepts/components/basics
  — named exports allow multiple components to be exported from a single
  file; do **not** impose one component per file.
- **Java — JLS §7.6**: https://docs.oracle.com/javase/specs/jls/se21/html/jls-7.html#jls-7.6
  — at most one public top-level class/interface that is referenced by
  other compilation units per compilation unit; putting each class or
  interface in its own file is the common convention.
- **Kotlin — Coding conventions**: https://kotlinlang.org/docs/coding-conventions.html
  — placing multiple closely related declarations in one source file is
  encouraged; do **not** impose one class per file.
- **PHP — PSR-4 autoloading**: https://www.php-fig.org/psr/psr-4/
  — a fully qualified class name maps to a file path; autoloaded classes
  are one per file in practice. **PSR-12** (https://www.php-fig.org/psr/psr-12/)
  covers coding style, not a one-class-per-file rule beyond autoloading.
- **Python, Go, Rust, C#/.NET, Ruby**: no official one-class/type-per-file
  mandate. Apply only test organization and project conventions.

### Unit Folders and Test Placement

- **Angular — Style Guide**: https://angular.dev/style-guide
  — group closely related component files in the same directory; unit
  tests live with the code under test.
- **Java/Maven — Standard directory layout**: https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html
- **Gradle — Java plugin layout**: https://docs.gradle.org/current/userguide/java_plugin.html
- **Python/pytest — Test discovery and good practices**: https://docs.pytest.org/en/stable/explanation/goodpractices.html
- **Go — Add a test (package layout)**: https://go.dev/doc/tutorial/add-a-test
- **Rust — Test organization**: https://doc.rust-lang.org/book/ch11-03-test-organization.html
- **Cargo — Tests**: https://doc.rust-lang.org/cargo/guide/tests.html
- **.NET — Unit testing best practices**: https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- **PHPUnit — Organizing tests**: https://docs.phpunit.de/en/11.5/organizing-tests.html
- **RSpec — Core**: https://rspec.info/documentation/3.13/rspec-core/

### Test File Conventions

- **Vitest — Include patterns**: https://vitest.dev/config/#include
- **Jest — testMatch**: https://jestjs.io/docs/configuration#testmatch-arraystring
- **Playwright — Test configuration**: https://playwright.dev/docs/test-configuration

**Invariant:** group a unit's related artifacts in one folder/module and
co-locate its test when the ecosystem permits. Keep one primary unit per
file only where the ecosystem's documentation or authoritative style guide
supports it. The exact layout follows the official convention and the
project's coherent conventions; do not impose one universal template.

---

## Selection Workflow

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
