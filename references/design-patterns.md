# Design Pattern Selection

## Purpose

Every implementation task identifies the design patterns its design forces
require, before code is written. The selection is independent of
the language, framework, and runtime: the pattern is the design intent, and
the implementation is translated to the detected stack's idiom.

The result is a **set**, never a quota. A task may select 0, 2, 32, or 100+
patterns. The count is derived from the forces that are actually present.
A pattern without a signal is a defect, and a target count is a defect.

This reference is the single canonical procedure. Skills and workflows
reference it instead of restating the catalog or the algorithm. Load it in
Task-to-Code Phase 8, in architecture evaluation, in code-audit Dimension 2,
and in engineering review; the selection is recorded even when it is empty.

## Scope

The catalog covers:

- object and class design (GoF);
- enterprise application and persistence design (Fowler, PoEAA);
- messaging and integration (Enterprise Integration Patterns);
- architectural styles;
- concurrency;
- functional and language-level composition;
- UI and client composition;
- reliability, data consistency, and security.

Architectural styles are selected by the same procedure but at the
architecture gate, because they change boundaries and deployment. They are
part of the count when the task selects them; they are not a separate
quota.

This reference does not define folder structure, file placement, or stack
syntax. Those belong to `references/folder-architectures.md`,
`skills/implementation-recipes/SKILL.md`, and the official documentation
index.

## Model

- **Force** — a pressure the task must resolve, such as “several
  interchangeable payment providers”, “the order has a finite lifecycle”,
  or “the report is a tree and needs several aggregations”.
- **Signal** — observable evidence in the task, the acceptance criteria,
  the design, or the repository that the force is present.
- **Candidate** — a catalog pattern whose intent addresses the force.
- **Existing mechanism** — a project abstraction, framework capability, or
  language construct that already resolves the force. It is preferred over
  a new pattern.
- **Selected** — a candidate with at least one signal, a force that is not
  already resolved, and a benefit that justifies its cost.
- **Rejected** — a candidate with no signal, a force already resolved, a
  redundant force, or a cost that exceeds its benefit.
- **Count** — the number of distinct selected patterns after composition
  and deduplication. It is an output, never an input.

Names are weak evidence. A class called `PaymentStrategy` does not prove a
Strategy, and an interface called `Repository` does not prove a Repository.
Evidence is structural (roles, collaborations, variation points) and
behavioral (dispatch, lifecycle, call sites), not nominal.

## Signal Taxonomy

Signals are language-neutral. Each signal names evidence that can be
observed in a task or a codebase, not a technology.

### Creation

| ID | Signal | Evidence |
|---|---|---|
| CR1 | Multiple interchangeable products | Several implementations of the same contract, chosen by context |
| CR2 | Runtime selection by key or configuration | A provider, plugin, or adapter is chosen from a key, config, or environment |
| CR3 | Complex construction with optional parts | Many optional fields, defaults, or invariants assembled step by step |
| CR4 | Expensive creation or shared instance | Creation is costly, or exactly one instance must coordinate access |
| CR5 | Family of related products | Several products must be created together and stay consistent |
| CR6 | Clone or configured prototype | A preconfigured instance is copied and adjusted |

### Structure

| ID | Signal | Evidence |
|---|---|---|
| ST1 | Incompatible external interface | A third-party SDK does not match the domain contract |
| ST2 | Transparent behavior augmentation | Logging, caching, retry, audit, or metrics wrap an existing behavior |
| ST3 | Access, lazy, or remote control | A boundary must gate, defer, or proxy access behind the same interface |
| ST4 | Part-whole tree | Leaves and groups are treated uniformly |
| ST5 | Many collaborating subsystems | Callers must coordinate several collaborators to complete one use case |
| ST6 | Many fine-grained shared objects | Memory pressure from many small objects with shared state |
| ST7 | Abstraction and implementation vary independently | Two dimensions of change must not multiply into a class explosion |
| ST8 | Legacy or third-party integration boundary | An external system must be isolated behind a domain-owned boundary |

### Behavior

| ID | Signal | Evidence |
|---|---|---|
| BE1 | Interchangeable algorithms | Several algorithms or policies implement the same contract |
| BE2 | Queueable or reversible operation | An operation must be queued, retried, logged, or undone |
| BE3 | One-to-many event notification | A state change must notify several independent subscribers |
| BE4 | Lifecycle state transitions | Legal behavior depends on the current state |
| BE5 | Ordered handler chain | A request passes through ordered validators or handlers |
| BE6 | Many-to-many coordination | Components communicate through a coordinator instead of each other |
| BE7 | Snapshot and restore | State must be saved and restored without exposing internals |
| BE8 | Sequential traversal | A collection is traversed page by page or step by step |
| BE9 | Grammar or language interpretation | A small language, expression, or rule grammar must be evaluated |
| BE10 | Fixed algorithm skeleton | Several flows share a sequence but differ in individual steps |
| BE11 | Operation over a tree | An operation must be computed over a structure without changing every node |

