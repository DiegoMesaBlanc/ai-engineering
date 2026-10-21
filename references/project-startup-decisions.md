# Project Startup Decisions

## Purpose

The complete decision set that creating a project from scratch entails, per
project type. It is the reference for `workflows/project-initialization.md`.

Each decision states its recommended default, when the user must decide, and
where the answer comes from. The point is to settle the project once, in one
pass, instead of discovering the missing decisions later as rework.

## How to use

For every decision in this catalog, resolve it in this order:

1. **Already answered by the source analysis**: reuse it. A requirement the
   estimation document states is an explicit requirement. Do not re-ask it.
2. **Governed by a detected convention or an existing policy**: apply it and
   state which one. Do not ask.
3. **Has a defensible default**: take the default, state it explicitly in
   the proposal, and proceed.
4. **Materially changes the outcome and has no default**: ask the user.

## Asking rules

- **One message, grouped by theme.** Never one question per decision, and
  never a sequence of fifteen questions.
- **Each question carries its recommendation and its trade-off.** A
  recommendation is not a decision.
- **Ask only the decisions that reach category 4.** State the decisions
  resolved by categories 1 to 3 in the same message, so the user can correct
  them if needed.
- **Do not ask what the source analysis, the detected conventions, or the
  project policies already settle.**
- The architectural style is always asked, and is asked first, because it
  determines the shape of every other decision.

---

## Universal Decisions

Applies to every project type.

| Decision | Default | User decides when |
|---|---|---|
| Project name and package/module name | Derived from the product goal | The name is a commercial or trademark decision |
| Language and version | The stack assumption in the source analysis | The analysis states alternatives, or the version is not supported |
| Package manager and dependency policy | The ecosystem standard; no new dependency without authorization | A new dependency class, a paid service, or a license constraint |
| Formatter and linter | The ecosystem standard from the Formatting and Linting Policy | The project mandates a different one |
| Structural compliance rules | The rules the ecosystem linter can express, per the coverage matrix in `references/linter-rules.md` | Never; this is a policy decision |
| Test framework and test depth | The ecosystem standard, plus one test that proves the setup runs | The team has a mandated framework |
| CI provider and pipeline | The provider the organization already uses | No provider is detectable, or more than one is in use |
| Versioning and release model | Semantic versioning | The project is internal-only and never released |
| License | The organization's default | No default is known, or the project is proprietary |
| Repository visibility | Private until the user states otherwise | The project is open source or contains third-party-sensitive material |
| Secret and environment management | An environment file for local development, never a committed secret | The organization mandates a vault or secret manager |
| Documentation | A README with the stack, the architecture style, and the reproducible commands | Never |
| Internationalization | Only when the product has user-facing text in more than one language | The product has user-facing text, and the supported locales or RTL are not stated |

---

## Backend Decisions

### API surface

| Decision | Default | User decides when |
|---|---|---|
| API style | REST with JSON | The product is graph-oriented, needs subscriptions, or has a first-class contract shared with other systems |
| API versioning | URL path prefix, `/v1` | The organization standard differs, or the API is internal only |
| Documentation format | OpenAPI generated from the code and validated in CI | The organization mandates a different tool |
| Error model | One problem-details response shape for every error | The product exposes errors to end users with per-case handling |
| Request validation | Validate at the boundary, reject with the error model | Validation rules are domain invariants rather than shape checks |
| Pagination | Cursor pagination for collections | The data set is small, static, or the consumer is internal |
| Idempotency | Idempotency keys on every mutating endpoint | The endpoint is naturally idempotent, or a specific consumer needs a different contract |
| Rate limiting | Applied per consumer, with a documented default | The limit is contractual with a specific client |
| API versioning and deprecation policy | Documented deprecation window before removal | The organization has a published policy |

### Data and persistence

| Decision | Default | User decides when |
|---|---|---|
| Database engine | The engine the organization operates, or PostgreSQL | The engine is a contractual or licensing decision, or the workload is non-relational |
| ORM or query layer | The ecosystem standard, with explicit migrations | The domain has complex queries that the ORM cannot express |
| Migrations | Versioned, run in CI, never auto-applied to production | The organization uses a different migration tool |
| Schema strategy | Managed migrations owned by the team | Schema ownership is per bounded context or per service |
| Connection pooling | The managed service default | The deployment target requires a specific pooler |
| Transaction boundaries | One transaction per use case, owned by the application layer | A transaction must span more than one service |

### Identity and access

| Decision | Default | User decides when |
|---|---|---|
| Authentication | OIDC or OAuth 2.1 against an external identity provider | The product needs local accounts, or a customer identity provider is contractual |
| Authorization | Role-based, enforced in the application layer | The domain needs attribute-based, policy-based, or relationship-based access |
| Multi-tenancy | Single tenant | The product is multi-tenant, and the isolation strategy is a data decision |
| Service-to-service identity | Workload identity with short-lived credentials | The environment does not provide it |
| Secrets rotation | Delegated to the platform, with no rotation in application code | The application owns a credential it must rotate |

### Asynchronous work and integrations

