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

- A dimension is `PASS` when every applicable criterion holds on the change set,
  and each criterion's `PASS` cites its evidence: a command with its exit code, a
  `file:line`, or a measurement. An assertion without cited evidence is not a `PASS`;
  it is `NOT VERIFIED` (M-75).
- A PASS in a verdict matrix carries its evidence: a `file:line`, a detector ID with its
  output, a command in backticks, or its exit code. A PASS row with no such evidence is a
  `FAIL` of `REPORT-001` (M-98), and a dimension that cites no detector it ran is not
  verified by a detector, whatever its status says.
- A dimension with a criterion `AMBIGUOUS` is never `PASS`. It stays open until the user
  resolves the criterion (M-75). A detector or check that a workflow defines is a control only when the procedure that a verdict cites runs it on the scope it covers (M-75).
- A criterion that is violated by the change set is a defect of the task. It
  is fixed in the task, or the task is `BLOCKED` with the reason. It is not
  delivered as a `WARN` or as an observation.
- `N/A` requires a concrete reason derived from the approved requirement
  (scope, non-goals, and the Phase 2 alignment), never from what the auditor
  notices in the code. Example: a textual i18n concern is `N/A` when the
  requirement does not ask for locales.
- `WARN` remains an auditor state for a non-defect observation that violates
  no criterion. It is not an accepted terminal state for code the task creates.
- A claim the change makes about performance or a technical property is evidence,
  not an assertion (M-65): it is `PASS` only with the measurement or the plan that
  decides it, and without it the dimension is `FAIL` at the line of the claim.
- A criterion violated by code that already existed before the change set is
  a recorded finding with file and line. It does not block the task, unless the
  change set consumes a pre-existing unit which violates a requirement the change
  states: then the consumed unit is part of the change set and the task is
  `BLOCKED` until it is corrected (M-67). `CONSUME-001` in
  `workflows/output-compliance.md` reports the inherited defect with both lines. A consumer that renders the
  violating unit inherits its violation, so the consumer is `FAIL` as well.
- A criterion that the approved requirement does not settle is `AMBIGUOUS`:
  the auditor states the open question and does not judge it `FAIL` by its own
  interpretation, nor `PASS` by assumption. An `AMBIGUOUS` criterion is
  resolved by the user, not by the auditor.
- A `NOT VERIFIED` verdict on an applicable dimension blocks delivery (M-39). A
  dimension whose review was not executed is run before its verdict is written;
  "no review was made" is never a `NOT VERIFIED` reason. A device-only limit is
  the reason for a `NOT VERIFIED` that stays after the review ran.
- A dimension is judged only after this file has been read in full in the
  current audit. A dimension judged without that read is `NOT VERIFIED` with
  the reason, never `PASS` or `FAIL`.
- A verdict matrix is consistent with its own rules (M-80). Every dimension has its
  own row with its own reason, and a grouped row (`9, 10, 12, 13`) is not a verdict
  for each of them. A `PASS` row that names an open item (pending, not run, an
  equivalent runtime or environment that was not executed, or a diff-only read) is a
  defect of the matrix, and the dimension is reopened. A red whole-project gate with
  no failing file in the change set is `BASELINE_FAILURE` in the report, with its
  count and first location, and never a `PASS` of that gate. `REPORT-001` in
  `workflows/output-compliance.md` checks the matrix mechanically and is run before
  the report is delivered (`workflows/system-audit.md`, Phase 5).

- A generated proposal, pull request, or report is judged on its statements of
  fact: every count, name, removed or added artifact, advisory identifier, and
  version is derived from the evidence it cites and re-derived before it is
  reported. A statement copied from memory or from an earlier draft is a finding
  of the report, `FAIL` for the dimension that owns the claim (M-65).
- A required step that the agent does not execute, such as a teardown, reset,
  cleanup, or rollback, is `NOT DONE` in the report with its reason and is put to
  the user. An end state reached by a side effect is not completion: the step's own
  evidence verifies it, or the user accepts the equivalence in the current
  conversation (M-74, `workflows/task-to-code.md`, Phase 13).

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

- A concrete adapter is constructed only in the composition root (AGENTS.md,
  the dependency rule). A hook, component, or service that calls a `create*`
  factory or `new`s an infrastructure class is a DIP `FAIL` with file and line.