### Data and persistence

| ID | Signal | Evidence |
|---|---|---|
| DA1 | Storage isolation | Domain code must not depend on a database, ORM, or transport |
| DA2 | Transaction boundary | Several changes must commit or roll back together |
| DA3 | Identity map | One instance per identity per session |
| DA4 | Lazy loading | A relationship is loaded only when used |
| DA5 | Query construction or composition | A query has many optional filters or reusable predicates |
| DA6 | Domain/data mapping | Objects and rows have different shapes and must be mapped |
| DA7 | Table or row gateway | Access is organized around a table or a row |
| DA8 | Use-case or service layer | A use case needs an explicit boundary over domain logic |
| DA9 | Rich domain model | Behavior and invariants belong to domain objects |
| DA10 | Value semantics | A concept has equality and no identity (money, date range, code) |
| DA11 | Caching | Repeated reads are expensive and tolerate stale or shared data |
| DA12 | Consistency across services | A change spans services and needs compensation or reliable publication |
| DA13 | Partitioning or sharding | Data or load must be partitioned by a key |
| DA14 | Replication | Data or load must be copied for availability or scale |

### Integration and messaging

| ID | Signal | Evidence |
|---|---|---|
| IN1 | Message channel | Sender and receiver are connected through a channel |
| IN2 | Message routing | A message is routed by content or rules |
| IN3 | Message translation | Producer and consumer use different formats |
| IN4 | Message endpoint | An application must connect to a messaging system |
| IN5 | Publish-subscribe | One message reaches many subscribers |
| IN6 | Point-to-point | One message reaches exactly one consumer |
| IN7 | Dead letter | Undeliverable messages must be isolated |
| IN8 | Idempotent receiver | Duplicate delivery must be safe |
| IN9 | Aggregation | Related messages must be combined |
| IN10 | Splitting | One message must become many |
| IN11 | Enrichment | A message needs data from another source |
| IN12 | Filtering | A message contains data the consumer must not receive |
| IN13 | Routing slip | The route travels with the message |
| IN14 | Process manager | A multi-step process needs central state |
| IN15 | Saga or compensation | A distributed transaction needs compensating actions |
| IN16 | Claim check | A large payload is stored and referenced |

### Architecture

| ID | Signal | Evidence |
|---|---|---|
| AR1 | Layering | Technical layers with allowed dependency directions |
| AR2 | Ports and adapters | The domain is isolated behind ports and adapters |
| AR3 | Dependency rule | Dependencies point toward the domain |
| AR4 | MVC family | Model, view, and controller/presenter responsibilities are separated |
| AR5 | MVVM | A view binds to a view model |
| AR6 | CQRS | Reads and writes have different models |
| AR7 | Event sourcing | State is persisted as a sequence of events |
| AR8 | Modular monolith | Modules with explicit boundaries in one deployable |
| AR9 | Microservices | Independently deployable services by business boundary |
| AR10 | Backend for frontend | A backend is tailored to one client experience |
| AR11 | API gateway | One entry point fronts several services |
| AR12 | Service mesh | Service-to-service communication is handled by infrastructure |
| AR13 | Sidecar | Supporting behavior runs beside the service |
| AR14 | Strangler fig | A legacy system is replaced incrementally |

### Concurrency

| ID | Signal | Evidence |
|---|---|---|
| CO1 | Async result | A result is pending and will complete later |
| CO2 | Resource pool | Expensive resources are reused by a bounded set of workers |
| CO3 | Event loop or reactor | Events are dispatched to handlers from one loop |
| CO4 | Producer-consumer | Production and consumption have different rates |
| CO5 | Actor | State is isolated and communicated by messages |
| CO6 | Read-write lock | Concurrent reads are allowed, writes are exclusive |
| CO7 | Active object | Method execution is decoupled from invocation |
| CO8 | Half-sync/half-async | Async I/O and synchronous processing are separated |
| CO9 | Leader/followers | Workers share an event source |
| CO10 | Proactor | Completion events drive asynchronous operations |
| CO11 | Monitor | Access to an object is synchronized |

### Functional and language-level

| ID | Signal | Evidence |
|---|---|---|
| FN1 | Higher-order function | A function is passed or returned |
| FN2 | Closure | State is captured by a function |
| FN3 | Currying | A multi-argument function is applied in stages |
| FN4 | Function composition | A function is built from smaller functions |
| FN5 | Functor | A value in a context is mapped |
| FN6 | Monad | Computations in a context are sequenced |
| FN7 | Maybe or option | Presence and absence are explicit |
| FN8 | Either or result | Success and failure are explicit |
| FN9 | Lens | Nested immutable data is focused and updated |
| FN10 | Transducer | Transformations are composed without intermediate collections |
| FN11 | Persistent or immutable data | Versions share structure instead of copying |

### UI and client

