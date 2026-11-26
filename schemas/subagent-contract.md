---
id: subagent-contract
schemaVersion: 1
schema: ai-engineering.subagent.v1
title: AI Engineering subagent contract
---

# Subagent contract

Canonical, harness-neutral contract for every subagent in the AI Engineering
System. One definition is authored in `agents/<id>.md` and projected to each
client (OpenCode Markdown, Claude Markdown, Codex TOML, manual guidance). Tools
are named canonically; each projection maps them through its own tool map.

This is an adaptation of the ECC agent IR idea, not a copy: it adds delegation
mode, a completion contract, security invariants, and an activity-state
contract. Budgets are report-only here, never a hard gate.

This repository is documentation-only: there is no executable validator.
Conformance is verified by structural review over the change set, following
`workflows/output-compliance.md`. The fields below are the authoritative list.

## Required fields

| Field | Type | Constraint | Meaning |
|---|---|---|---|
| `schema` | string | const `ai-engineering.subagent.v1` | Version discriminator; a different value is rejected by the loader. |
| `id` | string | `^[a-z0-9-]+$` | Stable canonical id; the source filename stem in kebab-case. |
| `name` | string | non-empty | Human-readable name shown in logs and the status file. |
| `description` | string | non-empty | When to delegate; consumed by the skill-trigger proxy. |
| `mode` | enum | `underneath` \| `parallel` | `underneath`: the caller waits. `parallel`: the caller may continue and join later. This is the delegation mode, not the client's agent-type flag. |
| `responsibility` | string | non-empty | The single reason this unit changes (SRP). |
| `context` | object | see below | Retrieval inputs: profile plus on-demand skills and references. |
| `tools` | array | enum, see below | Canonical capabilities the subagent may use. |
| `invariants` | object | see below | Security and boundary rules that survive every projection. |
| `completion` | object | see below | What the result contains and how its end is observed. |
| `activity` | object | see below | Visibility contract for the client status bar. |
| `body` | string | non-empty | The Markdown body after the frontmatter: system prompt and steps, in English. |

No additional fields are allowed at the top level.

## `context`

| Field | Type | Constraint |
|---|---|---|
| `profile` | enum | `lean@1` or `full@1`; id of a profile in `manifests/context-profiles/`. |
| `skills` | array of string | ids this subagent may load, in retrieval order; `^[a-z0-9-]+$`. |
| `references` | array of string | optional; reference paths relative to the repository root. |

Skills are loaded on demand, never eagerly beyond the profile.

## `tools`

Allowed values: `read`, `grep`, `glob`, `shell`, `webfetch`, `websearch`,
`edit`, `write`, `todowrite`, `delegate`.

A reader-type subagent (analyst, advisor, reviewer, auditor, compliance runner)
lists only `read`, `grep`, `glob`, and when needed `webfetch` or `websearch`. It
never lists `edit`, `write`, or `delegate`. The PR writer lists `read` and
`write` and writes only to the neutral state root.

## `invariants`

| Field | Type | Constraint |
|---|---|---|
| `readOnly` | boolean | `true` = no repository writes. |
| `scope` | enum | `target-project` or `evidence-only`. |
| `noCommit` | boolean | const `true`; a subagent never creates a commit. |
| `noHomeWrite` | boolean | const `true`; a subagent never writes to home or client-config directories. The single exception is `writer-pr`, which writes generated PR content to the neutral state root and nowhere else. |
| `timeout` | object | `{ source: client-enforced \| unavailable, seconds?: integer >= 1 }`. `seconds` is required when `source` is `client-enforced`. A declared value with no enforcing timer is `NOT VERIFIED`. |

## `completion`

| Field | Type | Constraint |
|---|---|---|
| `returns` | array | one or more of `findings`, `evidence`, `verdict`, `checklist`, in the order the result must present them. |
| `terminalStates` | array | one or more of `TERMINADO`, `DETENIDO`, unique. |
| `checklist` | boolean | const `true`; the result carries the subagent's own R-step checklist. |

A delegated result without its checklist is incomplete.

## `activity`

| Field | Type | Constraint |
|---|---|---|
| `states` | array | one or more of `EN_CURSO`, `TERMINADO`, `DETENIDO`, unique. |
| `statusFile` | boolean | const `true`; the orchestrator writes the runtime status file in the neutral state root, outside the repository. |
| `lifecycleAuthority` | string | const `client-status-bar`; no activity line in message prose and no lifecycle label in the task panel. |