- A platform capability with a side effect (system settings, a permission request,
  a system alert, an accessibility or device state) is reached through a port. A
  hook, service, or feature-logic module that imports it from the platform module
  (for example `Linking` or `PermissionsAndroid` from `react-native`) is a DIP
  `FAIL` at the file and line of the import (`DIP-002` in
  `workflows/output-compliance.md`). A component that renders only platform
  primitives is exempt. A pre-existing violation is a finding, not a blocker, for
  a task that does not touch it.
- A concrete module is imported only by the composition root (the concrete-module
  rule of M-96). Every non-composition file that imports an
  adapter, store, policy, or `infrastructure/` module by a relative or aliased path is
  a DIP `FAIL` at the file and line, detected by `DIP-001` in
  `workflows/output-compliance.md`. A decision record whose DIP row says APPLIES or
  PRESERVED while `DIP-001` reports a finding is a `FAIL` of `DEC-001` (M-96). The
  Dimension 1 verdict cites the `DIP-001` run, never a reading of the record.

### 2. Patterns & complexity

- Every pattern in use is justified by a design force from
  `references/design-patterns.md` and is not already covered by an existing
  mechanism.
- No pattern is present without a signal, and no abstraction exists without
  a second consumer or a stated reason.
- A decision record that claims a human approval quotes the verbatim user message and its
  turn (M-92). Decision records in the change set and in the sweep run through
  `approval_citation_check` (`APPROVAL-001` in `workflows/output-compliance.md`); an
  uncited approval is `NOT VERIFIED` and blocks the gate that depends on it.
- An SRP row names the unit and each responsibility it keeps, and every consumer
  claim of a design record is traced to the code (M-78). An SRP row with no `file:line` is a `FAIL` of this dimension (`DEC-001`).
  The auditor then reads the cited unit and lists the responsibilities it keeps: a
  unit that renders, queues, retries, classifies errors, and logs, or that owns a
  transaction and its retry policy together, is an SRP violation even when its row
  says `APPLIES`. A selected pattern with a single consumer is rejected, not kept as
  an observation; a stated reason such as testability or size is not a design force,
  so it does not keep a single-consumer pattern. A signal present at several sites is
  applied to each of them, or every site not converted is listed with its reason.
- Each mechanism that the design needs (retry, queue, dead letter, routing) is
  either explicit and named, or absent with a stated reason.
- A significant, hard-to-reverse decision that meets the recording threshold of
  `skills/architecture-advisor/SKILL.md` ("Decision Records") is recorded: in
  the project's existing records convention, or, when the project has none, in
  the presentation record shown before implementation. A decision that meets the
  threshold and has no record is a `FAIL` of this dimension, never an implicit
  approval and never `NOT VERIFIED`: the auditor checks the change set for the
  record, and its absence is the verdict. The same applies to a selected design
  pattern that has no record of its forces: its record is in the change set
  (M-50).
- A `file:line` that a design record cites resolves to the code it describes: the
  file exists, the line is inside it, and the identifier the row names appears
  within five lines of it (`DEC-002` in `workflows/output-compliance.md`). A citation
  that points to another line of the same file is a `FAIL` of this dimension with
  the cited line and the line where the identifier is found.
- The record in the change set names every selected pattern by name with its
  force and stack translation, and it has the five SOLID rows (SRP, OCP, LSP, ISP,
  DIP) with their status. A record that states a count alone, or lacks a SOLID row,
  is a `FAIL` of this dimension at the file and line of the record (`DEC-001` in
  `workflows/output-compliance.md`, M-50 and M-45).
- Every `APPLIES` or `PRESERVED` row of the record, and every claim in it that a
  bound is applied, a port is received, or a dependency is kept out of a unit, is
  checked against the cited unit's imports and calls (M-65). A claim the code
  contradicts is a `FAIL` of this dimension at the line of the record; a note that
  is written from the design intent instead of the code is never accepted as
  evidence.

### 3. Code quality & separation (SRP)