| ID | Signal | Evidence |
|---|---|---|
| UI1 | Container/presentational | Data concerns are separated from rendering |
| UI2 | Compound component | Related components share implicit state and composition |
| UI3 | Render props | Behavior is shared through a function prop |
| UI4 | Higher-order component | A component is wrapped to add behavior |
| UI5 | Provider/context | Dependencies are shared without prop drilling |
| UI6 | Custom hook or composable | Stateful logic is reused across components |
| UI7 | State reducer | State transitions are centralized |
| UI8 | Observable store | Views subscribe to a store |
| UI9 | Optimistic UI | A change is shown before confirmation |
| UI10 | Virtualized list | Only visible items are rendered |

### Reliability and security

| ID | Signal | Evidence |
|---|---|---|
| RE1 | Circuit breaker | Calls to a failing dependency must stop temporarily |
| RE2 | Bulkhead | Failures must be isolated by resource pool |
| RE3 | Retry | Transient failures are retried |
| RE4 | Timeout | Waiting is bounded |
| RE5 | Fallback | A degraded response is acceptable |
| RE6 | Rate limiter | Request rate is bounded |
| RE7 | Health check | Liveness or readiness is exposed |
| RE8 | Idempotency key | Repeated requests must be safe |
| SE1 | Authentication strategy | Several authentication mechanisms are selectable |
| SE2 | Authorization policy | Access rules are evaluated at a boundary |
| SE3 | Session | Authenticated state is maintained |
| SE4 | Token | Claims are carried without server state |
| SE5 | Sandbox | Untrusted code or content is isolated |
| SE6 | Audit log | Security-relevant actions are recorded |
| SE7 | Guard | A precondition is enforced at a boundary |

## Catalog

The catalog is the candidate space. It is intentionally larger than the
GoF catalog: a task can need a persistence pattern, an integration pattern,
a UI pattern, and an architectural style at the same time. The catalog is
extensible: adding a pattern means adding a row, not changing the algorithm.

The GoF section is complete: 5 creational, 7 structural, and 11 behavioral
patterns (23). Refactoring Guru's public catalog lists 22 classic patterns
and omits Interpreter; this catalog keeps Interpreter because it is part of
the GoF set. Refactoring Guru is a practical implementation reference, not
a replacement for the GoF catalog.

### GoF — Creational

| ID | Pattern | Intent | Signals |
|---|---|---|---|
| GOF-01 | Abstract Factory | Create families of related objects without naming concrete classes | CR5, CR2 |
| GOF-02 | Builder | Separate construction of a complex object from its representation | CR3 |
| GOF-03 | Factory Method | Let a creator defer instantiation to subclasses | CR1, CR2 |
| GOF-04 | Prototype | Create new objects by cloning a configured instance | CR6 |
| GOF-05 | Singleton | Guarantee one instance with a controlled access point | CR4 |

### GoF — Structural

| ID | Pattern | Intent | Signals |
|---|---|---|---|
| GOF-06 | Adapter | Convert an incompatible interface into the one clients expect | ST1, ST8 |
| GOF-07 | Bridge | Separate abstraction from implementation so both vary independently | ST7 |
| GOF-08 | Composite | Treat leaves and containers uniformly in a part-whole tree | ST4 |
| GOF-09 | Decorator | Add behavior transparently without changing the wrapped object | ST2 |
| GOF-10 | Facade | Provide one entry point to a subsystem | ST5 |
| GOF-11 | Flyweight | Share fine-grained objects to reduce memory | ST6 |
| GOF-12 | Proxy | Control access, laziness, or remoteness behind the same interface | ST3 |

### GoF — Behavioral

| ID | Pattern | Intent | Signals |
|---|---|---|---|
| GOF-13 | Chain of Responsibility | Pass a request through ordered handlers until one handles it | BE5 |
| GOF-14 | Command | Encapsulate a request as an object for queueing, logging, or undo | BE2 |
| GOF-15 | Interpreter | Represent a grammar and interpret sentences | BE9 |
| GOF-16 | Iterator | Traverse a collection without exposing its representation | BE8 |
| GOF-17 | Mediator | Centralize many-to-many communication | BE6 |
| GOF-18 | Memento | Capture and restore state without exposing internals | BE7 |
| GOF-19 | Observer | Notify subscribers when state changes | BE3 |
| GOF-20 | State | Let an object change behavior when its state changes | BE4 |
| GOF-21 | Strategy | Make interchangeable algorithms selectable | BE1 |
| GOF-22 | Template Method | Fix an algorithm skeleton and vary steps | BE10 |
| GOF-23 | Visitor | Add operations over an object structure without changing its classes | BE11 |

### Enterprise application patterns

