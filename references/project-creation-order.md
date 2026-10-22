# Project Creation Order

## Purpose

The ordered steps for creating a project from scratch, with what each step
produces, how to verify it, and the rule that tells the project where the
next thing goes.

The order is not cosmetic. Each step depends on the previous one, and each
step leaves a defined extension point. Creating the structure before the
tooling, or the adapters before the domain, produces a project that has to
be reworked as it grows.

Used by `workflows/project-initialization.md` Phase 4.

## Target version

Every project is created for a **confirmed target version** of its
language, framework, and runtime, settled in the startup decision set. The
creation order does not choose a version by accident, and it never uses the
version that happens to be installed in the environment. When the
environment does not have the target version, the difference is reported and
the checks that could not run against it are reported as `NOT VERIFIED`.

## Deciding the folders

The folder layout is **decided**, not left to chance. The tree comes from
`references/folder-architectures.md`, translated to the detected ecosystem.
Before the files of a feature are created, the system records, for each unit:

- the **owner**: the feature, or `shared/` when it already has a second
  consumer;
- the **folder**: `features/<feature>/<unit>/` for a unit with its own test
  or types, and `features/<feature>/` only for a single trivial file;
- the **public surface**: the `index.ts` of the feature, listing what other
  parts of the application may use.

A feature folder that ends up holding a flat pile of implementation files,
tests, and types mixed together is the symptom of skipping this decision.

## Scope Control

Before creating anything, agree the **catalog of what the project will
contain**:

- the modules or bounded contexts;
- the public surfaces, one vertical slice at a time;
- the dependencies, each with its reason.

The catalog is a **ceiling, not a floor**. The creation never exceeds it
without asking. A project that ends with five endpoints nobody discussed, or
twenty dependencies nobody requested, is a defect, not a generous reading of
the scope.

**Dependency budget**: a new project starts with the ecosystem minimum.
Every additional dependency is named, justified, and approved like any other
new dependency. A generated project must never add a library the user cannot
explain.

**One slice at a time**: the first vertical slice is complete before the
second one starts. Do not generate the whole surface area and let the user
discover it afterwards.

## Checkpoints

Every step is a **checkpoint**. The **stopping points** are these
checkpoints: the user decides where the creation ends. After a step is
verified, the creation stops and reports:

- what the step produced and how it was verified;
- the state of the project: the steps done, the next step, and the rest;
- the next suggested step, and exactly what it would add.

Then it **waits**. The user may continue, stop, or change the order.

**Before continuing, compare what exists against the scope catalog.** Any
endpoint, module, or dependency that is not in the catalog is reported and
asked about, never kept silently. This comparison is part of the
checkpoint, and it is the evidence that the creation stayed inside its
scope.

**Stopping is a normal outcome, not a failure.** The project is left exactly
at that checkpoint. The progress tracker records where it stopped, so the
next session resumes from there instead of restarting or, worse, generating
the rest.

### Progress tracker

Keep this state alongside the project and report it at every checkpoint.

| Step | State | Note |
|---|---|---|
| not started | `pending` | |
| created and verified | `done` | what it produced |
| the current point | `stopped here` | why the user stopped, when known |

Rules:

- A step is `done` only when it passed its verification.
- The tracker is the single record of what exists. It is updated before the
  next step starts, never at the end.
- A session that ends with a `stopped here` state is complete, not broken.
- The next session reads the tracker, confirms the repository still matches
  it, and continues from the next `pending` step.

## The growth rule

Every step states where the **next** thing goes. That is what makes a
project grow well instead of decaying:

- A new **business rule** goes in the domain layer, never in a route or a
  controller.
- A new **use case** goes in the application layer, next to its siblings.
- A new **external integration** gets a port in the domain and an adapter in
  infrastructure. The port is declared where it is needed, not where it is
  implemented.
- A new **endpoint** goes in the API layer of its feature, and only
  orchestrates.
- A new **field of a contract** changes the contract file and its consumers.
  It never changes an adapter.
- A new **cross-cutting concern** (logging, tracing, configuration, errors)
  has one home, and every layer uses it.

When a change does not fit the growth rule, the structure is wrong, not the
change.

---

## Part A — Universal Steps

Applies to every project type, in this order.