- Each unit has one reason to change. A unit that renders, queues, retries,
  records dead letters, and logs is a violation. A unit with more than one reason to
  change is an SRP violation even when its design record says otherwise (M-107): the
  candidates are listed by `SRP-001` in `workflows/output-compliance.md` with their
  `file:line`, the reviewer confirms each listed unit, and the decision record names
  every listed unit (`srp_record_check`). A listed unit the record does not name is a
  `FAIL` of this dimension.
- A helper repeated in two or more units is a duplicate (M-105): a change that
  introduces a copy of a helper that another unit already declares promotes the helper
  to the shared unit that owns it and imports it from there. `DUP-001` reports each copy with its
  `file:line`; a copy in a changed file is a `FAIL`, and a duplicate that existed in
  the base revision is a finding that does not block the task.
- Errors are explicit: no swallowed exceptions, typed or documented error
  paths, and boundary input validated.
- A `catch` that converts a failure into a fallback value, state, or default
  records the cause through the project's logger or error model. Discarding the
  cause while keeping the fallback is a violation, even when the user-facing
  behavior is correct. Sibling units of the same kind use the same logging
  convention, and a deviation from it is a violation.
- A failure raised from a module uses the module's typed error class, never a
  generic `Error`, unless the project has no error model and that is stated (M-81).
  A project has an error model when a non-test source file in any folder declares a
  class that extends `Error`; the error model is a project property, so a module
  whose folder holds no error file is still bound by it. A generic error raised in
  any module of such a project is a `FAIL` of this dimension, and `ERR-001` in
  `workflows/output-compliance.md` reports it with `file:line`. A reviewer does not
  accept "the message is correct" as the justification for a bare `Error`.
- A boundary input is validated before any side effect (M-83): an entry point that
  opens a file, a connection, a database, or a network resource before it validates
  its arguments leaves an artifact for a rejected invocation, and that is a `FAIL` of
  this dimension at the line of the first resource opened before the validation. The
  auditor traces each validator to its call and reports the order. `VAL-001` in
  `workflows/output-compliance.md` lists the candidates; an opened resource whose
  validation is in another file is `NOT VERIFIED` until the auditor traces the call.
- A secondary (optional) dependency that fails does not change the outcome of
  the primary operation: a failed read from it is a miss, and a failed write
  to it is skipped and recorded. A write already committed in the primary store
  is never reported as failed because a secondary store failed.
- No dead code, debug output, or unused exports. An export the change adds is
  imported by another module; a barrel that re-exports it is not a consumer. An
  export with no consumer is a `FAIL` with file and line (`DEAD-EXPORT-001` in
  `workflows/output-compliance.md`, M-49).
- A unit that the change extends with a responsibility outside its declared one
  (for example, a root component that also decides routing, session, and banner
  wrapping) extracts that responsibility into its own unit in the same change
  set. The auditor names the responsibilities of each unit the change touches; a
  unit with a second reason to change is a `FAIL` of SOLID SRP, not a `WARN`.
- Names are clear and the code is typed; no `any` without a documented reason.
- A comment, JSDoc, README, or document that claims a universal or exclusive scope
  (`only`, `every`, `each`, `all`, or the equivalent in its language) is verified by
  the verifier enumerates every element of the scope with a search, and the verdict
  cites the search (M-68). A claim the enumeration contradicts is a finding with both lines, and
  a claim that cites an artifact the repository does not hold is `NOT VERIFIED`
  (`CLAIM-001` in `workflows/output-compliance.md`).
- A correction to a claim, a contract, or a pattern is applied to every surface that
  states it. The change set and the project sweep are searched for the old text, and a
  surviving occurrence in a source comment or a document is a finding with its
  `file:line`, even when the test or the contract was already corrected (M-75,
  `CLAIM-001` in `workflows/output-compliance.md`).
- A comment, JSDoc, or document that states a behavior of the code, a workflow,
  or a pipeline is checked against the code it describes. A statement the code
  contradicts is a finding: `P3` when it has no runtime effect, `P2` when it
  misleads a gate, a security boundary, or a pipeline decision. The auditor cites
  the comment line and the line that contradicts it. An ordering or traversal claim
  in a changed comment is listed by `COMMENT-001` in `workflows/output-compliance.md`
  and stays `NOT VERIFIED` until the reviewer cites the code and the return type that
  decide it.