| Decision | Default | User decides when |
|---|---|---|
| Background jobs | A durable queue with retries and a dead-letter path | The workload is not asynchronous |
| Cache | None until a measured need appears | The product has an explicit latency or cost target |
| Event streaming | None until a bounded context needs decoupling | The product integrates systems that cannot be called synchronously |
| File storage | Object storage with a signed URL flow | The product stores sensitive files that require different handling |
| Email and notifications | Delegated to a transactional provider | The organization has an internal delivery service |
| Third-party integrations | Behind a port in the domain layer, with a fake in tests | The contract is a hard external dependency with its own SLA |

### Operations

| Decision | Default | User decides when |
|---|---|---|
| Containerization | A multi-stage Dockerfile, non-root user, no dev dependencies in the runtime image | The deploy target does not use containers |
| Deployment target | The organization's platform | No platform is detectable, or the deployment model is a customer decision |
| Configuration | Environment variables with a documented schema | The product needs per-tenant configuration |
| Logging | Structured JSON with a request correlation id | The organization has a mandated log format |
| Metrics and tracing | The platform's standard instrumentation from the first endpoint | The organization mandates a different stack |
| Health checks | Liveness and readiness endpoints, plus dependency checks | The platform provides them as a managed feature |
| Graceful shutdown | Drain in-flight requests on termination | The workload is not request based |
| Backups and recovery | Managed by the platform, with a stated recovery objective | The product has a contractual recovery objective |
| Performance targets | Derived from the source analysis, expressed as an SLO | The analysis states no target and the product is user-facing |

---

## Frontend Decisions

| Decision | Default | User decides when |
|---|---|---|
| Framework and rendering mode | The framework in the source analysis, with its official rendering mode | The analysis states alternatives, or the SEO requirement decides the mode |
| Routing | The framework's file-based router | The application is a single embedded view |
| State management | Server state in the data layer, local state in the component, a store only when shared state demands it | The product has complex cross-screen state |
| Data fetching | The framework's server data layer, or the ecosystem's query client | The application is a real-time dashboard or an offline-first app |
| Styling approach | The framework's recommended approach with CSS custom properties | The design system is an existing in-house system |
| Component library | The organization's library, or none with local primitives | The visual language is a brand decision |
| Forms | The framework's form solution with schema validation | The forms are highly dynamic |
| Accessibility target | WCAG 2.2 AA | A different conformance level is contractual |
| SEO | The framework's metadata API, with a static rendering path | The application is not indexable |
| Analytics and error tracking | The organization's providers | None is detectable |
| Form validation approach | Schema validation at the form boundary, with the server validating again | The form rules are domain invariants rather than shape checks |
| Error and empty states | Defined per feature, with the shared boundaries in the shell | The product needs a specific pattern, such as an inline retry or a partial result |
| Offline and caching | The framework's data layer, offline only when the product requires it | The product must work without connectivity |
| Progressive enhancement and browser support | The browser support matrix of the framework, stated explicitly | The audience requires an older browser |
| Component library and design system | The organization's library, or local primitives with the design tokens | The visual language is a brand decision, already settled |

---

## Library and CLI Decisions

| Decision | Default | User decides when |
|---|---|---|
| Package format and entry points | The ecosystem standard, with one public entry point | The library exposes several independent entry points |
| Public API stability | Semantic versioning, with a documented stability policy | The library is pre-1.0 and used internally |
| Minimum supported version | The lowest version still supported by the toolchain | The audience needs an older runtime |
| Distribution | The ecosystem registry | The library is consumed only internally |
| Documentation generation | The ecosystem tool, published alongside the package | The API reference needs a dedicated site |
| Configuration surface | Constructor arguments over global state | The tool needs a config file |
| Error reporting | Typed errors returned to the caller | The tool is a CLI and reports to the exit code and stderr |

---

## Mobile Decisions

| Decision | Default | User decides when |
|---|---|---|
| Platform | The platform in the source analysis | The product targets both platforms with different capabilities |
| Navigation | The framework's navigator | The navigation model is product specific |
| Local storage | The platform secure store for credentials, an ordinary store for preferences | The data is large or offline-first |
| Push notifications | The platform push service | The product has no user notifications |
| Crash reporting | The platform service plus the organization's provider | The organization mandates a different provider |
| Release channel | The platform store, staged rollout for mobile | The organization uses an enterprise distribution channel |
| Device support matrix | The two most recent major versions of the platform | The audience is defined by a contracted device fleet |
| Bundle and package identifier | Derived from the product name, set once | The identifier is a contractual or store-facing decision |
| Minimum and target OS version | The minimum the platform still supports, targeting the current one | The audience is limited to older devices |
| Offline behavior | Read-only caching of the last known data | The product must be usable without connectivity, and the sync strategy is a product decision |
| Deep linking | The web URL scheme, mapped to the in-app routes | The link formats are contractual with an external system |
| Analytics and product analytics | The organization's provider, with the events the product actually needs | The event taxonomy is a product decision |
| Monetization | None | The product sells, and the purchase and entitlement model is a product decision |
| Push notification opt-in | Asked at the moment of value, with a preference to decline | The notification strategy is a product decision |

---

## Completion Check

Before creating the structure, confirm that every decision in the applicable
categories is resolved:

- [ ] Every decision from categories 1 to 3 is stated in the proposal.
- [ ] Every decision from category 4 was asked, with its recommendation.
- [ ] The architectural style was asked, and answered.
- [ ] No question was asked that the source analysis already answers.
- [ ] No decision was left implicit in a default that was never stated.
