# Folder Architectures

## Purpose

The concrete folder tree of each architectural style, per ecosystem, and the
decisions that produce it.

This reference is the source of truth for **where the folders go**. The
styles are listed in `skills/architecture-advisor/SKILL.md`; the trees they
produce are here, so the answer to "what would this look like on disk" is
never a guess.

## The folder structure is a decision

The folder structure is **decided and asked**, not derived silently from the
architectural style. Two projects can share a style and still differ in
their folders, and the difference is worth one question.

The decision is part of the startup decision set. It is asked together with
the architectural style, and both are settled before any file is created.

## Authority of each tree

Every tree below **names its source**, and the kind of source differs.
The difference matters:

- **Ecosystem official rule** — the framework or language prescribes it.
  Angular's "one concept per file", Vue's single-file component, Go's
  package per directory, PSR-4's class per file, the Java Language
  Specification on public types.
- **Recognized community methodology** — a published method that a project
  adopts deliberately. Feature-Sliced Design for the feature-driven tree,
  Cockburn's Ports and Adapters for the hexagonal tree, Fowler's
  Presentation-Domain-Data Layering for the layered tree, Spring Modulith
  for a feature-driven Java module.
- **System policy** — a rule of this system, not of any ecosystem. Arrow
  functions, one type per file, and no pass-through re-export are the
  system's own policy and apply wherever the ecosystem does not contradict
  them.

A tree is never presented as an ecosystem standard when it is not. When the
detected framework prescribes its own structure, that structure wins and the
methodology is applied inside it.

## Canonical Matrix

This matrix is the machine-checkable form of the trees below. A checker
derives the expected folder layout from it, so the trees and the checks
cannot drift apart.

| Style | Top-level folders | Public API of a module | Core dependency direction |
|---|---|---|---|
| Layered | `api/`, `services/`, `domain/`, `infrastructure/` | none required | `infrastructure` → `services` → `domain`; `domain` depends on nothing |
| Feature-driven | `app/`, `shared/`, `features/<feature>/` | `features/<feature>/index.ts` (or the ecosystem equivalent) | a feature → `app` and `shared`; never another feature's internals |
| Hexagonal | `domain/`, `application/`, `adapters/inbound/`, `adapters/outbound/` | `domain/ports/` holds the ports | `adapters` → `application` → `domain`; `domain` depends on nothing |
| DDD | `domain/<context>/`, `application/`, `infrastructure/`, `interfaces/` | `domain/<context>/` exports the model | `interfaces` → `application` → `domain`; `domain` depends on nothing |
| Modular Monolith | `modules/<context>/`, `shared/` | `modules/<context>/index.ts` | a context → `shared`; never another context's internals |
| Event-driven | `producers/`, `consumers/`, `contracts/`, `messaging/` | `contracts/` holds the schemas | `producers` and `consumers` → `contracts`; `contracts` depend on nothing |

The folder names are translated to the ecosystem by the rules in
**Per-ecosystem translation** below; the dependency direction is the
invariant every implementation of that style must satisfy.

**How this matrix is verified.** A checker derives the expected layout from
this table, so the trees and the checks cannot drift apart. For every
combination of style and ecosystem it verifies that the folders of the style
exist, that a unit owns its implementation and keeps its test where the
ecosystem places it, that the module exposes its public API, and that the
dependency direction of the style holds. Where the ecosystem has a toolchain
available, the generated project is compiled, built, imported, or linted. The
matrix covers the six styles and the ecosystems of the per-ecosystem
translation table below; the ecosystems without a local toolchain are
verified structurally and reported as not verified for execution, never as
executed.

## Rules that hold in every tree