- A contract name declared in more than one file of the change set is a `FAIL` of
  the Type/contract ownership check (`TYPE-004` in `workflows/output-compliance.md`)
  unless the declarations have different consumers and the reason is recorded.

- A failure handled by failing closed reaches the user as a visible state that
  is named and recoverable (M-64). A `failed` result mapped to `idle` or `retrying` is a
  `FAIL`. A terminal result of a queued operation releases what it blocked.
- A promise or a result of a failed load is not memoized: the next request
  retries it, and a permanent failure is reported as permanent.
- A documented timing, size, or limit has a test, and its unit matches the unit
  the code measures. A comment that claims a unit the code does not measure is
  a `FAIL` of the documentation (M-65 and the comment-accuracy rule). A technical
  property in a comment (a covering index, an index-only scan, a constant-time
  operation) is checked against the plan or the code that decides it, and it is
  `NOT VERIFIED` until that evidence is cited (`CLAIM-001`).
- A listener, subscription, or controller created for an identity is released
  when the identity changes or signs out.

- A changed value of existing behavior (a timeout, a limit, a retry budget, a default,
  a pool size, a threshold) is inside the approved scope only when
  the scope manifest names it (M-79). `SCOPE-002` reports each candidate changed value
  with `file:line`; a candidate with no manifest line is a `FAIL` of this dimension,
  whether or not the file is in scope, and a change to a value the requirement did
  not name blocks delivery until the user approves it.

### 4. API & contracts

- Every boundary has an explicit contract (schema or typed port) documented
  where it is owned.
- The error model is explicit and stable.
- The error codes are declared in both directions (M-77): every error code a
  boundary emits is declared in the contract that names it (`ERR-003` in
  `workflows/output-compliance.md`), and every declared code has an executing test
  (`ERR-002`). An emitted code with no declaration, or a declared code no test
  asserts, is a `FAIL` of this dimension at the file and line of the code and of the
  declaration.
- Responses that the change does not intend to alter remain unchanged, and
  the change does not leak internal types across the boundary.

### 5. Data & database

- Applicable only when the change touches schema, migrations, queries, indexes,
  or transactions. Otherwise `N/A` with that reason.
- Queries are parameterized, migrations are reversible or justified, no N+1
  pattern is introduced, and transaction boundaries are explicit.
- A database change is compatible in either deployment order (M-69): the new
  release starts on a database that has not received its migration, and the
  previous release keeps working on the migrated one. A migration with no test
  that executes both orders is `NOT VERIFIED`, and a `FAIL` when the change set
  states order-safety with no such test (`DB-002` in
  `workflows/output-compliance.md`).
- A paginated or cursor-based read that issues several statements runs in one read
  transaction or snapshot (M-84). A page loop with no read transaction or snapshot
  is a `FAIL` of this dimension at the line of the loop (`DB-003` in
  `workflows/output-compliance.md`), because a row changed between pages can be
  counted twice or skipped. A claim that the result is consistent is `NOT VERIFIED`
  until a test changes a row between pages and asserts the count.
- A query predicate on an application table is served by an index whose leading
  column is the predicate's column (M-85). A predicate with no such index is a `FAIL`
  of this dimension at the line of the statement (`DB-004` in
  `workflows/output-compliance.md`). A statement with a JOIN, whose table the detector
  cannot resolve, is `NOT VERIFIED` until a reviewer confirms its table, and an index
  claim stays `NOT VERIFIED` until the plan confirms it.
- Application code never changes the schema (M-70). A `CREATE`, `ALTER`, or
  `DROP` at startup, boot, or in a request path is a `FAIL` of this Dimension at
  the file and line of the statement (`DB-001` in `workflows/output-compliance.md`).
- An inventory of stores, tables, files, or commits in a report is produced by an
  enumeration of the target in the same session (M-72). The report cites the
  enumeration command and its count with its definition. A listed name the
  enumeration does not return is a `FAIL`, and an enumerated item that is neither
  listed nor excluded with a reason is a `FAIL` (`INV-001` in
  `workflows/output-compliance.md`).
