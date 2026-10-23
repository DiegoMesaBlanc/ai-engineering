# Official Documentation Index

## Purpose

Provide authoritative, version-aware documentation references for every
technology supported by the AI Engineering System.

This index contains URLs and selection guidance, not copies of the
documentation.

Consult the smallest relevant source on demand.

## Freshness

Documentation that is ten years old is worse than no documentation: it
teaches patterns the ecosystem has moved away from. Every source below is
classified by how it tracks versions.

- **Version-agnostic** sources follow the current release. They are the
  default choice, and they never expire. A version number in the path
  refers to a *major* line that is still current, such as `/3/` for Python 3.
- **Version-pinned** sources document one specific release. They are
  necessary only when the guidance differs between releases. Every pinned
  source is listed in **Version-pinned sources** with its version and its
  review rule.
- **Version guides** are the source of truth for what is current. Consult
  the guide of the ecosystem before its other pages.

Rules:

- Never cite a source without knowing whether it is current. When a
  version-pinned source is needed, register it in the pinned table.
- When the consulted source does not cover the target version, report it
  instead of applying older guidance silently.
- Record the version the consulted source documents, and the date of the
  consultation.
- A pinned source that is past its review rule is `NOT VERIFIED`, not
  silently applied.

## Version and Support

The **target version** of a language, framework, or runtime is a decision,
never an accident. When the stack is detected, consult the version guide of
that ecosystem below and propose the version the project should use, with its
support status. The source of truth is, in order, the source analysis, the
official documentation, then a stated default.

- React: https://react.dev/versions
- Angular: https://angular.dev/guide/versioning
- Next.js: https://nextjs.org/docs/app/guides/upgrading (upgrade guide per major version)
- Vue: https://vuejs.org/guide/scaling-up/tooling.html
- Svelte: https://svelte.dev/docs/svelte/v5-migration-guide
- Node.js: https://nodejs.org/en/about/previous-releases
- Python: https://www.python.org/downloads/
- Spring Boot: https://spring.io/projects/spring-boot#support
- Go: https://go.dev/doc/devel/release
- Rust: https://doc.rust-lang.org/edition-guide/
- .NET: https://learn.microsoft.com/en-us/dotnet/core/versions/
- PHP: https://www.php.net/releases/
- Ruby: https://www.ruby-lang.org/en/downloads/releases/

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
- Versions and release channels: https://react.dev/versions
- Learn: https://react.dev/learn
- API reference: https://react.dev/reference/react
- Rules of React: https://react.dev/reference/rules

### Angular

- Official documentation: https://angular.dev/docs
- API reference: https://angular.dev/api/
- Style guide: https://angular.dev/style-guide
- Agent Skills: https://angular.dev/ai/agent-skills
- Versions and release schedule: https://angular.dev/guide/versioning

Confirm the **target version** before using Signals, Signal Forms, SSR,
routing, or other version-sensitive APIs. The version guide is the source
for what is current and supported; the installed version is only what the
environment happens to have.

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

### Svelte

- Official documentation: https://svelte.dev/docs/svelte/overview
- SvelteKit: https://svelte.dev/docs/kit/introduction
- Runes and reactivity: https://svelte.dev/docs/svelte/what-are-runes
- Versions and the v5 migration guide: https://svelte.dev/docs/svelte/v5-migration-guide

### Solid

- Official documentation: https://docs.solidjs.com/
- Components basics: https://docs.solidjs.com/concepts/components/basics
- SolidStart: https://docs.solidjs.com/solid-start

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

### Kotlin

- Official documentation: https://kotlinlang.org/docs/home.html
- Coding conventions: https://kotlinlang.org/docs/coding-conventions.html
- Type aliases: https://kotlinlang.org/docs/type-aliases.html
- Spring Boot with Kotlin: https://docs.spring.io/spring-boot/reference/features/kotlin.html

### Go

- Official documentation: https://go.dev/doc/
- Language specification: https://go.dev/ref/spec
- Effective Go: https://go.dev/doc/effective_go
- Modules and versioning: https://go.dev/ref/mod#semver-versions
- Package layout and testing: https://go.dev/doc/tutorial/add-a-test

### Rust