- **One folder per unit inside its responsibility folder.** A UI unit gets
  `components/<unit>/`, a behavioral unit gets `hooks/<unit>/` when it has a
  test or several files, and a service gets `services/<unit>/`. The unit
  folder holds the implementation and, where the ecosystem co-locates them,
  its test and its unit-local contract. A feature-level contract lives in
  `types/`. A feature root never holds a flat pile of implementations,
  tests, and types mixed together, and a responsibility folder never mixes
  components with contracts. The test joins the unit folder **only where the
  ecosystem co-locates tests**: TypeScript and JavaScript, Angular, Vue, Go,
  and Python. Where the ecosystem mirrors the test tree — Java, Kotlin,
  C#/.NET, PHP, and Ruby — the test belongs to that mirrored tree, not to the
  unit folder. A test case is never placed at the root of the source tree or
  of a layer folder, and shared test support lives in an explicit support
  location (`testing/`, `test/support/`, `spec/support/`, `conftest.py`,
  `src/testFixtures`, the test project) that never contains test cases.
- **One public API per module.** `index.ts` (or the ecosystem equivalent)
  exports what other modules may use; the rest stays encapsulated.
- **The dependency direction is downward.** A layer imports the layer
  below it, never the layer above.
- **Contracts live with their owner.** Types and interfaces live in the
  folder of the layer or module that owns them.

---