- A performance claim (a query, index, or path said to be faster or fixed) is
  `PASS` only with a before-and-after measurement of the named query: its plan
  (`EXPLAIN`) and its timing, recorded before and after the change, on a dataset of
  stated size and environment (M-65). A claim with no measurement is a `FAIL` at the
  line of the claim, and a production claim with no production measurement is
  `NOT VERIFIED` for that part. A development measurement does not establish a
  production effect.
- When the requirement does not identify which query is slow, the criterion is
  `AMBIGUOUS`: the auditor does not choose a query and measure that one instead.
  The user identifies the query (see the `AMBIGUOUS` rule above).
- A technical property of a query or structure (a covering index, an index-only
  scan, a constant-time operation) stated in code, a comment, or a document is
  checked against the plan or the code that decides it (`CLAIM-001` in
  `workflows/output-compliance.md`). A property the plan contradicts is a `FAIL` of
  this dimension with both lines.
- A change that alters how values already stored are read, normalized, or ordered
  states the treatment of the existing records: migrated, backfilled, or accepted as
  a limit, with the number of records it affects. The treatment is a decision of the
  user when the requirement does not settle it, so the criterion is `AMBIGUOUS` and
  the dimension stays open. A change that corrects only the rows written after it,
  while the legacy rows keep the defect, is never `PASS`, and a conclusion that calls
  the defect closed is a `FAIL` of this dimension (M-80, `REPORT-001` in
  `workflows/output-compliance.md`).

### 6. Security

- No secret, token, or credential is committed or logged.
- Personal data is retained only as long as needed: any in-memory or
  persistent store that holds it has an explicit bound (size, time, or count).
- Input from outside the process is validated at the boundary.
- A recursive removal or an overwrite of a path the user supplies (an option such as
  `--scratch`, `--out`, or `--target`) operates only inside a subdirectory the program
  creates for the run, removes nothing else, and rejects a supplied path that equals or
  contains a data, backup, manifest, or source path before any side effect (M-103).
  `DEL-001` in `workflows/output-compliance.md` reports the removal with its `file:line`.
  A program that removes a supplied path is a `FAIL` of this dimension until the overlap
  guard is shown by a test that passes the overlapping path and asserts that nothing was
  removed or overwritten.
- A value a parser returns from external input is typed before it is dereferenced
  (M-104): `null`, a scalar, or an array where an object is required is rejected by the
  validator of that input, and an invalid source maps to the input-error status, never to
  the verification-mismatch status. `PARSE-001` in `workflows/output-compliance.md` reports
  each dereference with its `file:line`, and a test that passes `null` asserts the status.
- A secret or content scan that excludes inputs by extension, path, or size
  reports every input it did not read as `NOT VERIFIED`, never as clean (M-51).
  A scan that reports clean without listing its exclusions is a `FAIL` of this
  Dimension when an excluded input could hold a secret.
- A validator, parser, sanitizer, or guard the change creates or exports is a
  control only when every external input it guards reaches it on the path from
  the trust boundary to the sink (M-48). The auditor follows each exported control to its call sites from the
  entry point. A control with no reachable call is an unwired control: the
  boundary is unprotected, the Dimension is `FAIL` at the severity of the
  boundary, and any document that states the boundary is validated is a false
  claim to be corrected or removed in the same change set.
- Authentication and authorization are not weakened.
- In a project with an identity mechanism, every entry point the change creates
  or touches identifies its caller only from a credential the client cannot
  forge (M-52). An identity read from a header, query string, or body the client
  controls is a `FAIL` of this Dimension at the severity of the boundary. Each
  object the entry point reads or writes is checked for ownership, a check that
  finds no record fails closed, and the check and the write are atomic. A
  project with no identity mechanism is `N/A` for this bullet, with that reason.
- An access gate applies exactly to the paths that the approved requirement
  names. A path the requirement does not gate is not gated by default. When the
  requirement does not settle which paths are gated, the scope is `AMBIGUOUS`
  and is resolved by the user before delivery.
- Development-only entry points (debug routes, development URL schemes or hosts,
  dev tooling endpoints) are enabled only in development builds, through the
  build or environment configuration, and are never reachable in production
  builds. A broader acceptance rule than the production surface needs is a
  finding.
- Production configuration never sends a credential or session data over
  cleartext (M-55). A base URL of an endpoint that receives a credential is
  validated by its scheme, and `http://` is rejected for every non-local build;
  a build that accepts it is a `FAIL` of this Dimension at the severity of the
  credential exposure.
