# Subagent projection

## Purpose

Define how one canonical subagent definition (`agents/<id>.md`, conforming to
`schemas/subagent-contract.md`) is projected to each client, and which
invariants survive every projection. Load this reference when authoring,
reviewing, or validating a projection.

This is provider-neutral. It does not assume a client exposes a given tool or
surface; where a capability is absent, the projection reports it and the check
is `NOT VERIFIED` with the capability named
(`references/portability.md:116-119`).

## Model

One canonical definition, many thin projections. The canonical Markdown file
is the single source of truth. Projections contain no independent behavior:
they name the same `id`, `description`, retrieval profile, and tool
capabilities, and translate them to the client's native format. A change to
behavior is made once, in the canonical file.

The composition root is `agents/orchestrator.md`. It is the only module that
selects and dispatches subagents. A subagent never dispatches another subagent
unless its contract explicitly lists the `delegate` capability.

## Canonical tool map

The contract names capabilities, never client tool names. Each projection maps
a capability to its client tool and records the mapping. A capability the
client cannot express is omitted from the projection and reported; it is never
silently replaced with a different tool.

| Capability | Meaning | Projected where supported |
|---|---|---|
| `read` | Read a file at a path | Native read tool of the client |
| `grep` | Search file contents | Native search tool |
| `glob` | Match file paths | Native glob tool |
| `shell` | Run a bounded shell command | Native shell tool, with an explicit timeout |
| `webfetch` | Read a public URL | Native fetch tool |
| `websearch` | Search the public web | Native web-search tool |
| `edit` | Modify an existing file | Native edit tool (writable agents only) |
| `write` | Create a file | Native write tool (writable agents only) |
| `todowrite` | Maintain the native task panel | Native task tool, no lifecycle labels |
| `delegate` | Dispatch a further subagent | Native subagent tool, composition root only |

A reader-type subagent (an analyst, advisor, reviewer, auditor, or compliance
runner) lists only `read`, `grep`, `glob`, and, when needed, `webfetch` and
`websearch`. It never lists `edit`, `write`, or `delegate`. The PR writer lists
`read` and `write` and writes only to the neutral state root, never to the
target repository.

## Security invariants (every projection)

These come from the contract's `invariants` object and are non-negotiable in
every client:

- `readOnly`: a reader subagent performs no repository writes. Its projection
  grants no write tool.
- `scope`: `evidence-only` means the subagent reads evidence paths but never
  modifies them; the target project is the only writable scope and only for a
  writable agent acting inside it.
- `noCommit`: no subagent creates a commit. Commit approval is a human gate in
  the caller session.
- `noHomeWrite`: no subagent writes to the home or client-configuration
  directories. Only the orchestrator writes the runtime status file, and only
  after the human gate in Phase D authorizes the client configuration paths.
- `timeout`: a declared bound counts only when the client tool enforces it. If
  the tool exposes no enforced timeout, the projection records `unavailable`
  and boundedness is `NOT VERIFIED`; the caller applies Phase 7 of
  `workflows/session-resilience.md` (report symptoms, timestamps and affected
  step; no unverified root cause; checkpoint and recover), and never claims a
  hard stop.

## Retrieval profiles

A projection never loads the full skill catalog. It selects a profile:

- `lean@1`: the minimum discovery kernel for routing; most skills stay routed.
- `full@1`: the complete set for deep audit or design tasks.