| ID | Pattern | Intent | Signals |
|---|---|---|---|
| EAA-01 | Repository | Isolate domain access from storage | DA1 |
| EAA-02 | Unit of Work | Track changes and commit them as one transaction | DA2 |
| EAA-03 | Identity Map | Keep one instance per identity per session | DA3 |
| EAA-04 | Lazy Load | Defer loading until needed | DA4 |
| EAA-05 | Data Mapper | Move data between objects and tables without coupling | DA6 |
| EAA-06 | Active Record | Put persistence on the domain object | DA6, DA9 |
| EAA-07 | Table Data Gateway | One gateway per table | DA7 |
| EAA-08 | Row Data Gateway | One gateway per row | DA7 |
| EAA-09 | Query Object | Represent a query as an object | DA5 |
| EAA-10 | Service Layer | Define a use-case boundary over the domain | DA8 |
| EAA-11 | Domain Model | Model domain behavior and rules | DA9 |
| EAA-12 | Transaction Script | Organize procedural use cases | DA8 |
| EAA-13 | Money | Represent amount and currency as a value | DA10 |
| EAA-14 | Special Case | Represent a special situation as an object | DA10 |
| EAA-15 | Plugin | Load implementations by configuration | CR2 |
| EAA-16 | Separated Interface | Put an interface in a different package from its implementation | ST7 |
| EAA-17 | Gateway | Encapsulate access to an external system | ST8, DA1 |
| EAA-18 | Mapper | Move data between two representations | DA6 |
| EAA-19 | Record Set | Represent tabular data in memory | DA5 |
| EAA-20 | Value Object | Represent a value with equality and no identity | DA10 |
| EAA-21 | Specification | Compose business rules as predicates and reuse them across queries | DA5 |

### Integration patterns

| ID | Pattern | Intent | Signals |
|---|---|---|---|
| EIP-01 | Message Channel | Connect sender and receiver through a channel | IN1 |
| EIP-02 | Message Router | Route a message to one of several destinations | IN2 |
| EIP-03 | Message Translator | Translate between data formats | IN3 |
| EIP-04 | Message Endpoint | Connect an application to the messaging system | IN4 |
| EIP-05 | Publish-Subscribe Channel | Deliver one message to many subscribers | IN5 |
| EIP-06 | Point-to-Point Channel | Deliver one message to one consumer | IN6 |
| EIP-07 | Dead Letter Channel | Store undeliverable messages | IN7 |
| EIP-08 | Idempotent Receiver | Process duplicates safely | IN8 |
| EIP-09 | Aggregator | Combine related messages | IN9 |
| EIP-10 | Splitter | Split one message into many | IN10 |
| EIP-11 | Content Enricher | Add missing data to a message | IN11 |
| EIP-12 | Content Filter | Remove unneeded data | IN12 |
| EIP-13 | Routing Slip | Carry the route with the message | IN13 |
| EIP-14 | Process Manager | Coordinate a multi-step process | IN14 |
| EIP-15 | Saga | Coordinate distributed transactions with compensation | IN15 |
| EIP-16 | Claim Check | Store large payloads and pass a reference | IN16 |

### Architectural styles

| ID | Pattern | Intent | Signals |
|---|---|---|---|
| ARC-01 | Layered Architecture | Organize by layers with allowed dependencies | AR1 |
| ARC-02 | Hexagonal / Ports and Adapters | Isolate the domain behind ports and adapters | AR2 |
| ARC-03 | Clean Architecture | Apply the dependency rule toward the domain | AR3 |
| ARC-04 | MVC | Separate model, view, and controller | AR4 |
| ARC-05 | MVP | Separate view and presenter | AR4 |
| ARC-06 | MVVM | Bind view to a view model | AR5 |
| ARC-07 | CQRS | Separate read and write models | AR6 |
| ARC-08 | Event Sourcing | Persist state as a sequence of events | AR7 |
| ARC-09 | Modular Monolith | Modules with explicit boundaries in one deployable | AR8 |
| ARC-10 | Microservices | Independently deployable services by boundary | AR9 |
| ARC-11 | Backend for Frontend | A backend tailored to a client | AR10 |
| ARC-12 | API Gateway | One entry point for multiple services | AR11 |
| ARC-13 | Service Mesh | Infrastructure-level service communication | AR12 |
| ARC-14 | Sidecar | Attach supporting behavior beside the service | AR13 |
| ARC-15 | Strangler Fig | Replace a legacy system incrementally | AR14 |

### Concurrency patterns

| ID | Pattern | Intent | Signals |
|---|---|---|---|
| CON-01 | Active Object | Decouple method execution from invocation | CO1, CO7 |
| CON-02 | Monitor Object | Synchronize access to an object | CO11 |
| CON-03 | Half-Sync/Half-Async | Separate async I/O from sync processing | CO8 |
| CON-04 | Leader/Followers | Share event sources among workers | CO9 |
| CON-05 | Thread Pool | Reuse a bounded set of workers | CO2 |
| CON-06 | Reactor | Dispatch events to handlers in one loop | CO3 |
| CON-07 | Proactor | Dispatch completion events from async operations | CO10 |
| CON-08 | Producer-Consumer | Decouple production from consumption through a queue | CO4 |
| CON-09 | Read-Write Lock | Allow concurrent reads, exclusive writes | CO6 |
| CON-10 | Future/Promise | Represent a pending result | CO1 |
| CON-11 | Actor | Isolate state and communicate by messages | CO5 |