- Client-side cached data is partitioned by the identity that owns it and
  cleared when that identity signs out (M-56). A cache key without the user,
  tenant, or account is a `FAIL` of this Dimension when the device can hold data
  of more than one identity. Personal and payment data kept in plain storage is a
  `FAIL` of this Dimension; the auditor verifies the sign-out path that clears
  the partitions by tracing it from the sign-out action to the storage calls.
- A dependency remediation never downgrades (M-57). A change that moves a
  dependency to an older major version to clear an audit finding is a `FAIL` of
  this Dimension unless the approved requirement names the downgrade and its
  reason.
- Project code and scripts carry no hardcoded environment endpoint and no fixed
  shared temporary path (M-58). A `FAIL` of this Dimension names the file and line
  of the literal.

- A request path built from a value the client supplies is encoded with the encoder of its context (M-59). A raw interpolation of an identifier into a path is a `FAIL` of this Dimension at the file and line of the template literal.
- A `401` answer ends the session: a rejected credential ends the session, the client clears that identity's cache partitions and returns to sign-in, and cached personal or payment data is never served to it (M-60). A `401` that leaves the session active is a `FAIL` of this Dimension at the severity of the data exposure.
- An application that holds personal or payment data does not allow backup of personal data (M-61): the build manifest or configuration sets the backup off. A build that enables backup is a `FAIL` of this Dimension; when no build exists the bullet is `NOT VERIFIED`, never passed.
- Cross-origin access and development tools (M-62): a wildcard origin is never combined with credentials, and the allowed origins are an explicit list. A wildcard origin with credentials, or a development tool enabled unconditionally, is a `FAIL` of this Dimension at the file and line of the setting. A conditional tool is read to its production-off check.
- A file that holds personal or sensitive data is written with an owner-only file mode, and an uploaded image is stored with its metadata stripped (M-63). A write with the default file mode is a `FAIL` of this Dimension at the file and line; an upload path with no metadata handling is `NOT VERIFIED`, never passed.
- Absence claims are verified on the complete path (the whole call, middleware chain, or handler) before a finding is `confirmed`; an absence that cannot be verified this way is `needs_validation` with no severity.

- An entry point that reads or writes an object by identifier, in a project with
  an identity mechanism, and receives no verified identity at all, is a `FAIL`
  of M-52, the same as a forged identity.
- A project that caches identity-owned data has a sign-out path that clears
  every partition of that identity (M-56). The absence of that path is a `FAIL`.

- A change that grants, widens, or adds a permission, role, scope, or access rule
  that the approved requirement does not name is a human decision gate (M-86). It is
  a `FAIL` of this dimension at the line of the grant (`AUTH-002` in
  `workflows/output-compliance.md`) until the user approves it in the current
  conversation. A grant the requirement names is `PASS` with the requirement cited,
  and a grant that exposes personal or payment data to a role that had no access to
  it is a finding that names the data it exposes.

### 7. Testing & validation

- Every behavior the change adds or modifies has a test that exercises it
  through its real entry point.
- Every error path the change introduces has a test: thrown error, failed
  render or dependency, and rejected input.
- An entry point of the project (a command-line program, a `bin` script, or a
  process `main`) is tested through its real invocation (M-82): the arguments, the
  exit status, and the output are asserted by a test that runs it. A documented exit
  code, flag, or argument error with no executing test is a `FAIL` of this dimension
  at the document that names it and the test file that should cover it. `ENTRY-001`
  in `workflows/output-compliance.md` reports an entry point that no test invokes, or
  whose exit status no test asserts. A test that imports a helper and never runs the
  entry point does not cover it.
- Every access gate the change adds or modifies has a test for each side of its
  scope: a path it gates is blocked without the credential, and a path the
  requirement leaves ungated is asserted to work without the credential. A gate
  tested only on the gated path is not covered.
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
- A guarantee that an external call is bounded by a timeout has a test in which the
  dependency never resolves, under controlled time, and the terminal state the user
  sees is asserted (M-54). A test that makes the dependency return a timeout result
  does not cover the guarantee: it proves the dependency, not the bound. A
  missing one is a `FAIL` of this Dimension at the test file that claims the bound.
  The bound is any declared wait, not only an HTTP timeout: a database busy or lock
  wait is covered too, and so is a retry budget (a number of attempts or retries,
  M-54): a test holds the dependency until the budget is spent under controlled time
  and asserts the number of attempts and the terminal state. `BOUND-001` in
  `workflows/output-compliance.md` lists the declared bounds that no test holds, and
  that finding is `FAIL` of Dimension 7.