Profiles are data in `manifests/context-profiles/`, not executable emitters.
They are read by the orchestrator to decide what context a dispatched subagent
receives. Overriding a budget reports and refines retrieval; it never becomes
a hard token gate (`workflows/session-resilience.md`, "Token and Context
Efficiency").

## Activity-state map

The status file is `${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}/activity/<repo-slug>.json`,
under the neutral state root and outside the repository. The orchestrator
writes it at dispatch and at closure. The client status bar is the lifecycle
authority; there is no activity line in message prose and no lifecycle label in
the task panel.

| Contract state | Status value | Terminal? | Step marker |
|---|---|---|---|
| `EN_CURSO` | `running` | no | `in_progress` |
| `TERMINADO` | `done` | yes | `completed` |
| `DETENIDO` | `stopped` | yes | `[ ! ]` plus the reason |

The file is keyed by session or run in addition to the repository slug, written
atomically, closed on anomalous termination through Phase 7, and expires stale
state so the bar never shows `done` while a subagent is active. Where a client
has no configurable status bar, the indicator is `NOT VERIFIED` for that
client; the design does not depend on it.

The step markers in the task panel remain the canonical `[ x ]`, `[   ]`,
`[ ! ]` (`AGENTS.md`, "Phase Checklist"); the panel mirrors progress only and
carries no lifecycle label.

## Per-client projections

The canonical `agents/<id>.md` is the source of truth and validates against
`schemas/subagent-contract.md`. Its `mode` field means delegation
mode (`underneath` / `parallel`), not the client's agent-type flag; the
projection records that value in a note and the canonical file still wins on
conflict. OpenCode projections emit `mode: subagent` for every canonical agent:
only `Plan`, `Build`, `orchestrator` and `external-auditor` are agents, and every
other definition is a subagent visible through the native subagent picker, not
the interactive agent list. Read-only is enforced by the deny-first permission
block, not by the client mode. At bootstrap, OpenCode and Claude
receive generated native Markdown projections and Codex receives generated TOML
projections under
`${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}/projections/<client>/agents/`;
that path is a stable symlink to a versioned catalog. The client discovery links
point at the stable path. Legacy canonical files without the v1 schema are
copied verbatim to both Markdown clients so the full catalogue stays
discoverable. No projection is stored in the repository.

| Client | Native projection | Status-bar support |
|---|---|---|
| OpenCode | `<id>.md` generated at bootstrap: native frontmatter, deny-first permissions, per-id skill grants; linked at the client discovery path | Native task navigation; no configurable count bar in v2.3.2 (`NOT VERIFIED`) |
| Claude Code | `<id>.md` generated at bootstrap: native `name`/`description`/`tools`/`skills`; linked at the client discovery path | `statusLine` in the user configuration file, resolved at setup time; writing it is a human gate |
| Codex | `<id>.toml` generated at bootstrap from the canonical files, outside the repository, and linked at the client discovery path | `SubagentStart`/`SubagentStop` hooks with a bounded bridge; real trigger in the user session `NOT VERIFIED` |
| Other chat clients | Manual selection from the canonical definition using the retrieval profile | No native bar; `NOT VERIFIED`, documented only |

All three generators live in the single POSIX bootstrap block of
`workflows/bootstrap.md`: one awk parse feeds the Codex, OpenCode and Claude
writers, each output has a structural gate, and an executing self-test covers
rejection, capability mapping, catalog completeness and idempotency. Legacy
canonical files without the v1 schema are copied verbatim to both Markdown
clients. A run stages the complete catalog in a private temp directory, checks
the exact id set and count, then swaps the stable symlink with `rename(2)`. A
client therefore sees either the previous complete catalog or the next complete
one, never a partial list — the failure that originally cached a broken agent
catalog. Superseded versioned directories are pruned only after a successful
swap. The generated catalogs are runtime targets, not checked-in projections.

Writing a client configuration file in the home directory is a human decision
gate: the exact paths require verbatim approval before any write. Client paths
in this reference resolve from environment variables; no machine-specific
absolute path is named.

## Delegation completion contract

A delegated result is incomplete unless it contains the fields named by the
contract's `completion.returns` and its own R-step checklist. A result that
carries no checklist is asked for it before the phase is reported. The caller
mirrors the subagent's checklist into its own panel before delegation and
rewrites it from each returned checklist
(`workflows/session-resilience.md`, "Delegated work").

## Validation

Validate each canonical definition against the contract in
`schemas/subagent-contract.md`: required fields present, field types and enums
respected, reader agents granted no write or delegate capability. The repository
is documentation-only, so validation is structural review over the change set
(`workflows/output-compliance.md`), not an executable JSON Schema check. Then
check, per projection: same `id` and description, same profile, no capability
the client cannot express, invariants preserved, and no write or commit granted
to a reader. A failing projection blocks the change set that introduces it. For
all three clients, the bootstrap structural gate, the staged catalog gate and
the executing self-test are the projection checks (`workflows/bootstrap.md`,
"The block"); the generated files are outside the change set and are never
checked in. The live catalog is checked with `opencode api get /api/agent`: all
canonical and legacy ids must be present; the only non-`subagent` custom ids are
`orchestrator` and `external-auditor`, and `Plan` and `Build` are the built-in
agents. Every other custom id must report `mode: subagent`.