- Official documentation: https://doc.rust-lang.org/book/
- Edition guide and toolchain: https://doc.rust-lang.org/edition-guide/
- Cargo workspaces: https://doc.rust-lang.org/cargo/reference/workspaces.html
- Test organization: https://doc.rust-lang.org/book/ch11-03-test-organization.html
- `pub use` and re-exports: https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#re-exporting-public-items-with-pub-use

### C# / .NET

- Official documentation: https://learn.microsoft.com/en-us/dotnet/
- .NET SDK and target frameworks: https://learn.microsoft.com/en-us/dotnet/core/versions/
- Project layout and tooling: https://learn.microsoft.com/en-us/dotnet/core/tools/
- StyleCop analyzers: https://github.com/StyleCop/StyleCop
- `TypeForwardedToAttribute`: https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.typeforwardedtoattribute

### PHP

- Official documentation: https://www.php.net/manual/en/
- PSR-4 autoloading: https://www.php-fig.org/psr/psr-4/
- PSR-12 coding style: https://www.php-fig.org/psr/psr-12/
- `class_alias()`: https://www.php.net/manual/en/function.class-alias.php

### Ruby

- Official documentation: https://docs.ruby-lang.org/en/master/
- RubyGems and gem versioning: https://guides.rubygems.org/specification-reference/
- Bundler: https://bundler.io/
- RSpec: https://rspec.info/documentation/latest/rspec-core/ (tracks the current release)

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
- `func-style` (arrow functions / no function declarations): https://eslint.org/docs/latest/rules/func-style
- `no-restricted-syntax` (forbid function expressions assigned to variables): https://eslint.org/docs/latest/rules/no-restricted-syntax
- Custom rules in flat config: https://eslint.org/docs/latest/extend/custom-rules
- Flat config configuration files: https://eslint.org/docs/latest/use/configure/configuration-files

### Contract Forwarding Mechanisms

Mechanisms that give a consumed contract a second public path, per ecosystem.

- **TypeScript / JavaScript**: `export … from` — https://typescriptlang.org/docs/handbook/modules/reference.html
- **React, Next.js, Angular, Vue, Svelte, Solid, React Native, Ionic, Capacitor, Node.js, Express, NestJS**: inherited from TypeScript and JavaScript
- **Go**: type alias — https://go.dev/ref/spec#Type_declarations
- **Rust**: `pub use` — https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#re-exporting-public-items-with-pub-use
- **Python, Django, FastAPI**: `__all__` re-export — https://docs.python.org/3/tutorial/modules.html#importing-all-names-from-a-module
- **Ruby**: constant assignment alias — https://docs.ruby-lang.org/en/master/syntax/literals_rdoc.html
- **Kotlin, Spring**: `typealias` — https://kotlinlang.org/docs/type-aliases.html
- **PHP**: `class_alias()` — https://www.php.net/manual/en/function.class-alias.php
- **C# / .NET**: `TypeForwardedToAttribute` — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.typeforwardedtoattribute
- **Java, Spring**: none. An import declaration makes a type available by its simple name only within the compilation unit that contains it — https://docs.oracle.com/javase/specs/jls/se21/html/jls-7.html#jls-7.5
- **SQL (PostgreSQL, MySQL, Oracle, MongoDB)**: none. There are no type or module declarations to forward.

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

- OWASP Dependency-Check: https://dependency-check.github.io/DependencyCheck/
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
  https://learn.microsoft.com/en-us/azure/devops/pipelines/security/overview

### Bitbucket Pipelines

- Official documentation:
  https://support.atlassian.com/bitbucket-cloud/docs/configure-bitbucket-pipelinesyml/

Detect the platform actually used by the project and consult the
corresponding documentation. CI/CD and delivery decisions follow
`workflows/secure-delivery.md`.

## Architecture, Design Patterns & Clean Code (Foundational)

These are the authoritative sources that inform our internal rules. Consult when making architectural decisions, evaluating patterns, or establishing conventions.

### Design Patterns

The pattern catalog and the selection procedure are defined in
`references/design-patterns.md`. These are the authoritative sources behind
the catalog; the implementation idiom is always taken from the detected
stack's official documentation.

- **Design Patterns: Elements of Reusable Object-Oriented Software**
  (Gamma, Helm, Johnson, Vlissides, 1994): ISBN 978-0201633610 — the GoF
  catalog.