- Every compatibility scenario the change declares has a test that executes it,
  for each deployment order (M-69): a schema change has a test in which the new
  code runs against a database without the migration and a test in which the
  previous code runs against the migrated database. A declared scenario with no
  executing test is a `FAIL` of this Dimension at the test file that claims it,
  or at the document that declares it when no test exists.
- Tests are deterministic (M-76). A test or test-support helper reaches its state
  through a controlled clock or an event, never through a real timer hold: a timer that
  holds a lock, a connection, or a resource while a retry budget expires is a `FAIL`
  of this dimension at each `file:line` (`TEST-003`), even when the test passes on the
  reviewer's machine, because it depends on load and cannot prove the bound it claims.
- Every value that a public input's type admits and that the implementation branches
  on or dereferences (`null`, `undefined`, an empty result, a generic callback whose
  return type admits them) has a test that passes that value through the real entry
  point and asserts the defined behavior (M-77). The auditor enumerates the parameter
  and return types of the changed public surface to find the admitted values; a value
  the code dereferences with no test is a `FAIL` at the test file that should cover it.
  Mechanical detection of this is not available, so the enumeration is the evidence,
  and the verdict states that it is an auditor judgment.
- Every error outcome that a contract or decision record names (its error type, an
  aggregate error, a terminal state) has an executing test that asserts that exact
  outcome (M-77). A documented error path with no test is a `FAIL` of this dimension at
  the document that names it and the test file that should cover it.
- When the approved requirement includes right-to-left support, each
  direction-dependent unit the change creates or modifies has a rendered assertion
  under an RTL locale in its own test (M-66). A project-level RTL test, or a test of
  another unit, does not cover it, and a claim of coverage per unit is verified by
  listing the units and their test files (M-68). `I18N-003` in
  `workflows/output-compliance.md` reports a unit without that assertion.
- Platform-dependent behavior has an assertion on its rendered result for each
  branch: safe-area insets, orientation or viewport-height branches (for example
  a compact layout below a height threshold), and keyboard avoidance. A test
  helper that injects a value without asserting its effect is not coverage.

### 8. Performance

- Every external call has an explicit timeout (M-54) for every project type,
  mobile and frontend included, and each attempt of a retry loop has its own
  timeout. This bullet is never `N/A` for a project that makes an external call.
  Its `PASS` requires the controlled-time test of Dimension 7 (M-54, M-76): a test that
  holds the dependency with a real timer, or that is only a timeout result returned by
  the dependency, does not establish the bound, and the bullet is then `FAIL` of
  Dimension 7 and `NOT VERIFIED` here, never `PASS`.
  A call with no timeout is a `FAIL` of this Dimension at the file and line of
  the call.
- A timeout declared as a value (a `timeoutMs`, `timeout`, or `deadline` option,
  field, or constant) is applied: the auditor traces the value from its
  declaration to the timer or abort that enforces it. A declared value that no
  code applies, or that a helper receives and ignores, is a `FAIL` of this
  Dimension at the file and line of the declaration, whatever the call's result
  type says.
- The waits and growth bullets apply to hot paths and web delivery; elsewhere
  they are `N/A` with that reason.
  - No blocking wait (for example `sleep`) on a shared worker stalls independent
    work; waits are scheduled per item or are non-blocking.
  - Unbounded growth is not introduced in memory, queues, or caches.

- A cache keyed by entity identifier has an explicit entry bound or an expiry
  that is checked on read. Without it the dimension is `FAIL`.

- A performance criterion is `PASS` only against a measurable metric and a threshold
  in the approved requirement (M-88). When the requirement names performance without
  either, the criterion is `AMBIGUOUS` and stays open until the user supplies them; the
  auditor does not pick a metric. A benchmark of a different path, such as a currency
  formatting benchmark offered for a payment-listing requirement, is not evidence for
  this criterion.

