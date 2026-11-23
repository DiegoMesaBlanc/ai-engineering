# 0002. Agent teams for token efficiency

**Status:** proposed, pending the user's approval of the commit.
**Scope:** the subagent definitions, projections, manifests, context profiles, and orchestration wiring introduced on branch `feat/agent-teams`.
**Base:** `13c3812` on `main`; all changes on `feat/agent-teams`; `main` unchanged.

## Context

The kernel of mandatory rules in `AGENTS.md` cannot be trimmed without losing
correctness, traceability, or validation. Token cost instead comes from loading
large skill catalogs and repeating reads in the root session. The ECC repository
(`https://github.com/affaan-m/ECC/tree/main`) was read as evidence only. Its
useful ideas — a harness-neutral IR, thin per-client projections, lean/full
profiles, skill triggers, a delegation completion contract, iterative retrieval,
per-mode contexts, and a host-compatibility gate — are adapted here, not copied.
In particular, ECC's `budget.mode: "blocking"` is rejected: budgets in this
system are report-only and never a hard task stop.

## Design patterns selected

Selection follows `references/design-patterns.md`, derived from the task's
design forces. Each row names the force it resolves, its translation into this
repository's Markdown stack, and at least two consumers with `file:line`.

| Pattern | Force it resolves | Stack translation | Consumers (`file:line`) |
|---|---|---|---|
| Proxy (GOF-12 / ST3) | Lazy retrieval of a large catalog so the root does not load it | On-demand skill loading through a trigger manifest | `manifests/skill-triggers@1.md:19`, `contexts/dev.md:16`, `contexts/review.md:16` |
| Facade (GOF-10 / ST5) | One entry point for command dispatch across clients | `agent: orchestrator` frontmatter in command adapters | `commands/adopt-baseline.md:3`, `commands/task.md:3`, `commands/validate.md:3` |
| Command (GOF-14 / BE2) | A request object per command adapter, decoupled from execution | One Markdown adapter per command bound by `agent:` | `commands/code-audit.md:3`, `commands/generate-pr.md:3`, `commands/system-check.md:3` |
| Adapter (GOF-06 / ST1) | One canonical definition projected to several clients | Bootstrap emitter maps canonical tools to each client tool map | `workflows/bootstrap.md:293`, `references/subagent-projection.md:112` |

The Proxy row's lazy-retrieval trigger is defined at
`manifests/skill-triggers@1.md:19`; the Adapter row's runtime target under
`${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}/projections/codex/agents/`
is generated at bootstrap and is outside the repository, so it has no
`file:line` in the change set. No pattern was kept without a present force; no
quota was applied.

## SOLID

| Principle | Status | Unit that implements it |
|---|---|---|
| SRP | APPLIES | Each subagent owns one responsibility (`agents/analyst-task-context.md:7`, `agents/advisor-complexity.md:7`, `agents/reviewer-engineering.md:7`); the composition root wires only. |
| OCP | APPLIES | A new subagent is added by registering it in `manifests/skill-registry@1.md`; the orchestrator is not edited. |
| LSP | APPLIES | Every projection satisfies `schemas/subagent-contract.md`, so callers depend on the contract, not the client. |
| ISP | APPLIES | `lean@1` and `full@1` expose only the skills each mode needs; readers receive no write tool. |
| DIP | APPLIES | `agents/orchestrator.md:12` is the composition root; every other module receives context and tools through `schemas/subagent-contract.md:10`. No concrete client import outside the root. |

The composition root is `agents/orchestrator.md`.

## Decisions

1. **Kernel rules stay intact.** Token reduction comes from routing knowledge
   and executing large catalogs in subagents, not from removing mandatory
   wording from `AGENTS.md`.
2. **One canonical definition, thin projections.** `agents/*.md` is the only
   source of truth. At bootstrap one parse feeds three native emitters: Codex
   TOML, OpenCode Markdown with deny-first permissions, and Claude Markdown
   frontmatter, all under
   `${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}/projections/<client>/agents/`,
   outside the repository. Legacy files without the v1 schema are copied
   verbatim to both Markdown clients so no agent disappears from discovery.
   The four discovery links are repointed by a guarded, idempotent relink that
   fails closed on any destination it does not recognize. The emitters are
   embedded in the single POSIX bootstrap block of `workflows/bootstrap.md`:
   no separate executable and no `package.json` are added to this
   documentation-only repository. The ready record is invalidated at the start
   of each run and rewritten only when all checks pass, so a failed regeneration
   leaves the repository `missing` and the first-run gate retries (fail-closed).
   Each catalog is staged in full, gated by exact id set and count, and published
   by an atomic symlink swap (`workflows/bootstrap.md:509`, `:646`), so a client
   never caches a partial catalog.