- **Patterns of Enterprise Application Architecture** (Martin Fowler):
  https://martinfowler.com/books/eaa.html — persistence, domain, and
  layering patterns.
- **Enterprise Integration Patterns** (Hohpe, Woolf):
  https://www.enterpriseintegrationpatterns.com/ — messaging and
  integration patterns.
- **Refactoring catalog** (Martin Fowler):
  https://refactoring.com/catalog/ — the refactoring vocabulary used when a
  pattern replaces an existing structure.
- **Refactoring Guru — Design Patterns** (Spanish):
  https://refactoring.guru/es/design-patterns — practical catalog with
  intent, trade-offs, structure, and per-language examples. Consult it when
  explaining or implementing a selected pattern in the detected stack. It
  is a secondary reference, not a replacement for the GoF catalog; its
  public catalog lists 22 classic patterns and omits Interpreter, while the
  canonical catalog contains all 23 GoF patterns.
- **TypeScript Handbook** — generics, classes, modules, iterators:
  https://www.typescriptlang.org/docs/handbook/
- **MDN Web Docs** — `Set`, `EventTarget`, iterators:
  https://developer.mozilla.org/en-US/docs/Web/JavaScript

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
- **Ports and Adapters**: https://alistair.cockburn.us/hexagonal-architecture/

Every structural rule and folder tree **names its source** here, and the
kind of source is stated: an ecosystem official rule, a recognized community
methodology, or a policy of this system. A methodology is never presented as
a framework standard.

### Spring Boot and Java

- Structuring your code: https://docs.spring.io/spring-boot/reference/using/structuring-your-code.html
  — the official example groups a domain in one package (`customer/` with
  its controller, service, and repository), and points to Spring Modulith
  when the structure is to be enforced by domain.
- **Spring Modulith**: https://docs.spring.io/spring-modulith/reference/fundamentals.html
  — application modules, with a **provided interface** that other modules
  may use and **internal** components that are not supposed to be accessed
  from outside. This is the Java authority for a feature-driven module
  with a public API.
- Spring Modulith project structure: https://docs.spring.io/spring-modulith/reference/project-structure.html

### Layered Architecture

- Presentation, Domain, Data Layering (Martin Fowler):
  https://martinfowler.com/bliki/PresentationDomainDataLayering.html

### Feature-Sliced Design (Frontend Architecture)

A **community methodology**, not a framework standard. It is the authority
for the feature-driven tree and the public API per slice. No framework
prescribes `src/features`; the framework's own convention is followed where
it exists, and the feature-driven layout is used when the project adopts
this methodology.

- **Official Methodology**: https://feature-sliced.design/docs/get-started/overview
- **Layers & Slices**: https://feature-sliced.design/docs/reference/layers
  — layers are folders with lowercase names, such as `shared`, `pages`, and
  `app`; a project uses the layers that bring it value, and adding layers
  beyond the standardized ones is not recommended.
- **Public API**: https://feature-sliced.design/docs/reference/public-api
  — a public API is a contract between a group of modules and the code
  that uses it, it acts as a gate allowing access only through that API,
  and it is usually implemented as an index file with re-exports.

### Modular Monolith & Bounded Contexts

- **Modular Monolith (Kamil Grzybek)**: https://github.com/kgrzybek/modular-monolith-with-ddd
- **Modular Monolith vs Microservices**: https://github.com/dotnet-architecture/eShopOnWeb
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
- **Property-Based Testing**: https://github.com/leanprover-community/mathlib4/tree/master/docs

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

## Version-Pinned Sources

These sources document one specific release. They are listed so that a
stale one is visible instead of silently applied. The check in
`workflows/system-audit.md` requires every URL that contains a version
number to appear here.

| Source | Version | Why pinned | Review rule |
|---|---|---|---|
| https://docs.phpunit.de/en/11.5/organizing-tests.html | PHPUnit 11.5 | The test organization guide is version-specific and there is no `current` alias | Review when PHPUnit 12 becomes the default branch |
| https://nextjs.org/docs/app/guides/upgrading/version-16 | Next.js 16 | The upgrade guide is per major version | Review when Next.js 17 is released |
| https://rspec.info/documentation/3.13/rspec-core/ | RSpec 3.13 | The core documentation is version-specific | Prefer https://rspec.info/documentation/latest/rspec-core/, which tracks the current release |
