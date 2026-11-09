---
name: database-engineering
description: Guide safe schema, migration, query, and transaction work so database changes protect data integrity, performance, and production safety.
---

# Database Engineering

## Purpose

Guide changes that touch the database: data modeling, migrations, queries,
indexes, and transactions. Database work carries a different risk profile
from ordinary application code — a mistake can lose data or take down
production — so it is handled deliberately.

This Skill is loaded only when a task actually touches the database; it is
not part of the default implementation path.

## When to Apply

Apply when a task:

- adds or changes a table, collection, column, index, constraint, or
  relationship
- writes, edits, or reorders a migration, or changes the schema
- changes a query, its performance characteristics, or adds/removes an index
- introduces or changes transaction boundaries, locking, or concurrency
  behavior
- backfills, migrates, or transforms existing data
- changes how the application accesses or persists data at the boundary

Do not load this Skill for ordinary feature work that only uses existing
data access unchanged. Do not design a schema or migration for a purely
local refactor.

## Detect Before Changing

Inspect the real project first:

- database engine and version, and whether environments differ
- existing ORM/ODM, query builder, or driver and its conventions
- existing migration tooling, naming, and ordering
- existing schema conventions: keys, constraints, timestamps, soft deletes,
  naming, and types
- data classification: which fields are sensitive, personal, or regulated
- indexing strategy and known hot queries
- environment topology: local, staging, production, and how migrations are
  applied

Reuse the established conventions. Do not introduce a second ORM, migration
tool, or schema style without a justified reason and human approval.

## Data Modeling and Integrity

- Model entities around real access patterns and invariants, not around a
  single screen.
- Express integrity in the schema: primary keys, foreign keys, unique
  constraints, `not null`, and check constraints where the engine supports
  them.
- Choose types that match the data and its valid range; avoid untyped or
  overloaded columns.
- Follow the project's naming, timestamp, and soft-delete conventions.
- Prefer additive, low-risk changes; avoid renaming or retyping existing
  columns without a compatibility plan.

## Safe Migrations

- Apply schema changes through the project's migration tooling only; never
  mutate a shared or production database by hand.
- Prefer backward-compatible, expand/contract migrations over
  change-in-place so deploy and migration order never break running code.
- Keep migrations small, ordered, and reversible where practical.
- Separate schema changes from data backfills; batch large backfills instead
  of one long transaction.
- Account for lock and table-size impact on busy tables.
- Test migrations against realistic data, including rollback, before
  applying them to shared environments.
- Never edit an already-applied migration; add a new one.
- A migrated database must end with the same schema as a fresh install:
  columns, types, defaults, `CHECK` constraints, indexes, and foreign keys. An
  `ALTER TABLE ... ADD COLUMN` that omits a constraint the fresh schema has
  leaves two schemas that behave differently. Compare both schemas (for
  example with `PRAGMA table_info` and `sqlite_master` in SQLite, or the
  equivalent introspection) and report any difference as a finding.
- Verify a migration on a copy of a database created by the previous schema,
  and run it twice to confirm it is idempotent. Report the result as evidence;
  a migration that was not run against an existing database is reported as not
  verified.
- A database change is compatible in either deployment order (M-69). Prove both
  orders with tests that execute the code: the new release against a database
  that has not yet received the migration, and the previous release against the
  migrated database. A test named for the legacy or previous schema is the
  evidence; a claim of order-safety with no such test is a false claim and is
  reported as `NOT VERIFIED`.
- Application code never changes the schema (M-70). A `CREATE`, `ALTER`, or
  `DROP` statement, including `CREATE ... IF NOT EXISTS`, is written only in a
  migration and run only by the migration tooling. Startup, boot, and request
  paths do not issue DDL, so a release never mutates a database it did not
  migrate.

## Queries and Performance

