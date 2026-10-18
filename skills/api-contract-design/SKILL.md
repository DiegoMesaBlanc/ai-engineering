---
name: api-contract-design
description: Design and evolve APIs and integration contracts (REST, GraphQL, gRPC, events) with explicit schemas, error contracts, versioning, and compatibility guarantees.
---

# API Contract Design

## Purpose

Define the interface between systems as an explicit contract before
implementing it, so clients, servers, and integrations evolve without
silent breakage.

The contract is the source of truth for consumers; implementation follows
it, not the other way around.

## When to Apply

Apply when a task:

- adds or changes an endpoint, resolver, RPC method, or message/event
- defines an integration with an external or internal service
- changes an existing request, response, error, or payload shape
- introduces asynchronous communication (queues, topics, events, webhooks)

Do not design a contract for a purely local refactor that does not cross a
process or module boundary, unless a public interface is being introduced.

## Detect Before Designing

Inspect the real project first:

- existing API style, conventions, and naming
- existing schema or specification files (OpenAPI, AsyncAPI, `.proto`,
  GraphQL SDL, JSON Schema)
- existing error format, status-code usage, and pagination conventions
- authentication and authorization mechanisms already in use
- existing serialization, validation, and versioning approach
- existing clients and generated types

Reuse the established contract style. Do not introduce a second style
without a justified reason and human approval.

## Define the Contract

Specify explicitly:

- operations: purpose, inputs, outputs, and side effects
- request and response schemas with types, required/optional fields,
  nullability, and formats
- identifiers, pagination, filtering, sorting, and idempotency semantics
- status and error model, including stable machine-readable error codes
- validation rules and boundary conditions
- authentication, authorization, and scopes per operation
- rate limits, timeouts, and retry semantics where relevant
- security-sensitive fields and data classification (never log secrets)

Prefer an explicit machine-readable specification over prose when one is
available for the detected stack.

## Style Guidance

Choose the style the project already uses or that best fits the problem;
do not impose a style by preference.

- REST: resource-oriented, correct HTTP semantics, consistent status codes,
  explicit content types.
- GraphQL: schema-first types, explicit nullability, deliberate use of
  queries versus mutations, pagination conventions.
- gRPC / RPC: explicit message and service definitions with stable field
  numbering.
- Events and messaging: explicit message schema, event naming, delivery
  guarantees, ordering, idempotency, and dead-letter handling.

## Error Contract

Define errors as part of the contract, not as an afterthought:

- a consistent envelope or convention across operations
- stable, documented error codes distinct from human messages
- correct use of transport status codes
- no internal details, stack traces, or secrets leaked to clients
- explicit handling of validation, authorization, not-found, conflict, and
  rate-limit conditions

## Versioning and Compatibility

- Prefer additive, backward-compatible evolution over breaking changes.
- Treat a change as breaking when it removes or renames a field, changes a
  type or meaning, tightens validation, or changes an error contract.
- When a breaking change is unavoidable, define an explicit versioning or
  deprecation strategy and migration path.
- Keep deprecated operations available for a defined window with clear
  deprecation signals.
- Never change the meaning of an existing field silently.

## Security

- Apply authentication and authorization per operation; default to deny.
- Never place secrets, tokens, or sensitive personal data in URLs, logs, or
  error messages.
- Validate and constrain all inputs at the boundary.
- Reference the project's existing security mechanisms instead of inventing
  a parallel one.

## Contract Testing

The contract enables verification:

- validate payloads against the schema in tests
- verify consumer expectations (contract tests) when consumers exist
- cover error responses, boundary values, and authorization failures
- keep generated types or clients in sync with the specification

## Relation to Other Components

- `architecture-advisor` decides boundaries, layering, and where the
  contract belongs.
- `dependency-assessment` governs any new specification or code-generation
  tool.
- `implementation-recipes` implements the server and client sides.
- `testing-strategy` selects the depth of contract and integration tests.

## Decision Gates

Introduce a new API style, a breaking change, a new public interface, or a
new specification/code-generation dependency only with human approval when
it materially affects scope, compatibility, or cost.

## Output

### API CONTRACT

- Scope and operations
- Style and conventions (detected or chosen, with reason)
- Schemas (request/response/messages)
- Error model
- Auth and authorization per operation
- Versioning and compatibility impact
- Validation and boundary conditions
- Testing approach
- Affected consumers and migration notes
- Decision gates

## Safety

- Never invent endpoints, fields, or consumers that evidence does not
  support.
- Never expose secrets, tokens, or sensitive data in the contract or its
  examples.
- Prefer the existing contract style; a new one requires justification and
  approval.