3. **Exactly four agents; everything else is a subagent.** The only agents are
   the built-in `Plan` and `Build`, plus `orchestrator` (`mode: primary`) and
   `external-auditor` (`mode: all`). The OpenCode emitter therefore writes
   `mode: subagent` for every canonical agent (`workflows/bootstrap.md:293`).
   The canonical delegation mode (`underneath` / `parallel`) stays in the note,
   the canonical file still wins on conflict, and read-only is enforced by the
   deny-first permission block. The legacy `auditor` and `reviewer` were returned
   to `mode: subagent` (`agents/auditor.md:3`, `agents/reviewer.md:3`). A
   publish-time allowlist rejects any other client mode in the staged catalog
   (`workflows/bootstrap.md:652`).

   This revision was approved by M-99 review #4 (PASS, no open P1). The exact
   outside paths were approved with the verbatim user message "Sí, autorizo
   ambas rutas". The P2 finding R4-01 was closed with the verbatim user message
   "Invalidar en fallo (Recomendado)".
4. **Budgets are report-only.** Each context profile declares
   `budget.enforcement: report` (`manifests/context-profiles/lean@1.md:8`,
   `manifests/context-profiles/full@1.md:8`); exceeding a budget reports and
   refines retrieval, never stops a task. The subagent contract states the same
   policy in prose (`schemas/subagent-contract.md:17`) and has no `budget`
   field. ECC's blocking mode is rejected.
5. **`implementation-recipes` stays eager only for tasks that write code**
   (`agents/orchestrator.md:244-248`); it is not loaded in read-only tasks.
6. **Visibility is a client status bar only.** The activity state lives in
   `${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}/activity/<repo-slug>.json`,
   written by the orchestrator. There is no activity line in message prose and
   no lifecycle label in the task panel. Accepted limitation: AC7 "for every
   client" is not verifiable; clients without a configurable bar are
   `NOT VERIFIED`. Writing client configuration in the home directory is a
   human gate with verbatim approval of exact paths.

   The Phase D gate was resolved on 2026-11-21 with the verbatim user message:
   "Desde que no dañe nada de lo que hay ahora en el settings.json, no pasa nada".
   The agent links were created from `git rev-parse --show-toplevel`, the
   state-root path uses `${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}`
   with `<repo-slug>` resolved at runtime, and the `statusLine` key was inserted
   into the user settings file with the pre-existing keys unchanged, verified by
   a before/after comparison.
7. **`AGENTS.md` carries no status-bar rule.** The status-bar protocol is
   best-effort, not an always-active rule, so it needs no canonical wording, no
   row in `references/mandatory-rules.md`, and no check in
   `workflows/system-validation.md`. The only edit there is the first-run
   one-liner, whose ready key is versioned with the projection block (`.v2`) so
   the gate re-runs when the bootstrap changes.
8. **AC1 is measured, not claimed.** Baseline in Phase A and post-measurement
   in Phase F use the same tree aggregation. No 20–30% figure is asserted
   before measurement. At baseline time only one root task existed, so the
   P50 over at least three completed tasks is `NOT VERIFIED`.
9. **Data files use the repository's `.md` convention.** The contract and the
   four manifests are Markdown with YAML frontmatter, matching `skills/` and
   `agents/`. JSON was the initial serialization, chosen for machine
   parseability and ECC alignment; it was replaced at the user's direction
   because this repository has no runtime JSON consumer and every brace and
   quote is a token paid on each read. Conformance is now verified by
   structural review over the change set (`workflows/output-compliance.md`),
   not by a JSON Schema validator. The runtime status file remains JSON
   because it is written outside the repository and consumed by client status
   bars, not read as repository documentation. This is a format-only change:
   retrieval semantics, profiles, triggers, and subagent contracts are
   unchanged, so the pre-code design review of the substance still stands.

## Documentation consulted

Version and date are recorded per client. The consulted version is the one
installed or documented at the date below; a source that does not cover the
target version is reported, not applied.

