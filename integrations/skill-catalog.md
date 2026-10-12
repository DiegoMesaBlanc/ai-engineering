# External Skill Catalog

## Purpose

Evaluate reusable external Skills across every supported technology.

This is a catalog of decisions and candidates, not an installation list.

Only Skills approved for use should be imported into the canonical
skills/ directory and recorded in external-skill-registry.md.

## Status Definitions

- INSTALLED: imported into the canonical repository and verified.
- CANDIDATE: potentially useful; needs compatibility, license, and content
  review before installation.
- TASK_SPECIFIC: useful only for a narrow operation such as a migration.
- DOCS_ONLY: official documentation is sufficient for the current needs.
- REJECTED: not suitable for the Engineering System's requirements.
- REASSESS: revisit when the technology or task scope changes.

## Evaluation Criteria

For each Skill evaluate:

1. Source ownership and maintenance.
2. License and redistribution permissions.
3. Compatibility with the Agent Skills format.
4. Compatibility with OpenCode and other intended clients.
5. Compatibility with the detected language and framework version.
6. Support for the project's existing tools.
7. Additional value beyond official documentation.
8. Overlap with existing internal or external Skills.
9. Token cost and unnecessary context loading.
10. Conflicts with AGENTS.md, workspace safety, or human authorization.
11. Required supporting files, tools, dependencies, or permissions.
12. Reproducibility using a recorded upstream commit SHA.

Do not import a Skill until its license and required supporting files
have been verified.

## Installed Skills

### frontend-design

- Source: https://github.com/anthropics/skills
- Source directory: skills/frontend-design/
- License: Apache-2.0
- Status: INSTALLED
- Scope: visual design and frontend UI implementation
- Activation: creating or improving UI
- Constraints:
  - respect supplied designs
  - preserve existing design systems
  - avoid unrelated redesigns
  - reuse existing components when appropriate

### test-driven-development

- Source: https://github.com/obra/superpowers
- Source directory: skills/test-driven-development/
- License: MIT
- Status: INSTALLED
- Scope: behavior-changing implementation and testing
- Activation: when meaningful tests can protect the required behavior
- Constraints:
  - language-neutral applicability
  - use existing test frameworks
  - preserve existing implementation and user work
  - do not mandate irrelevant tests
  - validate proportionally to risk
  - record local modifications

## Languages

### TypeScript

- Documentation: https://www.typescriptlang.org/docs/
- Status: DOCS_ONLY
- Decision: Use the official documentation and existing project tooling.

### JavaScript

- Documentation: https://developer.mozilla.org/docs/Web/JavaScript
- Status: DOCS_ONLY
- Decision: Use runtime-specific documentation when relevant.

## Frontend Technologies

### React

- Documentation: https://react.dev/
- Candidate source:
  https://github.com/reactjs/react.dev/tree/main/.claude/skills
- Current status: DOCS_ONLY for general application development
- Rationale:
  Investigated React documentation Skills are oriented toward researching
  or maintaining React documentation, not general application coding.
- Decision:
  Do not import them as a general React implementation Skill.

### Angular

- Documentation: https://angular.dev/docs
- Official Skills: https://github.com/angular/skills
- Candidate Skill: angular-developer
- Candidate Skill: angular-new-app
- License declared by angular-developer: MIT
- Status: CANDIDATE

Evaluate angular-developer for Angular-specific implementation guidance.

Evaluate angular-new-app only when creating a new Angular project.

Before import:

- verify the exact upstream commit
- verify the applicable license and supporting files
- adapt version-specific recommendations to the project's installed Angular
  version
- preserve existing project conventions
- prevent the Skill from forcing unnecessary dependencies or migrations

### Vue

- Documentation: https://vuejs.org/guide/
- Community candidate: https://github.com/vuejs-ai/skills
- License: MIT
- Status: CANDIDATE

The community project describes itself as an early experiment.

Do not treat it as an official Vue team Skill.

Evaluate individual Vue 3, Composition API, TypeScript, testing, or
Vue Router guidance only when a real project requires it.

Do not import the entire repository by default.

### Next.js

- Documentation: https://nextjs.org/docs
- AI coding agent guidance:
  https://nextjs.org/docs/app/guides/ai-agents
- Status: DOCS_ONLY

Do not create or install a dedicated Next.js Skill by default.

Use stack-detection to identify the installed version and router.

Consult current version-compatible documentation when needed.