| # | Step | Produces | Verify | Growth rule |
|---|---|---|---|---|
| A1 | Repository skeleton | The top-level folders, the **entry point** of the project, the ignore file, the license, pinned to the **confirmed target version** | The tree matches the agreed structure, the entry point compiles, and the toolchain reports the confirmed version | A new top-level folder appears only when a new responsibility exists, never for a feature of an existing one |
| A2 | Tooling | Formatter, linter, structural compliance rules, test runner, scripts, **after the user approves the installation list** | The formatter check, the linter, and the test runner all run against the entry point from A1 and pass | Tooling changes in one place: the config and the scripts |
| A3 | Configuration | A typed configuration schema, a local environment example, no secret in the repository | The app starts with only the example file | Every new setting is declared in the schema, never read ad hoc |
| A4 | Observability baseline | Structured logging, a correlation id, the error shape | A request produces a log line with the correlation id | One logging and one error module; every layer uses them |
| A5 | Domain or core | The entities, value objects, invariants, domain errors | The core unit tests pass without any infrastructure | Business rules live here and nowhere else |
| A6 | Contracts | The public API types of each module, one per file, grouped by consumer | The contract files contain no runtime behavior | A contract is owned by the module that owns the concept |
| A7 | First vertical slice | One feature end to end, from the API to the persistence | The slice works against a real dependency, with a test | The slice is the template every next feature copies |
| A8 | Tests | Unit, integration, contract, and end-to-end layers, in proportion to risk | The whole suite passes | A test lives next to the unit it covers |
| A9 | CI | The pipeline that runs the checks | A pipeline run is green on the first commit | The pipeline is defined once, not per feature |
| A10 | Documentation | README with the stack, the architecture style, the commands, and the decisions | A new contributor can run the project from the README | Architecture decisions are recorded where the structure is described |

---

## Part B — Backend Creation Order

| # | Step | Produces | Verify | Growth rule |
|---|---|---|---|---|
| B1 | Repository and package skeleton | The module or package layout implied by the agreed architectural style, plus the **entry point** of the service, pinned to the **confirmed target version** | The tree matches the style that was chosen, the entry point compiles, and the toolchain reports the confirmed version | A new bounded context becomes a new module; a new feature inside a context becomes a folder |
| B2 | Tooling and scripts | Formatter, linter, structural compliance rules, test runner, type checker | The checks pass on an empty module | One config, one set of scripts |
| B3 | Configuration schema | Typed settings, environment schema, a local example, secrets documented but not committed | The service refuses to start with an invalid configuration | Every new setting is declared, validated, and documented in one place |
| B4 | Logging, correlation, and error model | Structured logger, request correlation id, the error response shape, the problem-details mapping | One failing request produces one error response and one correlated log | One error module; the API layer maps errors, the domain never knows HTTP |
| B5 | Domain layer | Entities, value objects, aggregates, domain errors, invariants | The domain unit tests pass with no database and no HTTP | Business rules live here only |
| B6 | Ports | The interfaces the domain needs: repositories, clocks, external services, event publishers | The domain compiles against the ports with no adapter implementation | A port is declared by its consumer, in the layer that needs it |
| B7 | Contracts | The API request and response types, the event schemas, one type per file, grouped by consumer | The contract files contain no runtime behavior and no framework types | A contract belongs to the module that owns the concept, not to the transport |
| B8 | Application layer | The use cases that orchestrate the domain through the ports | The use case tests pass with fakes for the ports | A use case never touches an adapter directly |
| B9 | Persistence | The connection, the repository implementation, the queries, the migrations | The migration applies to an empty database and the integration test passes | A new query goes in the repository implementation, not in a use case |
| B10 | API layer | The routes, the request validation, the mapping to contracts, the error mapping, the versioning | The endpoint returns the documented response for success and failure | A route orchestrates; it holds no business logic and no query |
| B11 | Identity and access | The authentication middleware, the authorization policy, the identity context | An unauthenticated and an unauthorized request behave as documented | Authorization decisions are policy objects, not conditions scattered in routes |
| B12 | Asynchronous work | The queue client, the job handlers, the retry and dead-letter policy, the event consumers | A job runs, retries, and lands in the dead-letter path on failure | A new consumer is a new module entry, not a branch in an existing handler |
| B13 | Integrations | One adapter per external system, behind its port, with a fake in tests | The integration test runs against a fake or a container | An external system is added as a port plus an adapter, never called directly from a use case |
| B14 | Caching | The cache client, the key strategy, the invalidation points | A cached read and its invalidation are covered by a test | A cache is added when a measured need appears, not by default |
| B15 | Observability | Metrics for the use cases and the endpoints, traces, health checks, graceful shutdown | The health endpoint reports the dependencies, and a termination drains in-flight requests | One instrumentation module, used by every layer |
| B16 | API documentation | The generated OpenAPI, validated in CI | The spec is regenerated and its diff is reviewed | The spec is generated, never written by hand |
| B17 | Tests | Unit for the domain, integration for the adapters, contract for the API, end to end for the critical paths | The whole suite passes, and the end-to-end path runs against real dependencies | A test lives next to the unit it covers |
| B18 | Containerization and runtime configuration | The multi-stage image, the runtime configuration, the health check wiring | The image builds and the service starts with only the runtime configuration | The image contains no development dependency and no secret |
| B19 | CI | The pipeline that builds, checks, tests, and produces the image | The first pipeline run is green | One pipeline definition |
| B20 | Local development environment | The compose file or run script, the seeded data, the documented bootstrap | A new contributor reaches a working service from the README alone | The local environment mirrors the production topology, not the code structure |
| B21 | Documentation and decisions | The README, the architecture decision records, the contribution guide | The README explains the stack, the style, the commands, and the local setup | A decision is recorded once, where it is explained |