| Client | Source | Version / scope | Date consulted |
|---|---|---|---|
| OpenCode | `references/portability.md`, `workflows/session-resilience.md`, local OpenCode DB schema `session_v2` | repository revision `13c3812`; DB schema as installed | 2026-11-21 |
| Claude Code | `references/portability.md` client-configuration map; official `https://code.claude.com/docs/en/settings-reference.md` (`statusLine` entry) and `https://docs.claude.com/en/docs/claude-code/settings` (settings precedence) | repository revision `13c3812`; installed CLI 2.1.318; docs current at consultation | 2026-11-21 |
| Codex | `references/portability.md` client-configuration map | repository revision `13c3812` | 2026-11-21 |
| ECC (reference only) | `https://github.com/affaan-m/ECC/tree/main`, local read cache | branch `main`, read-only evidence | 2026-11-21 |

The official Claude Code reference confirms `statusLine` as an object with
`type: "command"`, a `command` string, optional `padding`, optional
`refreshInterval` (seconds, minimum 1), and optional `hideVimModeIndicator`. It
is valid in any settings file. The Phase D insertion uses `type`, `command`, and
`refreshInterval` only; it preserves the other keys of the user settings file
unchanged. No `jq` or `python3` is required at render time.

A source that does not cover a client's target version is reported rather than
applied. Client-specific status-bar support is `NOT VERIFIED` where the client
does not expose a configurable bar. OpenCode v2.3.2 and Codex CLI 0.145.0 had no
configurable bar that reads an arbitrary file confirmed from their installed
configuration at the consultation date; without a concrete mechanism this stays
`NOT VERIFIED` rather than being asserted either way.

### Visibility follow-up (2026-11-22, option 1 authorized)

The user reported that no subagent activity was visible and authorized the
repair in the user turn with the verbatim message "Opción 1 y autorizo". Official docs
were re-consulted on 2026-11-22 at `opencode.ai/docs/cli/config`,
`opencode.ai/docs/cli/tui`, `opencode.ai/docs/plugins`, `opencode.ai/docs/keybinds`,
the published `opencode.ai/v2/cli.json` schema,
`docs.claude.com/en/docs/claude-code/statusline`,
`developers.openai.com/codex/config-reference` and `developers.openai.com/codex/hooks`.
OpenCode V2 `cli.json` has no `statusLine` and only `tabs.indicators` plus TUI
plugins; Claude Code has a configurable `statusLine` running
a shell command with JSON on stdin; Codex `config.toml` has no bar but supports lifecycle hooks including
`SubagentStart` and `SubagentStop` with JSON on stdin
(`developers.openai.com/codex/config-reference`, `developers.openai.com/codex/hooks`).

A real defect was found and fixed: the Phase D `statusLine` command used
`grep -o '"status":"[^"]*"'`, but the activity file is written with
`json.dump(..., indent=2)`, which emits `"status": "running"` with a space, so
the command printed nothing. It was replaced with a spacing-tolerant pattern
that also prints the count of `EN_CURSO` sessions. Verified against the real
activity file: the old command printed empty, the new one printed
`running | activos=<n>`. Only `statusLine.command` changed in
`~/.claude/settings.json`; `enabledPlugins`, `extraKnownMarketplaces`, `theme`,
and `refreshInterval` are byte-preserved, confirmed by before/after comparison.

For Codex, `~/.codex/config.toml` gained `[[hooks.SubagentStart]]` and
`[[hooks.SubagentStop]]` command hooks at 5 s timeout that call a bridge script
under the neutral state root, `~/.ai-engineering/hooks/codex-activity-bridge.py`.
The bridge reads the hook JSON from stdin, updates the same activity file with
file locking and an atomic rename, sets the session status `EN_CURSO`/`TERMINADO`,
recomputes the aggregate, prints a `systemMessage`, and attempts a best-effort
`osascript` notification. It was verified against a temporary state: start →
`running`/`EN_CURSO`, stop → `done`/`TERMINADO`, unrelated events → no-op; the
real activity file was untouched by these probes. All pre-existing Codex keys
(`approval_policy`, `sandbox_mode`, `model_provider`, `model`, `model_providers`,
`auto_review`, `tui`, `mcp_servers`) are preserved, confirmed by a TOML parse
and a before/after diff.

Honest status after the fix: **Claude Code bar VERIFIED** (command renders real
output). **Codex: lifecycle mirrored and notification emitted, but Codex has no
configurable bar**; the visual channel is a notification and a `systemMessage`,
not a persistent bar. Codex non-managed hooks also require one-time review and
trust via `/hooks` before they run; until the user does that, the bridge is
installed but inert, so its in-session firing is `NOT VERIFIED`.