### 9. Accessibility

- Applicable only to UI, web, or mobile surfaces that the change creates or
  modifies. Otherwise `N/A` with that reason.
- Semantic structure, keyboard operation, focus, labels, and contrast meet
  WCAG 2.2 A/AA for the changed surface.
- Error text is associated with its field through a programmatic relation: an
  error role, a label, or the platform's equivalent. An accessibility hint or
  description is never used to carry error text, because it describes an
  action, not a state.

- A live announcement is implemented for each platform. On React Native,
  `accessibilityLiveRegion` is Android-only; the iOS path is
  `AccessibilityInfo.announceForAccessibility` or an equivalent. A change that
  sets only the Android property is `NOT VERIFIED` for iOS, never `PASS`. The
  detector `A11Y-001` in `workflows/output-compliance.md` reports every such
  file, so the platform gap is never absent from the verdict.
- Every state branch of a runtime status surface (error, denied, blocked,
  unavailable, or any state the change renders as text) is enumerated, and each
  one that changes at runtime carries the live region of its platform:
  `accessibilityLiveRegion` on Android, and the announcement on iOS. A state
  branch that renders without its platform live region is a `FAIL` of this
  Dimension at the file and line of the branch, and an announcement with no
  Android path in the change set is `A11Y-002` (`NOT VERIFIED` for Android).

### 10. SEO

- Applicable only to public web pages the change creates or modifies.
  Otherwise `N/A` with that reason.

### 11. Internationalization

- Applicable only when the approved requirement includes user-facing text that
  must support locales, RTL, or locale-aware formatting. Otherwise `N/A` with
  that reason.
- When applicable: text is externalized, plurals and formatting are
  locale-aware, and no locale is hardcoded.
- When the requirement includes RTL: every direction-dependent unit resolves its
  layout direction from the active direction, not from a literal row, left, or right
  value. On React Native, where automatic mirroring is disabled, a literal row
  direction under an RTL locale is a `FAIL` at the file and line of the literal
  (M-66, `I18N-002`).
- A formatter takes its locale as a required parameter. A default locale literal in a
  formatter signature, or a formatting call with no locale argument, is a `FAIL` at
  the file and line (M-66, `I18N-001`).
- Copy that reaches a user through a notification, a push payload, an email, or a
  service message is externalized like component copy. A hardcoded sentence there is
  a `FAIL` when the change set consumes or modifies that copy (`I18N-004` reports the
  candidates for review).
- A pre-existing unit that violates an i18n requirement and is consumed by the change
  is corrected in the same change set (M-67). The auditor names the consumer and the
  consumed unit with file and line for each.

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

- Where the project keeps a catalog of requirement identifiers, a document that cites one and
  a catalog that lists a document are traceable in both directions (M-106). `TRACE-001` in
  `workflows/output-compliance.md` reports each one-way or missing link with its `file:line`,
  and a catalog with another layout is `NOT VERIFIED` for the lines it cannot read.
- Applicable only to multi-package or multi-project changes. Otherwise `N/A`.
- Changes stay inside the target project; package boundaries and dependency
  direction are respected.
- A proposed commit group is self-contained: its relative imports and its decision-record citations
  resolve inside the group or at HEAD, so the group does not depend on an uncommitted file of another group (`COMMIT-001` in
  `workflows/output-compliance.md`, `workflows/commit.md` Phase 11). A dependency that
  is not in the group is a `FAIL` of this dimension.
- Every file in the change set is named in its approved scope manifest (M-71).
  A file the manifest does not name is a `FAIL` of this Dimension, and so is a
  formatter or linter rewrite that the approved scope does not own and that was
  not reverted before the change set was proposed (`SCOPE-001` in
  `workflows/commit.md`).

- A read-only audit records the state of the target repository before and after
  its run (M-73): HEAD, the branch, `git status --porcelain`, and the reflog tail.
  A state change observed during the audit is an unattributed anomaly with its
  reflog entry. The report does not name its author, and the verdict is
  `NOT VERIFIED` until the user attributes the change or it is reverted with
  authorization (`STATE-001` in `workflows/output-compliance.md`).
