# Token Efficiency

## Purpose

Define how to measure and calibrate context and token use without weakening
correctness. Load this reference on demand when a task needs a baseline,
budget calibration, or a review of context efficiency.

The canonical policy is in `AGENTS.md` and
`workflows/session-resilience.md`. This reference supplies metric
definitions and optional measurement queries; it does not create a hard
token cap.

## Principle

The policy — the minimum sufficient context, the exploration budgets, the
retrieval rules, and the stop rules — is defined in
`workflows/session-resilience.md` ("Token and Context Efficiency"). It is not
restated here. This reference holds only the measurement: what is measured,
how it is calibrated, and how it is read.

**Never trade correctness, safety, or validation for lower token use.** The
metrics calibrate the work; they never become the objective.

## Metrics

Measure per task, where a task is the root session plus all delegated
subagent sessions in its tree. Aggregating subagents into the parent avoids
double counting.

Record:

- input, output, reasoning, cache-read, and cache-write tokens;
- cost when the client exposes it;
- message count;
- tool calls by type;
- unique files read and repeated reads of the same path;
- Skills and references loaded, when traceable;
- compactions;
- task outcome and validation status.

## Baseline and calibration

1. Select representative completed tasks from the target project.
2. Aggregate each root session tree.
3. Classify by rigor when the information is available.
4. Compute P50 and P75 for input tokens, tool calls, and unique files read.
5. Keep the initial soft budgets until this baseline exists; do not invent
   token targets.
6. Calibrate the budgets from the measured distribution and the task type.

For a before/after comparison:

- use at least three representative tasks;
- target a 20–30% reduction in input tokens and 30% fewer repeated reads;
- require no reduction in correctness, validation, or task completion;
- if the target is not reached, improve retrieval and stop rules rather than
  reducing required evidence.

The canonical soft exploration budgets and stop rules are in
`workflows/session-resilience.md`. Update them there once, not in this
reference.

## Optional OpenCode queries

These queries are client-specific and optional. Run them read-only. Replace
`:directory` with the target project root. Do not publish session content,
credentials, prompts, or tool output.

Recent sessions for a project:

```sql
SELECT id, title, parent_id, tokens_input, tokens_output,
       tokens_reasoning, tokens_cache_read, tokens_cache_write, cost,
       time_created
FROM session
WHERE directory = :directory
ORDER BY time_created DESC
LIMIT 20;
```

Aggregate a root session tree:

```sql
WITH RECURSIVE task(id, root_id) AS (
  SELECT id, id
  FROM session
  WHERE directory = :directory AND parent_id IS NULL
  UNION ALL
  SELECT s.id, t.root_id
  FROM session s
  JOIN task t ON s.parent_id = t.id
)
SELECT t.root_id,
       SUM(s.tokens_input) AS input_tokens,
       SUM(s.tokens_output) AS output_tokens,
       SUM(s.tokens_reasoning) AS reasoning_tokens,
       SUM(s.tokens_cache_read) AS cache_read,
       SUM(s.tokens_cache_write) AS cache_write,
       SUM(s.cost) AS cost
FROM task t
JOIN session s ON s.id = t.id
GROUP BY t.root_id
ORDER BY input_tokens DESC;
```

Tool calls for a root session tree:

```sql
WITH RECURSIVE task(id, root_id) AS (
  SELECT id, id
  FROM session
  WHERE directory = :directory AND parent_id IS NULL
  UNION ALL
  SELECT s.id, t.root_id
  FROM session s
  JOIN task t ON s.parent_id = t.id
)
SELECT json_extract(p.data, '$.tool') AS tool, COUNT(*) AS calls
FROM part p
JOIN task t ON p.session_id = t.id
WHERE json_extract(p.data, '$.type') = 'tool'
GROUP BY tool
ORDER BY calls DESC;
```

Files read and repeated reads:

```sql
WITH RECURSIVE task(id, root_id) AS (
  SELECT id, id
  FROM session
  WHERE directory = :directory AND parent_id IS NULL
  UNION ALL
  SELECT s.id, t.root_id
  FROM session s
  JOIN task t ON s.parent_id = t.id
)
SELECT json_extract(p.data, '$.state.input.filePath') AS path,
       COUNT(*) AS reads
FROM part p
JOIN task t ON p.session_id = t.id
WHERE json_extract(p.data, '$.type') = 'tool'
  AND json_extract(p.data, '$.tool') = 'read'
GROUP BY path
HAVING reads > 1
ORDER BY reads DESC;
```

## Reporting

Record the task, rigor, budget, actual usage, any budget expansion and its
reason, repeated reads, and validation outcome. Keep the report compact and
store it with the existing checkpoint or task state rather than creating a
new document by default.

Do not use this reference to rank people or teams. It exists to improve
context selection and task execution.
