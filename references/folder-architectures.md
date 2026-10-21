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

## Rules that hold in every tree

- **One folder per unit.** A unit that owns an implementation and a test
  gets its own folder, with the test and its types next to the
  implementation. A folder never holds a flat pile of implementations, tests,
  and types mixed together.
- **One public API per module.** `index.ts` (or the ecosystem equivalent)
  exports what other modules may use; the rest stays encapsulated.
- **The dependency direction is downward.** A layer imports the layer
  below it, never the layer above.
- **Contracts live with their owner.** Types and interfaces live in the
  folder of the layer or module that owns them.

---

## Style 1 — Layered

Simple domains, one team, low coupling.

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

Multiple domains, independent features. The default for a modern web
application.

```
src/
├── app/                 # global configuration: providers, routes, state
├── shared/              # generic and reusable, bound to no capability
└── features/
    └── <feature>/
        ├── <unit>/      # one folder per unit: implementation + test + types
        │   ├── <unit>.tsx
        │   ├── <unit>.test.tsx
        │   └── <unit>.types.ts
        ├── api/         # network calls of the feature
        └── index.ts     # the public API of the feature
```

**The golden dependency rule.** A feature imports from `app/` and
`shared/`, and never the internals of another feature. Cross-feature
interaction goes through the other feature's public API, or through the
global state in `app/`. A unit that needs a second consumer is **promoted**
to `shared/`, in the folder that matches its kind; it is not copied.

## Style 3 — Hexagonal / Ports and Adapters

Multiple external integrations, testability critical. The domain is
surrounded by ports; adapters implement them.

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

## Style 4 — DDD

Complex domain logic, ubiquitous language.

```
src/
├── domain/              # aggregates, entities, value objects, domain events
│   └── <context>/       # one folder per bounded context
├── application/         # commands, queries, use cases
├── infrastructure/      # persistence, messaging, external services
└── interfaces/          # HTTP controllers, event handlers, CLI
```

## Style 5 — Modular Monolith

Multiple bounded contexts, one deployable.

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

Asynchronous integration between bounded contexts.

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

| Ecosystem | Unit of organization | Public API of a module | Unit folder |
|---|---|---|---|
| TypeScript / JavaScript | folder | `index.ts` | folder |
| React / Next.js | folder | `index.ts` | folder |
| Angular | folder | `index.ts` or a public `NgModule` | folder |
| Vue | folder | `index.ts` | folder |
| Svelte / SvelteKit | folder | `index.ts` | folder |
| Solid | folder | `index.ts` | folder |
| Java / Kotlin | package | package-level public API | package folder |
| Python (Django) | app | the app's `urls.py` and `models.py` | package folder |
| Python (FastAPI / Flask) | package | `__init__.py` | package folder |
| Go | package | the package's exported identifiers | package folder |
| Rust | module or crate | `pub use` in `mod.rs` or `lib.rs` | module folder |
| C# / .NET | project or folder | `public` types | folder |
| PHP | namespace | the namespace's public classes | folder |
| Ruby | engine or module | the module's public methods | folder |

**Concrete variants of the two most distinct trees.**

### Feature-driven, Angular

```
src/app/
├── core/                # singletons, services, guards
├── shared/              # reusable components, directives, pipes
├── features/
│   └── cart/
│       ├── data-access/ # services and HTTP calls
│       ├── ui/          # components
│       └── cart.routes.ts
```

### Feature-driven, Java with Spring Boot

```
src/main/java/com/app/
├── cart/
│   ├── api/             # controllers
│   ├── application/     # services
│   ├── domain/          # entities and value objects
│   └── infrastructure/  # repositories
└── shared/
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