---

## Part C — Other Project Types

### Frontend

| # | Step | Produces | Verify | Growth rule |
|---|---|---|---|---|
| C1 | Repository, tooling, and scripts | The app skeleton with its entry point, the formatter, the linter, the structural compliance rules, the test runner, pinned to the **confirmed target version** | Every check passes against the entry point, and the toolchain reports the confirmed version | A new page is a route, not a new build setup |
| C2 | Design tokens and the styling approach | The tokens, the global styles, the theme, the reset | The lint passes and the tokens are documented | A new color, spacing, or radius is a token, not a literal in a component |
| C3 | Application shell | The layout, the navigation, the providers, the error boundary, the loading boundary | The shell renders in every state, including the error state | A new provider is added at the shell, once |
| C4 | Data layer | The API client, the query client, the typed endpoints, the error mapping, the cache policy | A typed endpoint test proves the request and the error path | A new endpoint is a typed function in the data layer, never called from a component |
| C5 | Shared components | The primitives and the shared UI, with their states | The accessibility check of the component library passes | A component is shared only after a second consumer exists |
| C6 | Feature slices | One feature at a time, complete, with its route, its data, its states, and its tests. Each **unit** gets its own folder with the implementation, its test, and its types; the feature ends with an `index.ts` public API | The slice works end to end, the unit folders hold their own tests, and the public API exports only what is allowed | A feature is self-contained; it does not reach into another feature's internals |
| C7 | Forms and validation | The form solution, the schema validation, the error and submit states | The submit path and its validation errors are covered | A form owns its validation; a field does not validate itself |
| C8 | Localization | The message catalogs, the locale switching, the formatting helpers, the RTL handling, when the product has user-facing text | No hardcoded user-facing string remains, and the RTL layout is correct | Every user-facing string lives in a catalog, keyed by locale |
| C9 | Accessibility, SEO, and analytics | The audited behavior, the metadata, the tracking | The accessibility audit passes and the metadata is generated | Applied once, at the shell and the router |
| C10 | Performance budget | The measurable budgets and the CI check | The budget check fails when the budget is exceeded | Enforced in CI, not reviewed by eye |
| C11 | Tests | The unit, component, and end-to-end suites, in proportion to risk | The whole suite passes, and the critical paths run in a browser | A test lives next to the unit it covers |
| C12 | CI, deployment, and documentation | The pipeline, the deployment, the README, the recorded decisions | The first pipeline run is green and the README builds the app locally | The pipeline is defined once, not per feature |

### Library or CLI