### Functional and language-level patterns

| ID | Pattern | Intent | Signals |
|---|---|---|---|
| FUN-01 | Higher-Order Function | Pass or return functions | FN1 |
| FUN-02 | Closure | Capture state in a function | FN2 |
| FUN-03 | Currying | Transform a multi-argument function into a chain | FN3 |
| FUN-04 | Function Composition | Build a function from smaller functions | FN4 |
| FUN-05 | Functor | Map over a context | FN5 |
| FUN-06 | Monad | Sequence computations in a context | FN6 |
| FUN-07 | Maybe/Option | Represent presence or absence | FN7 |
| FUN-08 | Either/Result | Represent success or failure | FN8 |
| FUN-09 | Lens | Focus and update nested immutable data | FN9 |
| FUN-10 | Transducer | Compose efficient transformations | FN10 |
| FUN-11 | Persistent/Immutable Data | Share structure across versions | FN11 |

### UI and client patterns

| ID | Pattern | Intent | Signals |
|---|---|---|---|
| UI-01 | Container/Presentational | Separate data concerns from rendering | UI1 |
| UI-02 | Compound Component | Compose related components with shared context | UI2 |
| UI-03 | Render Props | Share behavior through a function prop | UI3 |
| UI-04 | Higher-Order Component | Wrap a component to add behavior | UI4 |
| UI-05 | Provider/Context | Share dependencies without prop drilling | UI5 |
| UI-06 | Custom Hook / Composable | Reuse stateful logic | UI6 |
| UI-07 | State Reducer | Centralize state transitions | UI7 |
| UI-08 | Observable Store | Subscribe views to a store | UI8 |
| UI-09 | Optimistic UI | Apply changes before confirmation | UI9 |
| UI-10 | Virtualized List | Render only visible items | UI10 |

### Reliability patterns

| ID | Pattern | Intent | Signals |
|---|---|---|---|
| REL-01 | Circuit Breaker | Stop calls to a failing dependency | RE1 |
| REL-02 | Bulkhead | Isolate failures by resource pool | RE2 |
| REL-03 | Retry | Retry transient failures | RE3 |
| REL-04 | Timeout | Bound waiting | RE4 |
| REL-05 | Fallback | Provide a degraded response | RE5 |
| REL-06 | Rate Limiter | Bound request rate | RE6 |
| REL-07 | Health Check | Expose liveness or readiness | RE7 |
| REL-08 | Idempotency Key | Make repeated requests safe | RE8 |

### Data and consistency patterns

| ID | Pattern | Intent | Signals |
|---|---|---|---|
| DAT-01 | Cache-Aside | Load on miss and populate the cache | DA11 |
| DAT-02 | Write-Through | Write through the cache to the store | DA11 |
| DAT-03 | Read-Through | The cache loads on behalf of the caller | DA11 |
| DAT-04 | Write-Behind | Write asynchronously to the store | DA11 |
| DAT-05 | Sharding | Partition data by key | DA13 |
| DAT-06 | Replication | Copy data for availability or scale | DA14 |
| DAT-07 | Outbox | Persist events with the transaction and publish later | DA12, IN8 |
| DAT-08 | Inbox | Deduplicate consumed messages | DA12, IN8 |

### Security patterns

| ID | Pattern | Intent | Signals |
|---|---|---|---|
| SEC-01 | Authentication Strategy | Select an authentication mechanism | SE1 |
| SEC-02 | Authorization Policy | Evaluate access rules | SE2 |
| SEC-03 | Session | Maintain authenticated state | SE3 |
| SEC-04 | Token | Carry claims without server state | SE4 |
| SEC-05 | Sandbox | Isolate untrusted execution | SE5 |
| SEC-06 | Audit Log | Record security-relevant actions | SE6 |
| SEC-07 | Guard | Enforce a precondition at a boundary | SE7 |

## Selection Procedure

### Step 1 — Build the design profile

Record, with evidence:

- the objective and acceptance criteria;
- the actors, data, boundaries, and lifecycle;
- the non-functional qualities that matter (latency, consistency,
  availability, security, cost, operability);
- the detected stack, version, and existing mechanisms;
- the constraints and non-goals.

### Step 2 — Extract signals

For every part of the task, record the signal IDs that are actually
present. Cite the evidence: a sentence in the task, an acceptance
criterion, a design artifact, or a `file:line` in the repository. Do not
infer a signal from a pattern name.

### Step 3 — Query the catalog

For each signal, collect the catalog rows that cite it. Union the
candidates into one set keyed by pattern ID. A candidate with several
signals keeps all of them.

### Step 4 — Reject before selecting

Reject a candidate when any of these is true:

- no signal supports it;
- an existing project abstraction, framework capability, or language
  construct already resolves the force;
- another selected pattern already resolves the same force and the two are
  not a recognized composition;
- its cost (indirection, indirection, runtime, cognitive load, operational
  burden) exceeds its benefit for this task;
- it would force a class shape the stack does not need.

Record every rejection and its reason. A rejection is evidence, not a
failure.

### Step 5 — Select and compose

Select the remaining candidates. Compose patterns that solve different
forces and reinforce each other. Order the set by dependency:

1. architectural style and boundaries;
2. creation and structure;
3. behavior and control;
4. data and persistence;
5. integration and messaging;
6. concurrency;
7. reliability and security;
8. UI and client.

Resolve conflicts explicitly. Record the composition in the plan.

### Step 6 — Translate to the stack

For every selected pattern, record:

- the stack idiom that implements it;
- the files or modules that will express it;
- the existing mechanism it reuses, if any;
- the official documentation consulted, when the idiom is version-sensitive.

The pattern name is the design intent. The implementation must be idiomatic
for the detected stack. A GoF class diagram is never a requirement.

### Step 7 — Validate and count

Every selected pattern must have:

- at least one signal;
- a named force;
- a stated benefit;
- a stack translation;
- no unresolved conflict with another selected pattern.

`count = selected.length`. Report the count and the per-flow/program
breakdown. Do not adjust the set to reach a number.

### Procedure as pseudocode

```text
select(profile, catalog, existing, constraints):
  signals = extract(profile, catalog.signals)
  candidates = catalog.bySignals(signals)
  selected = []
  rejected = []

  for candidate in candidates ordered by dependency:
    if candidate.signals is empty:
      rejected.add(candidate, "no signal")
      continue
    if existing.covers(candidate.force):
      rejected.add(candidate, "existing mechanism")
      continue
    if selected.covers(candidate.force) and not composes(selected, candidate):
      rejected.add(candidate, "force already covered")
      continue
    if cost(candidate, profile) > benefit(candidate, profile):
      rejected.add(candidate, "cost exceeds benefit")
      continue
    selected.add(candidate)

  validate(selected)
  return { selected, rejected, count: selected.length }
```

There is no maximum in the algorithm. The only bounds are evidence,
non-redundancy, and cost.

## Existing Patterns

Before proposing a new pattern, inventory the patterns already present in
the affected modules:

- inspect the public contracts and their implementations;
- inspect the composition root and dependency wiring;
- inspect event, persistence, and boundary code;
- verify the roles and collaborations, not the names.

Mark each pattern as `EXISTING` when it already resolves the force,
`INCOMPLETE` when it exists but does not cover the task, and `ABSENT` when
the force is unresolved. Reuse `EXISTING`; extend `INCOMPLETE` only when the
task requires it; select a new pattern only for `ABSENT`.

The no-unsolicited-refactoring rule applies: a pre-existing pattern that is
not touched by the task is context, not a task.

## Composition and Conflicts

Common compositions:

| Composition | Why they coexist |
|---|---|
| Strategy + Factory Method | The strategy is selected at runtime through a factory |
| Composite + Visitor | The tree is stable and operations vary |
| Composite + Iterator | The tree is traversed without exposing its structure |
| Command + Memento | An operation is undone by restoring a snapshot |
| Repository + Unit of Work | Aggregates are persisted and committed together |
| Outbox + Idempotent Receiver | Events are published reliably and consumed safely |
| State + Command | Transitions are explicit and the triggering operation is undoable |
| Facade + Adapter | One entry point hides several vendor adapters |

Common conflicts:

| Conflict | Resolution |
|---|---|
| Strategy vs State | Strategy varies an algorithm; State varies behavior by lifecycle. Select by force, not by structure. |
| Adapter vs Facade | Adapter changes an interface; Facade simplifies a subsystem. They can compose. |
| Decorator vs Proxy | Decorator adds behavior; Proxy controls access. They can compose, but do not duplicate the same wrapper. |
| Active Record vs Data Mapper | Both map data; choose one persistence style per boundary unless a legacy constraint requires both. |
| CQRS vs simple CRUD | CQRS is selected only when read and write models genuinely diverge; otherwise it is cost without benefit. |

## Count Semantics and Scale

The count is `selected.length` after composition and deduplication. It is
not a target, a maturity score, or a measure of quality.

- A typo or a one-line fix selects 0 patterns.
- A settings screen with persistence and a simple entry point selects 2
  (Repository, Facade).
- A checkout flow with providers, validation, undo, persistence, and a
  vendor SDK selects 7.
- A fulfillment flow with lifecycle, events, compensation, and carrier
  integration selects 9.
- A multi-tenant commerce program that combines checkout, fulfillment,
  reporting, search, identity, and operations can select 32.
- A program that spans several products and all catalog families can select
  100+; the catalog currently contains 130 patterns.

