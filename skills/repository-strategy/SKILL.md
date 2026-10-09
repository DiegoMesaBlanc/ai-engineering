---
name: repository-strategy
description: Evaluate repository topology, workspaces, monorepos, and multi-repository organization based on actual project needs, dependency boundaries, team ownership, and delivery requirements.
---

# Repository Strategy

## Purpose

Determine the appropriate repository organization for a new or existing
software project.

Repository organization is a separate decision from application
architecture and frontend composition.

## Detect Existing Structure

Inspect relevant evidence such as:

- repository root directories
- package.json workspaces
- pnpm-workspace.yaml
- nx.json
- turbo.json
- lerna.json
- rush.json
- Maven multi-module configuration
- Gradle multi-project configuration
- solution files and project references
- CI/CD pipeline structure
- application and library boundaries

Do not infer a monorepo solely from directory names.

## Monorepo

Consider a monorepo when there is a concrete benefit from:

- coordinated changes across applications and shared packages
- shared libraries or types
- common development tooling
- consistent validation and dependency management
- cross-application changes in one commit
- centralized code ownership and visibility

Evaluate:

- package boundaries
- dependency direction
- affected-project detection
- build caching
- selective tests
- release coordination
- CI/CD cost
- repository permissions
- ownership boundaries

A monorepo is not automatically better because an organization has
multiple applications.

## Multiple Repositories

Consider separate repositories when justified by:

- independent ownership
- independent access control
- independent release lifecycles
- separate organizational boundaries
- minimal shared code
- operational constraints

Evaluate the cost of coordinating cross-repository changes.

## Tooling

For JavaScript and TypeScript, consider existing or justified tools such as:

- npm, pnpm, or Yarn workspaces
- Nx
- Turborepo

Do not introduce a workspace orchestrator unless it provides measurable
value for the actual project.

For other ecosystems, use the repository's established build and
workspace conventions.

## New Projects

Before choosing repository topology, understand:

- number of applications
- shared libraries
- expected team ownership
- independent deployment requirements
- dependency sharing
- CI/CD needs
- operational complexity

For a simple application, prefer the simplest viable repository structure.

## Existing Projects

Preserve the existing repository organization unless the task requires
a justified change.

Do not migrate to a monorepo or split repositories as incidental
refactoring.

## Output

Return:

### CURRENT REPOSITORY STRATEGY

...

### EVIDENCE

...

### OPTIONS

...

### RECOMMENDATION

...

### TRADE-OFFS

...

### MIGRATION COST AND RISKS

...

### HUMAN APPROVAL REQUIRED

...

## Safety

Never create a multi-repository or monorepo migration automatically.

Significant repository restructuring requires human approval.