- Keep queries parameterized; never build queries by string concatenation.
- Index for the actual access patterns; avoid redundant or unused indexes.
- Watch for N+1 access introduced through ORM lazy loading; load in batches.
- Avoid `SELECT *` and fetching more than is needed.
- Paginate large result sets; never load unbounded collections.
- Verify the plan for slow or critical queries (`EXPLAIN`) rather than
  guessing. A performance change is proven by a before-and-after measurement of
  the named query on a stated dataset, and a claim of a covering or index-only
  access is proven by its plan (M-65). Without that measurement the claim is not
  evidence.
- Keep raw SQL isolated and justified when the ORM is insufficient.

## Transactions and Concurrency

- Define transaction boundaries at the level that protects the invariant —
  not per query, not excessively wide.
- Choose the isolation level deliberately and understand its anomalies.
- Keep transactions short; never hold a transaction open across a network or
  user interaction.
- Design operations to be idempotent where retries are possible.
- Handle deadlocks, lock contention, and serialization failures with bounded
  retries, not silent failures.
- A paginated or cursor-based read that issues several statements runs in one
  read transaction or snapshot, so a row changed between pages is not counted
  twice or skipped (M-84; `DB-003` in `workflows/output-compliance.md` reports
  the page loops with no such transaction). A claim that the result is
  consistent is kept only with a test that changes a row between pages and
  asserts the count.

## Security and Privacy

- Use least-privilege database credentials for the application; never use an
  administrative account at runtime.
- Never store secrets or credentials in code, migrations, or seed data.
- Handle personal or sensitive fields per policy: minimize, encrypt where
  required, and never log them.
- Keep parameterized access and input handling at trust boundaries.
- Do not expose more data than the operation requires.

## Destructive and Production Operation Gates

Require explicit human approval before any action that can lose or
irreversibly change data, including:

- `DROP`, `TRUNCATE`, destructive `ALTER`, or removing a column or table
- irreversible migrations or data transformations on shared or production
  environments
- running migrations or backfills against shared or production data

Prefer reversible steps and confirm a recovery path (backup or rollback)
before destructive operations. Never run destructive database commands on
shared or production resources without authorization.

Before proposing a destructive or data operation, enumerate the stores, tables,
and files it touches from the target in the same session. The inventory is
produced by an enumeration of the target, cited with its command and its count,
and its definition is stated (M-72). A store name that the enumeration does not
return is not listed, and an enumerated store is listed or excluded with a reason.

## Relation to Other Components

- `architecture-advisor` decides where data access belongs and how it is
  layered.
- `implementation-recipes` implements the application-side data access.
- `api-contract-design` defines what is exposed; this Skill protects how it
  is stored and retrieved.
- `testing-strategy` selects the depth of migration, data, and query tests.
- `engineering-review` verifies the result.
- `dependency-assessment` governs any new database driver, ORM, or migration
  tool.

## Decision Gates

Introduce a new database engine, ORM, or migration tool; change an isolation
level or a core data model; or perform any destructive or production data
operation only with human approval.

## Output

### DATABASE PLAN

- Engine, version, and affected environments
- Detected conventions and tooling reused
- Schema and model changes, with integrity constraints
- Migration steps, ordering, and rollback plan
- Query and index changes with performance rationale
- Transaction and concurrency considerations
- Security and data-classification impact
- Validation approach (tests, `EXPLAIN`, migration dry run)
- Deploy-order evidence: the test that runs the new release against the
  unmigrated database and the previous release against the migrated one (M-69)
- Destructive steps and required approvals
- Data-store inventory produced by an enumeration of the target, with the
  command, the count, and its definition (M-72)
- Risks and deviations

## Safety

- Never run destructive or irreversible database operations on shared or
  production data without explicit human approval.
- Never store secrets or sensitive data in code, migrations, or logs.
- Never bypass the project's migration tooling or edit applied migrations.
- Never issue DDL from application code: a schema statement at startup or in a
  request path bypasses the migration tooling and breaks the deployment order
  (M-70, M-69).
- Never weaken integrity constraints to make a change easier.