A large count is valid only when the profile contains that many distinct
forces. If the count grows because one flow is being treated as a program,
split the work by flow and count each flow separately, then count the union
at the program level. Splitting is a scope decision, not a way to lower the
count.

A small count is not a failure. A pattern without a force is the failure.

## Stack Neutrality

The selection is language- and framework-neutral. The pattern is the intent;
the implementation is the stack's idiom.

Rules:

- Prefer the stack's native construct over a GoF class shape: functions,
  closures, modules, interfaces, protocols, traits, generics, events,
  streams, channels, macros, or decorators.
- Prefer a framework capability over a new abstraction: dependency
  injection, routing, middleware, state management, ORM, messaging, and
  observability are often already provided.
- Never force a class hierarchy where the language has first-class
  functions, sum types, or protocols.
- Never force a dynamic pattern where the language's type system or module
  system already expresses the intent.
- Consult the official documentation for the detected stack and version
  when the idiom is version-sensitive.
- Record the translation. “Strategy” in TypeScript may be an interface plus
  a map of functions; in Go it may be an interface plus a constructor; in
  Python it may be a `Protocol` plus callables; in Rust it may be a trait
  plus an enum; in Java it may be an interface plus DI.

### Translation examples

| Pattern | TypeScript / JavaScript | React / Next.js | Angular | Java / Kotlin | C# / .NET | Python | Go | Rust |
|---|---|---|---|---|---|---|---|---|
| Strategy | Interface + function map | Hook or prop strategy | Injection token + implementation | Interface + DI | Interface + DI | `Protocol` + callables | Interface + constructor | Trait + enum |
| Factory Method | Factory function or class | Server action or route factory | Factory provider | Factory class or `@Bean` | Factory method or DI | Factory function | Constructor function | Associated function |
| Adapter | Wrapper function or class | API client adapter | HTTP interceptor | Adapter class | Adapter class | Wrapper class or function | Interface implementation | Trait implementation |
| Decorator | Higher-order function | Wrapper component or hook | Interceptor | Decorator or proxy | Delegating handler | Decorator or wrapper | Middleware function | Wrapper type |
| Observer | `EventTarget` or observable | Context + subscription | RxJS `Subject` | `Flow` or listener | `event` or `IObservable` | Callback list or `asyncio` | Channel or callback | Channel or callback |
| Repository | Interface + implementation | Server data access | Service + repository | Interface + adapter | Interface + EF Core | Protocol + adapter | Interface + struct | Trait + struct |
| Chain | Array of handlers | Middleware | Interceptor chain | Filter chain | Middleware pipeline | List of callables | Middleware chain | Iterator of handlers |
| State | Discriminated union + reducer | Reducer | State service | Sealed class + `when` | Switch expression | `match` or enum | Enum + switch | Enum + `match` |
| Composite | Tree of nodes | Recursive component | Recursive component | Composite class | Composite class | Recursive dataclass | Recursive struct | Recursive enum |
| Visitor | Discriminated union + function | Render tree walk | Recursive visitor | Visitor interface | Visitor interface | `functools.singledispatch` | Type switch | Trait + `match` |

Other supported stacks translate the same intents with their own
mechanisms:

| Stack | Native mechanisms to prefer | Example translation |
|---|---|---|
| Vue / Svelte / Solid | Composables, stores, slots, signals, context | Strategy = composable returning a function; Observer = store subscription |
| PHP | Interfaces, traits, closures, PSR-15 middleware, DI | Strategy = interface + container binding; Chain = middleware pipeline |
| Ruby | Modules, blocks and procs, concerns, `Enumerable` | Strategy = lambda or object; Decorator = module `prepend` |
| Mobile (Swift, Kotlin, Flutter, React Native) | Platform DI, state holders, repositories, navigation | Strategy = protocol or interface; Observer = state flow or listener |
| SQL and data | Views, CTEs, transactions, indexes, constraints | Repository = query boundary; Unit of Work = transaction |

The tables are illustrative, not exhaustive. The detected stack's official
documentation wins.

## Worked Examples

### Eight flows

| Flow | Signals | Selected patterns | Count |
|---|---|---|---|
| Settings | DA1, ST5 | Repository, Facade | 2 |
| User administration | CR3, DA5, DA1, ST5 | Factory Method, Repository, Facade, Specification | 4 |
| Notifications | BE1, CR2, ST2, BE3, BE10 | Strategy, Factory Method, Decorator, Observer, Template Method | 5 |
| Federated authentication | BE1, ST1, ST2, ST3, BE4, ST5 | Strategy, Adapter, Decorator, Proxy, State, Facade | 6 |
| Checkout | BE1, CR2, ST1, BE5, BE2, DA1, ST5 | Strategy, Factory Method, Adapter, Command, Repository, Chain of Responsibility, Facade | 7 |
| Reporting | BE1, BE10, ST4, BE11, BE8 | Strategy, Template Method, Composite, Visitor, Iterator | 5 |
| Search | BE1, CR3, ST2, BE8, ST5 | Strategy, Decorator, Builder, Facade, Iterator | 5 |
| Fulfillment | BE1, CR2, ST1, BE5, BE2, DA1, BE4, BE3, ST5 | Strategy, Factory Method, Adapter, Observer, Command, Repository, Chain of Responsibility, State, Facade | 9 |

