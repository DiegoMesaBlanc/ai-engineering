# Audit PASS Criteria

## Purpose

This reference defines, for each of the 14 dimensions of the code audit, the
concrete and testable criteria that code we create must meet to be `PASS`.

It is the single source for both sides of the same judgment:

- the **implementer** uses it while planning (`workflows/task-to-code.md`,
  Phase 9) and before delivery (Phase 12), so that the change already meets it;
- the **auditor** uses it in `workflows/code-audit.md`, so that a change which
  meets it is judged `PASS`, and a change which does not is judged `FAIL`
  with the criterion it violates.

Source kind: a policy of this system, derived from the Skills named in each
dimension. A criterion is never presented as a framework standard.

## Judgment Rules

- A dimension is `PASS` when every applicable criterion holds on the change set.
- A criterion that is violated by the change set is a defect of the task. It
  is fixed in the task, or the task is `BLOCKED` with the reason. It is not
  delivered as a `WARN` or as an observation.
- `N/A` requires a concrete reason derived from the approved requirement
  (scope, non-goals, and the Phase 2 alignment), never from what the auditor
  notices in the code. Example: a textual i18n concern is `N/A` when the
  requirement does not ask for locales.
- `WARN` remains an auditor state for a non-defect observation that violates
  no criterion. It is not an accepted terminal state for code the task creates.
- A criterion violated by code that already existed before the change set is
  a recorded finding with file and line. It does not block the task.

## Criteria

### 1. Architecture & folder structure

- Dependencies point toward the inner layer: no consumer imports a concrete
  implementation that a port or contract should abstract.
- The composition root (the single place that wires concrete implementations)
  is the only module that knows the concrete types.
- Each module lives in the folder that owns its responsibility, following the
  Folder Architecture Map. No domain code in `lib/` or `utils/`.
- A type is in a dedicated contracts file when another file imports it; it is
  not re-exported by the module that consumes it.
- No hardcoded lists that a configuration or registry should own.

### 2. Patterns & complexity

- Every pattern in use is justified by a design force from
  `references/design-patterns.md` and is not already covered by an existing
  mechanism.
- No pattern is present without a signal, and no abstraction exists without
  a second consumer or a stated reason.
- Each mechanism that the design needs (retry, queue, dead letter, routing) is
  either explicit and named, or absent with a stated reason.

### 3. Code quality & separation (SRP)

- Each unit has one reason to change. A unit that renders, queues, retries,
  records dead letters, and logs is a violation.
- Errors are explicit: no swallowed exceptions, typed or documented error
  paths, and boundary input validated.
- No dead code, debug output, or unused exports.
- Names are clear and the code is typed; no `any` without a documented reason.

### 4. API & contracts

- Every boundary has an explicit contract (schema or typed port) documented
  where it is owned.
- The error model is explicit and stable.
- Responses that the change does not intend to alter remain unchanged, and
  the change does not leak internal types across the boundary.

### 5. Data & database

- Applicable only when the change touches schema, migrations, queries, indexes,
  or transactions. Otherwise `N/A` with that reason.
- Queries are parameterized, migrations are reversible or justified, no N+1
  pattern is introduced, and transaction boundaries are explicit.

### 6. Security

- No secret, token, or credential is committed or logged.
- Personal data is retained only as long as needed: any in-memory or
  persistent store that holds it has an explicit bound (size, time, or count).
- Input from outside the process is validated at the boundary.
- Authentication and authorization are not weakened.

### 7. Testing & validation

- Every behavior the change adds or modifies has a test that exercises it
  through its real entry point.
- Every error path the change introduces has a test: thrown error, failed
  render or dependency, and rejected input.
- Shutdown and resource-release behavior is tested when the change owns a
  resource (queues, connections, workers).
- Bounds (caps, limits, dead-letter size) are tested at the limit.
- Tests are deterministic and do not depend on wall-clock sleeps.

### 8. Performance

- Each external call or attempt has an explicit timeout.
- No blocking wait (for example `sleep`) on a shared worker stalls independent
  work; waits are scheduled per item or are non-blocking.
- Unbounded growth is not introduced in memory, queues, or caches.
- Applicable to hot paths and web delivery. Otherwise `N/A` with that reason.

### 9. Accessibility

- Applicable only to UI, web, or mobile surfaces that the change creates or
  modifies. Otherwise `N/A` with that reason.
- Semantic structure, keyboard operation, focus, labels, and contrast meet
  WCAG 2.2 A/AA for the changed surface.

### 10. SEO

- Applicable only to public web pages the change creates or modifies.
  Otherwise `N/A` with that reason.

### 11. Internationalization

- Applicable only when the approved requirement includes user-facing text that
  must support locales, RTL, or locale-aware formatting. Otherwise `N/A` with
  that reason.
- When applicable: text is externalized, plurals and formatting are
  locale-aware, and no locale is hardcoded.

### 12. Frontend/UI

- Applicable only when the change creates or reshapes UI. Otherwise `N/A`.
- The implementation matches the approved design context and is responsive.

### 13. Mobile

- Applicable only to mobile work. Otherwise `N/A`.

### 14. Organization boundaries

- Applicable only to multi-package or multi-project changes. Otherwise `N/A`.
- Changes stay inside the target project; package boundaries and dependency
  direction are respected.