OpenCode V2 v2.3.2 needs a sharper distinction than "no bar". Its official keybind
reference (`opencode.ai/docs/keybinds`, consulted 2026-11-22) documents native
subagent visibility: `session.child.first` ("Toggle subagent picker", default
`down`), `session.child.next` (`right`), `session.child.previous` (`left`),
`session.parent` (`up`), and `composer.subagent.select/interrupt`. The installed
binary's TUI footer also renders `<n> subagent(s)` from tabs whose status is
`running`, plus a keyed "View subagents" hint, and the command palette registers
"View subagents" when child sessions exist. Therefore OpenCode **does** have a
native visible surface for running subagents; it is not driven by this system's
`activity.json`, and there is **no configurable `statusLine`** that could read
that file. No plugin was installed under option 1 and `~/.config/opencode/` was
not written. The in-session appearance of the `<n> subagent(s)` footer in the
user's terminal is `NOT VERIFIED` from this environment; the keybind contract and
the rendering code were verified from official docs and the installed binary.

AC7 "for every client" therefore remains `NOT VERIFIED` in the universal sense:
Claude Code has a verified configurable bar, Codex has verified hook mirrors but
no bar, and OpenCode has native subagent indicators but no external-file bar. The
claim is narrowed to the mechanism verified per client.

### Agent catalogue repair (2026-11-22)

The user then reported that agents and subagents did not appear or work and
authorized the repair in the user turn with the verbatim message "Haz lo
necesario para arreglar los agentes y que funcione todo con agentes y
subagentes". One defect was confirmed: the TUI had started
while the four legacy agent projections were absent, so it cached an incomplete
catalogue and kept returning `AgentNotFoundError: orchestrator`. The projections
were restored verbatim (19 ids: 15 canonical plus 4 legacy) and the catalogue was
reloaded with `opencode api post /api/location/reload`. A first repair attempt
made every canonical agent `mode: all`; the user corrected the requirement in
the user turn with the verbatim message "solo deben ser agente el Plan, Build,
Orchestraitor y External-Auditor, todos los demás son putos sub agentes". That attempt was
reverted: the emitter writes `mode: subagent` again, `auditor` and `reviewer` are
`subagent`, and a publish-time allowlist keeps `orchestrator` (`primary`) and
`external-auditor` (`all`) as the only custom agents. Built-in `Plan` and `Build`
supply the other two. The live catalogue reported 26 total agents: 7 built-in
and 19 custom, of which exactly 2 custom are agents and 17 custom are subagents.
Subagents are visible through
OpenCode's native subagent picker and `<n> subagent(s)` footer, not through the
interactive agent list.

The only other legacy edits were the stale keybind text in
`agents/external-auditor.md`, corrected to `Shift+Tab` / `Ctrl+X then A` against
`opencode.ai/docs/keybinds`, and the mode corrections above; all four legacy
files are copied verbatim into the Markdown projections. Codex stays
byte-identical to the baseline: `shasum -c` over the 15 TOML files reports 15
`OK`. A real invocation through `/api/agent` returned
`Agent not found: "no-such-agent-x"` for a nonexistent id and started
`external-auditor` and `orchestrator` with the session model, which confirms
dispatch by id. The model balance error seen in earlier `--standalone` probes
belongs to the pay-as-you-go model, not to the agents; inside the TUI the agents
inherit the session model.

The direct visual confirmation inside the user's terminal is `NOT VERIFIED`
from this environment; it depends on the user opening the agent list. AC7's
universal claim remains `NOT VERIFIED` as stated above.

## Deviations

- ECC `schemas/agent.schema.json` → `schemas/subagent-contract.md`: added
  `mode`, `responsibility`, `context`, `invariants`, `completion`, and
  `activity`; the contract has no `budget` field and states report-only budgets
  in prose at `schemas/subagent-contract.md:17`; expressed the contract as
  Markdown with YAML frontmatter instead of a JSON Schema (decision 8).
- ECC `budget.mode: "blocking"` → `budget.enforcement: "report"` in the context
  profiles (`manifests/context-profiles/lean@1.md:8`,
  `manifests/context-profiles/full@1.md:8`), not in the subagent contract
  (decision 3).
- ECC emitters are executable scripts; this repository keeps generation inside
  the POSIX bootstrap block of `workflows/bootstrap.md`, adds no separate
  executable emitter and no `package.json` (decision 2), and writes the
  generated Codex projections outside the repository.
- `references/token-efficiency.md` optional SQL targets table `session`; the
  installed OpenCode version exposes `session_v2`/`session_part`. Baseline
  queries were adapted read-only; the reference file is unchanged in this
  change set.