### Scale example: 32 patterns

A commerce program that combines the flows above with cross-cutting
operations selects 32 distinct patterns. The count is the result of 32
distinct forces, not a target:

| Pattern | Signal | Force |
|---|---|---|
| Strategy | BE1 | Several payment and carrier algorithms |
| Factory Method | CR2 | Provider selected from a runtime key |
| Adapter | ST1 | Stripe and DHL SDKs do not match the domain |
| Decorator | ST2 | Audit, cache, and retry wrap existing behavior |
| Observer | BE3 | Order status notifies several subscribers |
| Command | BE2 | Checkout and fulfillment are undoable |
| Repository | DA1 | Orders are persisted through a domain boundary |
| Chain of Responsibility | BE5 | Cart and order validation are ordered |
| Facade | ST5 | One entry point coordinates the flow |
| State | BE4 | The order has a finite lifecycle |
| Builder | CR3 | Search queries have many optional filters |
| Composite | ST4 | Reports are a tree of leaves and groups |
| Visitor | BE11 | Totals and CSV are operations over the tree |
| Iterator | BE8 | Results are paged |
| Specification | DA5 | User and report filters compose |
| Proxy | ST3 | Secret access is guarded |
| Template Method | BE10 | Notification channels share a skeleton |
| Memento | BE7 | A cart snapshot can be restored |
| Mediator | BE6 | Fulfillment coordination is centralized |
| Saga | IN15 | Distributed checkout and fulfillment compensate |
| Outbox | DA12 | Events are published reliably |
| Idempotent Receiver | IN8 | Duplicate payment and shipment messages are safe |
| Circuit Breaker | RE1 | A failing payment provider is isolated |
| Retry | RE3 | Transient carrier errors are retried |
| Bulkhead | RE2 | Payment and search pools are isolated |
| Rate Limiter | RE6 | The public API is protected |
| API Gateway | AR11 | One entry point fronts the services |
| Backend for Frontend | AR10 | Web and mobile clients have tailored backends |
| CQRS | AR6 | Reporting and search use read models |
| Event Sourcing | AR7 | The order audit trail is an event log |
| Unit of Work | DA2 | Order and outbox commit together |
| Data Mapper | DA6 | Domain and database shapes are mapped |

A program with 100+ distinct forces selects 100+ patterns. The catalog has
130 rows; the algorithm has no cap. The selection is validated by evidence,
not by a preferred number.

## Validation Checklist

Before implementation:

- [ ] The design profile is recorded with evidence.
- [ ] Every signal is cited from the task, design, or repository.
- [ ] Every selected pattern has a signal, a force, a benefit, and a stack translation.
- [ ] Every rejected candidate has a reason.
- [ ] Existing mechanisms were inventoried before new patterns were added.
- [ ] Compositions and conflicts are explicit.
- [ ] The count is derived and reported, not targeted.
- [ ] The stack idiom is recorded; no class shape is imposed.
- [ ] Official documentation is consulted when the idiom is version-sensitive.
- [ ] The selected set is reflected in the plan; any file or unit introduced
      to implement a selected pattern is recorded in the file responsibility
      map.

After implementation:

- [ ] Every selected pattern is present and used by a real consumer.
- [ ] No pattern was added without a signal.
- [ ] No existing abstraction was duplicated.
- [ ] The implemented idiom matches the stack and the project conventions.
- [ ] The review or audit checks the selection record against the code.

## Sources

- Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides. *Design
  Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley,
  1994. ISBN 978-0201633610.
- Martin Fowler. *Patterns of Enterprise Application Architecture*.
  https://martinfowler.com/books/eaa.html
- Gregor Hohpe, Bobby Woolf. *Enterprise Integration Patterns*.
  https://www.enterpriseintegrationpatterns.com/
- Martin Fowler. *Refactoring* catalog. https://refactoring.com/catalog/
- Refactoring Guru. *Design Patterns* (Spanish):
  https://refactoring.guru/es/design-patterns — practical intent,
  trade-offs, structure, and per-language examples. Use it to explain and
  implement a selected pattern in the detected stack. Its catalog lists 22
  classic patterns and omits Interpreter; the canonical GoF section here
  contains all 23.
- TypeScript Handbook: Generics, Classes, Modules, Iterators and
  Generators. https://www.typescriptlang.org/docs/handbook/
- MDN Web Docs: `Set`, `EventTarget`, iterators.
  https://developer.mozilla.org/en-US/docs/Web/JavaScript
- The official documentation of the detected stack, indexed in
  `references/official-documentation.md`.
