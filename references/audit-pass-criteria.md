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
- A criterion that the approved requirement does not settle is `AMBIGUOUS`:
  the auditor states the open question and does not judge it `FAIL` by its own
  interpretation, nor `PASS` by assumption. An `AMBIGUOUS` criterion is
  resolved by the user, not by the auditor.
- A dimension is judged only after this file has been read in full in the
  current audit. A dimension judged without that read is `NOT VERIFIED` with
  the reason, never `PASS` or `FAIL`.

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
- A failure raised by a module uses the module's error model (its typed error
  class). A generic error crossing a module boundary is a violation unless the
  module has no error model and that is stated.
- A secondary (optional) dependency that fails does not change the outcome of
  the primary operation: a failed read from it is a miss, and a failed write
  to it is skipped and recorded. A write already committed in the primary store
  is never reported as failed because a secondary store failed.
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
- Bounds (caps, limits, dead-letter size) are tested at the limit, through the
  real entry point, asserting the defined behavior at the cap.
- Each secondary dependency is tested with that dependency failing, and the
  primary result is asserted.
- Every guarantee stated in the documentation the change creates or modifies
  (bounds, consistency, timeouts, external-system behavior) is backed by a test
  or an executed check. An external-system claim without evidence is
  `NOT VERIFIED` and must be stated as an assumption.
- Tests are deterministic and do not depend on wall-clock sleeps.
- Platform-dependent behavior has an assertion on its rendered result for each
  branch: safe-area insets, orientation or viewport-height branches (for example
  a compact layout below a height threshold), and keyboard avoidance. A test
  helper that injects a value without asserting its effect is not coverage.

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
- Error text is associated with its field through a programmatic relation: an
  error role, a label, or the platform's equivalent. An accessibility hint or
  description is never used to carry error text, because it describes an
  action, not a state.

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
- Every input declares its keyboard type, its return-key action, and its
  dismissal path. A keyboard type without a return key (for example a number
  pad on iOS) has an explicit dismissal path that the code provides; the
  absence of a return key is never a reason to leave the keyboard without one.
- Safe areas, touch targets, and layouts for each supported orientation are
  implemented in code. Where the behavior depends on a device value that the
  test environment cannot produce, the check is `NOT VERIFIED` on device, with
  that reason, and is not reported as `PASS`.

### 14. Organization boundaries

- Applicable only to multi-package or multi-project changes. Otherwise `N/A`.
- Changes stay inside the target project; package boundaries and dependency
  direction are respected.