| # | Step | Produces | Verify | Growth rule |
|---|---|---|---|---|
| D1 | Package skeleton and manifest | The package name, the entry points, the manifest, the license, plus a minimal entry file, pinned to the **confirmed target version** | The package builds and installs locally, with the confirmed version | The public API lives behind the entry points |
| D2 | Tooling, tests, and the CI pipeline | The formatter, the linter, the structural compliance rules, the test runner, the pipeline | The checks pass and the pipeline is green | One pipeline |
| D3 | Public API design | The exported types, the error surface, the deprecation policy | The exported surface is documented and type-checked | A public type is added with a deprecation path, never removed silently |
| D4 | Core implementation | The logic, free of framework and I/O assumptions | The core tests pass with no framework and no I/O | The core stays reusable, so the framework is an adapter |
| D5 | CLI or integration surface | The command layer, the argument parsing, the exit codes, the output format | The CLI runs, and its exit codes and output are asserted | The CLI is a thin adapter over the core |
| D6 | Documentation and distribution | The generated reference, the package publication, the release process | The reference is generated from the code, and a dry-run publish succeeds | The documentation is generated, never written by hand |
| D7 | Compatibility and support policy | The supported runtime versions, the compatibility matrix, the support window | The compatibility matrix is tested in CI | A support window is stated, so a version can be dropped deliberately |

### Mobile

| # | Step | Produces | Verify | Growth rule |
|---|---|---|---|---|
| E1 | Project, tooling, and build pipeline | The app skeleton with its entry point, the formatter, the linter, the structural compliance rules, the test runner, pinned to the **confirmed target version** | Every check passes against the entry point, and the toolchain reports the confirmed version | A new screen is a route, not a new setup |
| E2 | Identity and signing | The bundle or package identifier, the signing keys, the version scheme, the build flavors | A release build is signed and installs | The identifier and the signing keys change once, at the start |
| E3 | Navigation and shell | The navigator, the design system, the providers, the deep link routing | The shell renders in every state | The shell owns the shared providers and the deep link handling |
| E4 | Permissions and privacy | The declared permissions, the privacy manifest, the runtime permission requests, the rationale | The permission request flow is exercised, and the declaration matches the code | A permission is added only with a user-facing reason, declared once |
| E5 | Data and local storage | The API client, the cache, the secure store, the offline behavior | A read works offline, and the secure store holds only credentials | The secure store holds credentials only; everything else is cache |
| E6 | Feature screens | One screen at a time, with its state, its loading and error states, and its tests | The screen works end to end | A screen owns its state; shared state moves to the store only when a second screen needs it |
| E7 | Platform integrations | Push, analytics, crash reporting, payments, deep links, each one isolated | Each integration has a test and degrades when unavailable | An integration is one module with one owner, never called from a screen directly |
| E8 | Background work and synchronization | The background task, the retry policy, the sync conflict resolution | A sync runs with no connectivity and resumes correctly | Sync lives in one place, not in each screen |
| E9 | Accessibility and localization | The audited behavior, the catalogs, the right-to-left layouts, the dynamic type scaling | The accessibility audit passes and the text scales | Every user-facing string lives in a catalog, keyed by locale |
| E10 | Device and OS support matrix | The minimum and target OS versions, the device assumptions, the fallback behavior | The app runs on the minimum supported version | The support matrix is stated, so a device can be dropped deliberately |
| E11 | Tests | The unit, widget, and end-to-end suites, with the end-to-end path on a device or emulator | The whole suite passes | A test lives next to the unit it covers |
| E12 | CI | The pipeline that builds, tests, and produces the signed artifacts | The first pipeline run produces an installable artifact | The pipeline is defined once |
| E13 | Store release | The assets, the listing, the privacy policy, the staged rollout, the release notes | The store pre-submission checklist passes | Release is a pipeline step, not a manual action |
| E14 | Documentation and decisions | The README, the architecture decision records, the contribution guide | A new contributor builds the app from the README | A decision is recorded once, where it is explained |

## Part D — The Creation Checklist and Progress Tracker

This is both the checklist and the progress tracker. Copy it into the
initialization report, mark each item as it is completed, and keep it
updated at every checkpoint. Every `pending` item is an unfinished project,
not a detail.

States: `pending` → `done` (created and verified), or `stopped here` when
the user ends the session at that point.

### Analysis and decisions

- [ ] The source analysis was read completely, and its requirements were reused rather than re-derived
- [ ] The stack was confirmed against the official documentation
- [ ] Every startup decision was resolved: answered by the analysis, governed by a convention, defaulted and stated, or asked
- [ ] The user was asked once, grouped by theme, only for the decisions without a defensible default
- [ ] The architectural style was asked, and answered
- [ ] The rejected alternatives are recorded

### Universal steps