Reassess only if a specific workflow provides meaningful value beyond
version-aware documentation and existing Skills.

## Backend Technologies

### Node.js

- Documentation: https://nodejs.org/docs/
- Status: DOCS_ONLY

Use runtime documentation and the detected project's existing framework
guidance.

Evaluate framework-specific Skills only when a concrete need emerges.

### Express

- Documentation: https://expressjs.com/en/
- Status: DOCS_ONLY

### NestJS

- Documentation: https://docs.nestjs.com/
- Status: DOCS_ONLY

Use the project's existing TypeScript, testing, validation, and
architectural conventions.

### Python

- Documentation: https://docs.python.org/3/
- Status: DOCS_ONLY

Evaluate a framework-specific candidate when the repository uses
Django, FastAPI, Flask, or another identifiable framework.

Do not adopt repository-specific Skills merely because they contain
Python examples.

### Django

- Documentation: https://docs.djangoproject.com/
- Status: DOCS_ONLY

### FastAPI

- Documentation: https://fastapi.tiangolo.com/
- Status: DOCS_ONLY

### Java

- Documentation: https://docs.oracle.com/en/java/
- Status: DOCS_ONLY

Evaluate framework-specific candidates only when justified by the actual
Java project.

### Spring

- Documentation: https://docs.spring.io/spring-framework/reference/
- Status: DOCS_ONLY

## Mobile Technologies

### React Native

- Documentation: https://reactnative.dev/docs/getting-started
- Community Skills:
  https://github.com/react-native-community/skills
- License: MIT
- Status: TASK_SPECIFIC

Potentially useful for:

- React Native version upgrades
- Strict TypeScript API migrations

Do not load migration Skills during routine feature development.

### Ionic and Capacitor

- Ionic: https://ionicframework.com/docs
- Capacitor: https://capacitorjs.com/docs
- Status: DOCS_ONLY

Use the appropriate Angular, React, or Vue documentation in conjunction
with Ionic and Capacitor documentation.

## Database Technologies

### PostgreSQL

- Documentation: https://www.postgresql.org/docs/
- Status: DOCS_ONLY

### MySQL

- Documentation: https://dev.mysql.com/doc/
- Status: DOCS_ONLY

### Oracle Database

- Documentation: https://docs.oracle.com/en/database/
- Status: DOCS_ONLY

### MongoDB

- Documentation: https://www.mongodb.com/docs/
- Status: DOCS_ONLY

Database-specific Skills are not required by default.

Prefer the database's official documentation, the existing driver or ORM,
the project's migration tools, and its testing conventions.

## Testing and Engineering Tools

- Vitest: https://vitest.dev/guide/
- Jest: https://jestjs.io/docs/getting-started
- Playwright: https://playwright.dev/docs/intro
- Cypress: https://docs.cypress.io/
- Testing Library: https://testing-library.com/docs/

Use the installed tools.

Do not import a Skill or install a dependency solely to repeat the
test framework's documentation.

## UI, Styling, and Components

### Styling and UI Libraries

- Tailwind CSS: https://tailwindcss.com/docs
- Material UI: https://mui.com/material-ui/getting-started/
- PrimeNG: https://primeng.org/
- Status: DOCS_ONLY
- Decision: Use the UI library already present in the project.

## Linting and Formatting

- ESLint: https://eslint.org/docs/latest/
- Prettier: https://prettier.io/docs/
- Angular ESLint: https://github.com/angular-eslint/angular-eslint
- Ruff: https://docs.astral.sh/ruff/
- google-java-format: https://github.com/google/google-java-format
- Checkstyle: https://checkstyle.org/
- SQLFluff: https://sqlfluff.com/
- Status: DOCS_ONLY
- Decision: Follow the installed version and existing configuration.
  Configure the formatter and linter required by the Formatting and
  Linting Policy in AGENTS.md using the tools appropriate to the detected
  stack. Do not force Prettier onto file types it does not support.

## Security

- OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/
- Status: DOCS_ONLY
- Decision: Retrieve relevant guidance based on the application's threat model.

## Reassessment Rules

Reassess a candidate when:

- the project uses the relevant technology
- the task falls within the Skill's actual scope
- the Skill is actively maintained
- its license has been verified
- it provides meaningful benefits beyond documentation
- it complies with Engineering System policies

Do not import all candidates at once.

Do not create one Skill per language or framework merely for symmetry.

Record every installed Skill in external-skill-registry.md.

Run system validation after installing or updating a Skill.