Each style is a decision with trade-offs; those live in
`skills/architecture-advisor/SKILL.md` ("Architectural style
alternatives"). This reference holds only the **trees** each style produces.

## Style 1 — Layered

**Authority**: Presentation-Domain-Data Layering (Martin Fowler) —
https://martinfowler.com/bliki/PresentationDomainDataLayering.html

```
src/
├── api/                 # controllers, routes, request handling
├── services/            # application services, use cases
├── domain/              # entities, value objects, business rules
└── infrastructure/      # database, HTTP clients, queues, files
```

```
src/
├── controller/          # web layer
├── service/             # application layer
├── domain/              # business layer
└── infra/               # persistence and integrations
```

## Style 2 — Feature-driven

**Authority**: Feature-Sliced Design —
https://feature-sliced.design/docs/get-started/overview and
https://feature-sliced.design/docs/reference/public-api
(a community methodology, adopted deliberately)

**Authority of the internal folders.** Feature-Sliced Design defines the
layers, the slice, and the public API; it does not prescribe the folders
inside a slice. The responsibility folders below are a **policy of this
system**: they make the owner of every artifact explicit, so a component and
a contract never share a folder. Each ecosystem translates them to its own
idiom in **Feature responsibility folders per ecosystem** below.

```
src/
├── app/                 # global configuration: providers, routes, state
├── shared/              # generic and reusable, bound to no capability
└── features/
    └── <feature>/
        ├── components/  # UI units owned by the feature
        │   └── <unit>/  # a unit with a test, styles, or subcomponents
        │       ├── <unit>.tsx
        │       ├── <unit>.test.tsx
        │       └── <unit>.module.css
        ├── hooks/       # behavioral units, one per file with its test
        ├── services/    # API and transport calls of the feature
        ├── types/       # feature-level contracts, one type per file
        ├── context/     # feature-local state, only when the feature owns it
        └── index.ts     # the public API of the feature
```

**The feature root holds no source file.** Only the responsibility folders
and the public API live at `features/<feature>/`. A component, a hook, a
service, or a contract at the feature root is a misplaced artifact: it
belongs to the folder that owns its responsibility. `features/<feature>/`
is not a folder for "the feature's files"; it is the boundary that contains
the folders.

**Each responsibility has one folder.** The artifact's kind decides the
folder, not the file's proximity to the component:

| Artifact | Folder | Example |
|---|---|---|
| UI component | `components/` | `components/payment-form/payment-form.tsx` |
| Behavioral unit | `hooks/` | `hooks/use-payment.ts` |
| API or transport call | `services/` | `services/payment-api.ts` |
| Feature-level contract | `types/` | `types/payment.types.ts` |
| Local state or context | `context/` | `context/payment-context.tsx` |
| Public API | feature root | `index.ts` |

**A contract follows its consumer group.** A contract consumed by more than
one unit of the feature lives in `types/`. A contract consumed by exactly
one unit lives in that unit's folder as `<unit>.types.ts`, following the
consumer-group rule of `TYPE-002`. A contract file never sits at the feature
root and never sits directly in `components/`, `hooks/`, or `services/`.
A component and the types it does not own are never mixed in one folder.

**The golden dependency rule.** A feature imports from `app/` and
`shared/`, and never the internals of another feature. Cross-feature
interaction goes through the other feature's public API, or through the
global state in `app/`. A unit that needs a second consumer is **promoted**
to `shared/`, in the folder that matches its kind; it is not copied.

## Style 3 — Hexagonal / Ports and Adapters

**Authority**: Alistair Cockburn's original description —
https://alistair.cockburn.us/hexagonal-architecture/

```
src/
├── domain/              # entities, value objects, business rules
│   └── ports/           # the interfaces the domain needs
├── application/         # use cases that drive the domain through the ports
└── adapters/
    ├── inbound/         # HTTP, CLI, message consumers
    └── outbound/        # database, external APIs, queues
```

**The dependency rule.** `adapters/` depend on `domain/`. `domain/` depends
on nothing. The port lives with the domain; the adapter implements it and
lives on the other side.

### Clean Architecture mapping

Clean Architecture (Robert C. Martin) and Hexagonal express the same
dependency rule with different vocabulary. Either name selects the same
tree; the layers map directly:

| Clean Architecture | Hexagonal equivalent |
|---|---|
| Entities | `domain/entities/` |
| Use cases | `application/use-cases/` (or `domain/use-cases/`) |
| Interface adapters | `adapters/inbound/` and `adapters/outbound/` |
| Frameworks and drivers | `infrastructure/` and `config/` |

The **use-case layer** is Clean's addition; Hexagonal calls it application.
Neither prescribes classes or functions: the unit is a class when the
ecosystem's idiom or DI container requires it, and a function or factory
otherwise. The canonical unit-style table is in
`skills/implementation-recipes/SKILL.md` ("Unit style: functions or
classes"). Both are application architectures: a frontend keeps its
framework organization and may apply them only to its core.

### Hexagonal / Clean, TypeScript and Node.js

```
src/
├── domain/
│   ├── entities/
│   ├── value-objects/
│   └── ports/
├── application/
│   └── use-cases/
├── adapters/
│   ├── inbound/http/
│   └── outbound/persistence/
├── infrastructure/
│   └── config/
├── testing/              # test support, never test cases
└── main.ts               # composition root: wires ports to adapters
```

The test lives next to the unit it tests. `main.ts` is the only place that
constructs concrete adapters; domain and application receive their
dependencies as parameters.

### Hexagonal / Clean, per ecosystem

| Ecosystem | domain | application | inbound adapter | outbound adapter | composition root | tests |
|---|---|---|---|---|---|---|
| TypeScript / Node.js | `src/domain` | `src/application` | `src/adapters/inbound` | `src/adapters/outbound` | `src/main.ts` | next to the unit |
| NestJS | `src/domain` | `src/application` | controllers | providers | Nest module wiring | `*.spec.ts` next to the unit |
| Angular | `src/app/core/domain` | `src/app/core/application` | components/guards | services/adapters | `app.config.ts` | `*.spec.ts` next to the unit |
| Java / Kotlin | `domain/` package | `application/` | `api/` | `infrastructure/` | Spring configuration | mirrored `src/test` |
| C# / .NET | `Domain/` | `Application/` | `Api/` | `Infrastructure/` | `Program.cs` | `tests/` project |
| PHP | `src/Domain` | `src/Application` | `src/Api` | `src/Infrastructure` | `public/index.php` | `tests/` |
| Ruby | `lib/domain` | `lib/application` | `lib/api` | `lib/infrastructure` | `config.ru` | `spec/` |
| Go | `internal/domain` | `internal/application` | `internal/adapters/http` | `internal/adapters/postgres` | `cmd/<app>/main.go` | same package |
| Rust | `src/domain` | `src/application` | `src/adapters/inbound` | `src/adapters/outbound` | `src/main.rs` or `lib.rs` | inline or `tests/` |
| Python | `src/domain` | `src/application` | `src/adapters/inbound` | `src/adapters/outbound` | `src/main.py` | next to the unit or `tests/` |

## Style 4 — DDD

**Authority**: Domain-Driven Design, with the bounded context of
https://martinfowler.com/bliki/BoundedContext.html

```
src/
├── domain/              # aggregates, entities, value objects, domain events
│   └── <context>/       # one folder per bounded context
├── application/         # commands, queries, use cases
├── infrastructure/      # persistence, messaging, external services
└── interfaces/          # HTTP controllers, event handlers, CLI
```

## Style 5 — Modular Monolith

**Authority**: Spring Modulith for Java —
https://docs.spring.io/spring-modulith/reference/fundamentals.html
(application modules with a provided interface and internal components)

```
src/
├── modules/
│   └── <context>/
│       ├── domain/
│       ├── application/
│       ├── api/
│       └── index.ts     # the public API of the context
└── shared/              # genuinely shared kernel
```

**A context never imports another context's internals.** It uses the other
context's public API.

## Style 6 — Event-driven

**Authority**: event-driven architecture guidance; the contracts are the
schema registry of the chosen broker.

```
src/
├── producers/           # commands that publish events
├── consumers/           # handlers that react to events
├── contracts/           # the event schemas, one type per file
└── messaging/           # the broker client, serialization, retry policy
```

---

## Per-ecosystem translation

The unit of organization changes with the ecosystem. The tree keeps its
shape; the names change.

| Ecosystem | Unit of organization | Public API of a module | Unit folder | Test location | Source of the rule |
|---|---|---|---|---|---|
| TypeScript, JavaScript, React, Next.js | folder | `index.ts` | folder | next to the unit | Vitest include convention; no framework prescribes `src/features` |
| Angular | folder | `index.ts` | folder | next to the unit, `*.spec.ts` | Angular Style Guide, "One concept per file" |
| Vue | folder | `index.ts` | folder | next to the unit | Vue Single-File Component specification |
| Svelte | folder | `index.ts` | folder | next to the unit | Svelte, one component per `.svelte` file |
| Solid | folder | `index.ts` | folder | next to the unit | Solid, named exports allow multiple components per file |
| React Native | folder | `index.ts` | folder | next to the unit | React Native + Jest preset; no layout mandate |
| Ionic (Angular) | folder | `index.ts` | folder | next to the unit, `*.spec.ts` | Angular Style Guide; Ionic adds no layout of its own |
| Ionic (React) | folder | `index.ts` | folder | next to the unit | React conventions; Ionic adds no layout of its own |
| Java, Kotlin | package | package-level public API | package folder | mirrored test tree | Spring Modulith, application modules with a provided interface; JLS on public types |
| Python | package | `__init__.py` | package folder | next to the unit, `test_*.py` | Python packaging; no one-class-per-file rule |
| Go | package | the package's exported identifiers | package folder | next to the unit, `_test.go` | Go specification, a package is a directory |
| Rust | module or crate | `pub use` in `mod.rs` or `lib.rs` | module folder | inline module or `tests/` | The Rust Programming Language, module files |
| C# / .NET | project or folder | `public` types | folder | mirrored test project | StyleCop `SA1402`; no framework rule |
| PHP | namespace | the namespace's public classes | folder | mirrored tests directory | PSR-4, one class per file |
| Ruby | engine or module | the module's public methods | folder | `spec/` mirroring `lib/` | RSpec and Rails conventions; no Ruby rule |

### Feature responsibility folders per ecosystem

Every ecosystem translates the same responsibilities. The column names are
the **responsibility**, not a literal folder: the framework's own convention
wins when it names the folder differently. The frontend rows keep the literal
tree; the backend rows translate it to the idiom of the framework, because a
backend feature has no hooks or context and inventing them would be the same
mistake in the opposite direction. A dash means the responsibility does not
apply: a stateless backend feature has no local state folder.

| Ecosystem | Interface / UI | Behavior | Transport / service | Contracts / types | Local state | Public API |
|---|---|---|---|---|---|---|
| TypeScript, JavaScript, React, Next.js | `components/` | `hooks/` | `services/` | `types/` | `context/` | `index.ts` |
| Angular | `components/` | `data-access/` | `services/` | `models/` or `types/` | `store/` | `index.ts` |
| Vue, Nuxt | `components/` | `composables/` | `services/` | `types/` | `stores/` | `index.ts` |
| Svelte, SvelteKit | `components/` | `hooks/` | `services/` | `types/` | `stores/` | `index.ts` |
| Solid, SolidStart | `components/` | `hooks/` | `services/` | `types/` | `context/` | `index.ts` |
| React Native | `components/` | `hooks/` | `services/` | `types/` | `context/` | `index.ts` |
| Ionic (Angular) | `components/` | `data-access/` | `services/` | `models/` | `store/` | `index.ts` |
| Ionic (React) | `components/` | `hooks/` | `services/` | `types/` | `context/` | `index.ts` |
| Java, Kotlin (Spring) | `api/` | `application/` | `infrastructure/` | `domain/` | — (stateless) | package-level public API |
| Python (Django, FastAPI, Flask) | `api/` | `services/` | `repositories/` | `domain/` | `state/` | `__init__.py` |
| Go | `api/` | `service/` | `repository/` | `domain/` | `state/` | exported identifiers |
| Rust | `api/` | `service/` | `repository/` | `domain/` | `state/` | `mod.rs` or `pub use` |
| C# / .NET | `Api/` | `Services/` | `Infrastructure/` | `Domain/` | `State/` | public types |
| PHP | `Api/` | `Services/` | `Infrastructure/` | `Domain/` | `State/` | namespace |
| Ruby | `api/` | `services/` | `repositories/` | `contracts/` | `models/` | module |

**The source of each row.** The responsibility split is a policy of this
system. The framework's own convention decides the name and the file
granularity: Angular's "one concept per file" and its `data-access/`
convention, the Vue single-file component and its `composables/`, the Svelte
`.svelte` file, Solid's named exports, Spring Modulith's provided interface
and package layout, PSR-4 for PHP, and the Go package-per-directory rule.
No framework prescribes `src/features`; that layer comes from Feature-Sliced
Design. A tree is never presented as a framework standard when it is not.
React Native keeps its native projects (`android/`, `ios/`) at the repository
root, outside `src/`, uses the React tree below, and adds `.ios.tsx` /
`.android.tsx` suffixes when a file is platform-specific. Ionic uses the
Angular tree or the React tree, whichever it detected; Capacitor adds
`android/` and `ios/` at the root and no source-level layout of its own.
The official source behind each framework convention is registered in
`references/official-documentation.md`, including its "Project Structure and
Test Co-location" section and its Spring Modulith entry. The folder split
itself is stated as system policy, not attributed to a framework that does
not prescribe it.

### Feature-driven, concrete trees per ecosystem

**TypeScript, JavaScript, React, Next.js**

```
src/features/payment/
├── components/
│   └── payment-form/
│       ├── payment-form.tsx
│       ├── payment-form.test.tsx
│       └── payment-form.module.css
├── hooks/
│   └── use-payment.ts
├── services/
│   └── payment-api.ts
├── types/
│   └── payment.types.ts
├── context/
│   └── payment-context.tsx
└── index.ts
```

**Angular**

```
src/app/features/payment/
├── components/
│   └── payment-form/
│       ├── payment-form.component.ts
│       ├── payment-form.component.html
│       ├── payment-form.component.css
│       └── payment-form.component.spec.ts
├── data-access/
│   └── payment.service.ts
├── services/
│   └── payment-api.service.ts
├── models/
│   └── payment.model.ts
├── store/
│   └── payment.store.ts
└── index.ts
```

**Vue / Nuxt**

```
src/features/payment/
├── components/
│   └── PaymentForm/
│       ├── PaymentForm.vue
│       └── PaymentForm.test.ts
├── composables/
│   └── usePayment.ts
├── services/
│   └── paymentApi.ts
├── types/
│   └── payment.types.ts
├── stores/
│   └── paymentStore.ts
└── index.ts
```

**Svelte / SvelteKit**

```
src/features/payment/
├── components/
│   └── PaymentForm/
│       ├── PaymentForm.svelte
│       └── PaymentForm.test.ts
├── hooks/
│   └── use-payment.ts
├── services/
│   └── payment-api.ts
├── types/
│   └── payment.types.ts
├── stores/
│   └── payment-store.ts
└── index.ts
```

**Solid / SolidStart**

```
src/features/payment/
├── components/
│   └── PaymentForm/
│       ├── PaymentForm.tsx
│       └── PaymentForm.test.tsx
├── hooks/
│   └── use-payment.ts
├── services/
│   └── payment-api.ts
├── types/
│   └── payment.types.ts
├── context/
│   └── payment-context.tsx
└── index.ts
```

**Java / Kotlin with Spring Boot**

```
src/main/java/com/app/features/payment/
├── api/
│   └── PaymentController.java
├── application/
│   └── PaymentService.java
├── domain/
│   └── Payment.java
├── infrastructure/
│   └── PaymentRepository.java
└── package-info.java
src/test/java/com/app/features/payment/
├── api/PaymentControllerTest.java
├── application/PaymentServiceTest.java
└── infrastructure/PaymentRepositoryTest.java
```

**Python with Django, FastAPI, or Flask**

```
src/features/payment/
├── api/
│   ├── routes.py
│   └── test_routes.py
├── services/
│   ├── payment_service.py
│   └── test_payment_service.py
├── repositories/
│   ├── payment_repository.py
│   └── test_payment_repository.py
├── domain/
│   └── payment.py
├── state/
│   └── payment_state.py
└── __init__.py
```

**Go**

```
internal/features/payment/
├── api/
│   ├── handler.go
│   └── handler_test.go
├── service/
│   ├── service.go
│   └── service_test.go
├── repository/
│   ├── repository.go
│   └── repository_test.go
├── domain/
│   └── payment.go
├── state/
│   └── state.go
└── doc.go
```

**Rust**

```
src/features/payment/
├── api/
│   └── mod.rs
├── service/
│   └── mod.rs
├── repository/
│   └── mod.rs
├── domain/
│   └── mod.rs
├── state/
│   └── mod.rs
└── mod.rs
```

**C# / .NET**

```
src/Features/Payment/
├── Api/
│   └── PaymentController.cs
├── Services/
│   └── PaymentService.cs
├── Infrastructure/
│   └── PaymentRepository.cs
├── Domain/
│   └── Payment.cs
├── State/
│   └── PaymentState.cs
└── PaymentModule.cs
tests/Features/Payment/
├── Api/PaymentControllerTests.cs
└── Services/PaymentServiceTests.cs
```

**PHP**

```
src/Features/Payment/
├── Api/
│   └── PaymentController.php
├── Services/
│   └── PaymentService.php
├── Infrastructure/
│   └── PaymentRepository.php
├── Domain/
│   └── Payment.php
├── State/
│   └── PaymentState.php
└── PaymentModule.php
tests/Features/Payment/
├── Api/PaymentControllerTest.php
└── Services/PaymentServiceTest.php
```

**Ruby**

```
app/features/payment/
├── api/
│   └── payment_controller.rb
├── services/
│   └── payment_service.rb
├── repositories/
│   └── payment_repository.rb
├── contracts/
│   └── payment_contract.rb
├── models/
│   └── payment.rb
└── payment.rb
spec/features/payment/
├── api/payment_controller_spec.rb
└── services/payment_service_spec.rb
```

### Hexagonal, Go

```
internal/
├── domain/              # entities, value objects, no dependencies
│   └── ports.go         # the interfaces the domain needs
├── application/         # use cases
└── adapters/
    ├── http/            # handlers
    └── postgres/        # repository implementations
```

### Hexagonal, Python with FastAPI

```
src/
├── domain/              # entities, value objects
│   └── ports.py
├── application/         # use cases
└── adapters/
    ├── http/            # routers
    └── db/              # repository implementations
```

---

## Deciding the tree

For the detected stack and the agreed style, the system records:

1. the **architecture**: the style and why it fits the scope;
2. the **tree**: the top-level folders, translated to the ecosystem;
3. the **unit decisions**: for each unit, its owner folder, whether it
   becomes its own folder, and the public surface that exposes it;
4. the **dependency directions** allowed between the folders.

The record is presented with the question, so the user sees the folders
before they exist.