- [ ] A1 Repository skeleton, ignore file, and license
- [ ] A2 Formatter, linter, structural compliance rules, test runner, and scripts, all passing
- [ ] A3 Typed configuration schema and a local environment example, with no committed secret
- [ ] A4 Structured logging, correlation id, and the error shape
- [ ] A5 Domain or core, with its unit tests passing without infrastructure
- [ ] A6 Contracts, one type per file, grouped by consumer, with no runtime behavior
- [ ] A7 First vertical slice, working end to end
- [ ] A8 Test layers in proportion to risk
- [ ] A9 CI pipeline, green on the first run
- [ ] A10 README with the stack, the style, the commands, and the decisions

### Backend steps

- [ ] B1 Repository and package skeleton matching the agreed style
- [ ] B2 Tooling and scripts
- [ ] B3 Configuration schema, validated at startup
- [ ] B4 Logging, correlation, and the error model
- [ ] B5 Domain layer with its invariants and domain errors
- [ ] B6 Ports, declared by their consumers
- [ ] B7 Contracts, one type per file
- [ ] B8 Application layer, using only the ports
- [ ] B9 Persistence, with migrations that apply to an empty database
- [ ] B10 API layer, with validation, mapping, error handling, and versioning
- [ ] B11 Identity and access
- [ ] B12 Asynchronous work, with retry and dead-letter behavior
- [ ] B13 Integrations, one adapter per external system, behind a port
- [ ] B14 Caching, only with a measured need
- [ ] B15 Observability, health checks, and graceful shutdown
- [ ] B16 Generated API documentation, validated in CI
- [ ] B17 Test layers, with the end-to-end path against real dependencies
- [ ] B18 Container image without development dependencies or secrets
- [ ] B19 CI pipeline
- [ ] B20 Local development environment, reproducible from the README
- [ ] B21 Documentation and recorded decisions

### Frontend steps

- [ ] C1 Repository, tooling, and scripts, all checks passing
- [ ] C2 Design tokens and the styling approach documented
- [ ] C3 Application shell with the navigation, the providers, and the error and loading boundaries
- [ ] C4 Data layer with the typed endpoints, the error mapping, and the cache policy
- [ ] C5 Shared components with their states and their accessibility check
- [ ] C6 Feature slices, complete and self-contained
- [ ] C7 Forms and validation with the submit and error states
- [ ] C8 Localization with the catalogs, the locale switching, and the RTL layout
- [ ] C9 Accessibility, SEO, and analytics
- [ ] C10 Performance budget enforced in CI
- [ ] C11 Test layers, with the critical paths running in a browser
- [ ] C12 CI, deployment, and documentation

### Library or CLI steps

- [ ] D1 Package skeleton, manifest, entry points, and license
- [ ] D2 Tooling, tests, and the CI pipeline
- [ ] D3 Public API design with the exported types, the error surface, and the deprecation policy
- [ ] D4 Core implementation, free of framework and I/O assumptions
- [ ] D5 CLI or integration surface with the argument parsing, the exit codes, and the output format
- [ ] D6 Documentation and distribution
- [ ] D7 Compatibility and support policy tested in CI

### Mobile steps

- [ ] E1 Project, tooling, and build pipeline
- [ ] E2 Identity and signing, with a signed release build that installs
- [ ] E3 Navigation, shell, and deep link routing
- [ ] E4 Permissions and privacy, with the declaration matching the code
- [ ] E5 Data and local storage, with offline behavior and the secure store
- [ ] E6 Feature screens, complete and self-contained
- [ ] E7 Platform integrations, each isolated and degradable
- [ ] E8 Background work and synchronization, with the conflict resolution
- [ ] E9 Accessibility and localization, with the dynamic type scaling
- [ ] E10 Device and OS support matrix
- [ ] E11 Test layers, with the end-to-end path on a device or emulator
- [ ] E12 CI producing the signed artifacts
- [ ] E13 Store release: assets, listing, privacy policy, staged rollout
- [ ] E14 Documentation and recorded decisions

### Growth check

- [ ] Every new business rule has one home, and it is not the API layer
- [ ] Every new external integration has a port and an adapter, and is not called directly
- [ ] Every new setting is declared in the configuration schema
- [ ] Every new contract type has a single owning module
- [ ] The structure has an extension point for the next feature, and it is not a new top-level folder
