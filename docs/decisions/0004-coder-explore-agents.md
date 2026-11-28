# 0004. Coder and Explore agents

**Status:** proposed, pending the user's approval of the commit.
**Scope:** the agent catalog, the subagent contract, the bootstrap projection,
the routing manifests, the domain execution protocol, and the validation
inventories.
**Base:** `01d544a` on `feat/agent-teams`; the working tree before this change
set carried no other modification.

## Context

The catalog already carries `writer-pr` for copy-ready PR content. It is not a
code writer. The session requirement was a second, distinct writer for
application code and a reader agent, with a fixed model per role: an Opus
orchestrator, a Sonnet agent that writes code and dispatches the domain
subagents, and a Haiku agent named `Explore` that reads the project and loads
the skills each subagent needs. The request accepted Claude-only delivery.

The request also asked to check whether a writer already existed and to avoid
duplication. `writer-pr` exists, but its responsibility is PR content, not
application code; merging it with a code writer would violate SRP. The code
writer is therefore a new role under a new id, `coder`, and `writer-pr` is
unchanged.

## Decision

- `coder` is a primary agent (Sonnet). It writes application code inside the
  target project and dispatches the read-only domain subagents and `explore`
  for findings.
- `explore` is a read-only subagent (Haiku). It reads the project and returns
  the stack, architecture, conventions, and the skills each domain subagent
  needs. Its canonical id is `explore`; it is dispatched as `Explore` in
  Claude, where the projected name overrides the built-in.
- `orchestrator` stays primary (Opus) and remains the sole composition root:
  it alone binds the client tools and selects the concrete adapters, and it
  selects the dispatch. `coder` is the primary implementation surface, not a
  second composition root: it dispatches only read-only subagents and does not
  bind client tools.
- `external-auditor` stays the independent read-only auditor.
- `writer-pr` stays the PR-content subagent. `reviewer-pr` stays the PR-review
  subagent.
- The six domain subagents stay read-only.
- `implementation-recipes` routes to `coder`, not to `root`, because `coder`
  is the unit that writes code.

## Design patterns selected

Selection follows `references/design-patterns.md`. Each row names the force,
its signal, its translation into this Markdown stack, and two consumers.

| Pattern | Force | Signal | Consumers (`file:line`) | Stack translation |
|---|---|---|---|---|
| Strategy | The inference model is a per-role parameter (Opus, Sonnet, Haiku, or a provider-qualified id) selected at composition, not a literal in the role body | BE1 | `agents/orchestrator.md:5`, `agents/coder.md:5`, `agents/explore.md:53` | A `model` field per definition, projected by the Claude emitter; the requirement fixes the default selection per role, not a hardcoded model in the body |
| Proxy | Project reading and skill retrieval are lazy and delegated | ST3 | `agents/explore.md:5`, `:28`, `:66-71`, `:86-89`, `manifests/skill-registry@1.md:11-21` | `explore` reads on demand and returns context; the caller does not load the catalog |
| Adapter | One canonical definition is projected to several clients | ST1 | `workflows/bootstrap.md:224`, `:303`, `:343` | Three emitters translate the same canonical definition into Codex TOML, OpenCode Markdown, and Claude frontmatter |

Count: 3 selected. Rejected: Facade, `force already covered by the existing
orchestrator dispatch`; Template Method, `force already covered by
references/domain-execution-protocol.md`; Chain of Responsibility, `no
signal`; Observer, `no signal`; Singleton, `no signal`.

## SOLID

| Principle | Status | Unit or mechanism | Evidence or reason |
|---|---|---|---|
| SRP | APPLIES | `coder` writes application code; `explore` reads the project; `orchestrator` coordinates; `writer-pr` writes PR content; `external-auditor` audits | `agents/coder.md:11-27`, `agents/explore.md:66-71`, `agents/orchestrator.md:270-283` |
| OCP | APPLIES | A new role is a new definition plus a catalog row; the dispatch protocol is not edited | `agents/orchestrator.md:270-283`, `schemas/subagent-contract.md:23-41` |
| LSP | APPLIES | Every client projection is a substitutable implementation of the same canonical contract, so a caller depends on the contract and not on one client's file | `schemas/subagent-contract.md:1-8`, `references/subagent-projection.md:23-32` |
| ISP | APPLIES | `explore` lists only read tools, `skill`, and read-only skills. `coder` is a legacy client-level definition outside the v1 contract: it inherits the client's tools, and its dispatch authority is the orchestrator protocol, not a capability list. The scoped unit is `explore`; `coder` is stated, not claimed as contract-scoped. | `agents/explore.md:22-28`, `agents/coder.md:22-27`, `docs/decisions/0004-coder-explore-agents.md:160-165` |
| DIP | APPLIES | The orchestrator remains the sole composition root; `coder` is a primary surface selected by the client and does not bind concrete adapters; `explore` receives context through its definition | `agents/orchestrator.md:270-283`, `references/subagent-projection.md:23-32` |

## Final catalog: twelve definitions

| id | Client mode | Model hint | Responsibility |
|---|---|---|---|
| `orchestrator` | primary | opus | Coordinate task intake, dispatch subagents, and apply the artifacts they return. |
| `coder` | primary | sonnet | Write application code inside the target project and dispatch the domain subagents. |
| `external-auditor` | all | client default | Independent fresh-context audit. |
| `frontend` | subagent | client default | Web client structure, UI, accessibility, performance, SEO and visual craft. |
| `backend` | subagent | client default | Service contracts, implementation, dependency packages and engineering review. |
| `data` | subagent | client default | Schema, migrations, indexes, queries and transaction safety. |
| `devops` | subagent | client default | Pipelines, environments, artifact promotion and delivery gates. |
| `mobile` | subagent | client default | Native or cross-platform mobile structure, platform ports, storage and lifecycle. |
| `security` | subagent | client default | Threat modeling, security review, vulnerability catalog and secure boundaries. |
| `explore` | subagent | haiku | Read the project, return the stack, architecture, and conventions, and load on demand the skills each domain subagent needs. |
| `writer-pr` | subagent | client default | Draft copy-ready PR/MR content from cold evidence and write it only to the neutral state root. |
| `reviewer-pr` | subagent | client default | Review a PR or proposed change set from cold inputs; return copy-ready findings. |

`coder` and `explore` are new. `writer-pr` and `reviewer-pr` are unchanged. The
six domain subagents are unchanged. Three custom definitions are client-level
agents (`orchestrator`, `coder`, `external-auditor`); the other nine are
subagents.

## Deviations

This record cites `schemas/subagent-contract.md`, `workflows/bootstrap.md`,
`references/subagent-projection.md` and `workflows/system-validation.md`; the
fields and artifacts present in the repository that matter to this change are
named here.

1. `coder` is a third custom agent. This supersedes decision 3 of
   `docs/decisions/0003-domain-subagents.md`, which counted two client-level
   agents, and the earlier four-agent rule of
   `docs/decisions/0002-agent-teams.md` decision 3, which `0003` had already
   superseded. The session requirement is the direction; the commit approval
   gate is the human decision on the supersession. `explore` remains a
   subagent, so the earlier direction that every other definition is a
   subagent is preserved for it.
2. The v1 contract gains the optional top-level field `model`. Its allowed
   values are `inherit`, `opus`, `sonnet`, `haiku`, `fable`, or a
   provider-qualified id. The Claude projection emits it; a client that cannot
   bind it reports `NOT VERIFIED`. The field is a hint, not an absolute
   binding: an invocation `--model`, `CLAUDE_CODE_SUBAGENT_MODEL`, or
   `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` takes precedence, so "always Opus" is
   the configured default and the absolute runtime binding is `NOT VERIFIED`.
3. The bootstrap gate accepts `coder` with `mode: primary`. It derives the
   agent/subagent counts from the staged catalog instead of a hardcoded count.
4. Claude Code 2.1.318 supports a `model` frontmatter field, nested subagents
   up to three layers below the main conversation, and `claude --agent <id>` to
   run a definition as the main session agent. `coder` is therefore usable as
   a primary surface (`claude --agent coder`) and, where the orchestrator
   dispatches it, as a subagent that can spawn the domain subagents. `explore`
   is projected as `Explore`; a definition with that name overrides the built-in
   Explore and keeps `model: haiku`.
5. OpenCode v2.3.2 does not bind the `opus`/`sonnet`/`haiku` aliases in this
   projection. The OpenCode emitter omits the model hint for a canonical agent,
   and the OpenCode legacy pass-through strips it, so `orchestrator` and `coder`
   run on the client's default model and `explore` inherits the session model.
   `coder` is a primary agent with the client's default permissions. Model
   binding is Claude-only in this change set; the request accepted Claude-only
   delivery.
6. `orchestrator`, `external-auditor` and `coder` gain a top-level `name`.
   Claude Code treats a subagent file without `name` as documentation and skips
   it, so the legacy pass-through definitions were invisible to the client.
   The task consumes those definitions, so the defect is corrected here. The
   bootstrap `gate_claude` now asserts the projected name and its self-test
   covers the single-token rule that projects `Explore`.
7. The v1 contract gains the `skill` capability, and `explore` lists it. The
   Claude projection maps it to `Skill`, so Explore can load a skill that is not
   in its own `context.skills` allowlist when a domain subagent needs it. The
   OpenCode projection grants only the allowlist through `action: skill`, so
   loading an unlisted skill there is `NOT VERIFIED`; the request accepted
   Claude-only delivery.
8. The legacy pass-through is client-aware. The OpenCode copy strips the
   semantic model hint, because OpenCode binds a provider-qualified id. The
   Claude copy of `external-auditor` adds
   `disallowedTools: Write, Edit, NotebookEdit, Agent`, because Claude does not
   read the OpenCode permission block and a legacy definition without a tool
   restriction inherits every tool. The denylist keeps the MCP tools the
   canonical permission block allows while denying the writing and dispatch
   tools. Both behaviors are asserted by the bootstrap self-test. `Bash`
   remains available on Claude because the auditor executes read-only evidence
   commands; a shell write is a prompt-level violation, not a mechanically
   denied one, so mechanical read-only for `Bash` is `NOT VERIFIED`.
9. `writer-pr` is not merged, removed, or renamed. It remains the sole writable
   subagent and writes only to the neutral state root.
10. `implementation-recipes` moves from `root` to `coder` in
    `manifests/skill-registry@1.md`, because `coder` is the unit that writes
    code. The skill remains mandatory for every code change. The three domain
    agents that still said the orchestrator loaded it in the root
    (`agents/backend.md:95`, `agents/frontend.md:99`, `agents/mobile.md:107`)
    are corrected in the same change set, because their text contradicted the
    new routing.
11. `coder` is a legacy client-level definition outside the v1 subagent
    contract. It has no `tools`, `delegate`, `invariants`, or `completion`
    block, inherits the client's tools, and is authorized to dispatch by the
    orchestrator's dispatch protocol. The v1 `delegate` field therefore does
    not apply to it, and its ISP evidence is the stated boundary rather than a
    capability list.
12. Claude ignores `mode`, has `Plan` but no `Build`, and can run any
    definition with `claude --agent <id>`. The five-agent client-level split
    is therefore enforced by the canonical catalog and the OpenCode gate;
    Claude enforcement is `NOT VERIFIED`. The canonical id `explore` is
    dispatched as `Explore` in Claude, where the projected name overrides the
    built-in.

## Documentation consulted

Version and date are recorded per client. The consulted version is the one
installed or documented at the date below; a source that does not cover the
target version is reported, not applied.

| Client | Source | Version / scope | Date consulted |
|---|---|---|---|
| Claude Code | `https://code.claude.com/docs/en/sub-agents` | installed CLI 2.1.318; subagent frontmatter (`name`, `description`, `tools`, `disallowedTools`, `model`), nested subagents up to three layers, the built-in `Explore` override | 2026-11-27 |
| Claude Code | `https://code.claude.com/docs/en/cli-reference` | installed CLI 2.1.318; `--agent <id>` runs a definition as the main session agent | 2026-11-27 |
| OpenCode | `references/portability.md`; installed `opencode --version` | v2.3.2; no verified binding for the `opus`/`sonnet`/`haiku` aliases | 2026-11-27 |

The Claude Code documentation states that a file without `name` is treated as
documentation, that a subagent with `model` keeps that model, and that a user or
project subagent named `Explore` overrides the built-in and keeps its own
`model` field. Those three statements justify the `name` fix, the model
projection, and the `Explore` name. Claude Code has no client-level
agent/subagent flag: `mode` is not a Claude field and is ignored, `Build` is not
a Claude built-in, and `claude --agent <id>` accepts any definition. The
five-agent restriction is therefore an OpenCode and canonical-catalog property,
not a Claude one.

## M-99 pre-code design review

The full review record lives in the task checkpoint
(`workflows/session-resilience.md`); this section summarizes it.

Two independent fresh-context reviews ran as `external-auditor` subagents on
2026-11-27, both on the user-authorized `opencode/seaglass-alpha-free` model.
Earlier attempts on 2026-11-25 and 2026-11-27 failed before producing findings
(provider usage limit, then an unavailable model); they are history, not
review results.

**Review 1** returned `PASS` with no open P1 and 15 findings (M-1 to M-15:
3 P2, 12 P3), independence established, and the runtime limits recorded as
`NOT VERIFIED`. All 15 were dispositioned `Revised` and applied.

**Review 2** ran on the revised proposal in a new fresh context. It found no
open P1 and verified M-1 to M-15, with M-10 and M-12 partial, plus four new
findings: N-1 (P2, three domain agents still said the orchestrator loaded
`implementation-recipes` in the root) and N-2 to N-4 (P3, citation precision
in this record). All four were dispositioned `Revised` and applied. Review 2
returned `NOT VERIFIED`, not `PASS`, because it also demanded runtime client
execution: the Claude provider is not logged in, so actual `claude --agent`
model execution, absolute always-Opus behavior, mechanical read-only `Bash`,
Claude five-agent enforcement, the OpenCode live catalog, and the dispatch
timeout could not be executed. Those runtime items are `NOT VERIFIED` with the
environment limit named; they are not the design-review checks of
`workflows/design-review.md`, whose input, acceptance, assumption, design,
scope, boundary, and consistency checks were run in both reviews.

| Finding | Severity | Disposition | Applied in |
|---|---|---|---|
| M-1 | P2 | Revised | Deviation 11, ISP row, `contexts/dev.md`, `references/subagent-projection.md` |
| M-2 | P2 | Revised | Orchestrator, projection, `contexts/dev.md`, `workflows/task-to-code.md`, domain protocol |
| M-3 | P2 | Revised | Projection, bootstrap limits, `workflows/system-validation.md`, deviation 12; Claude enforcement `NOT VERIFIED` |
| M-4 | P2 | Revised | Projection, bootstrap, deviation 12; `Build` is an OpenCode built-in only |
| M-5 | P2 | Revised | Coder, orchestrator, projection, task-to-code, domain protocol, skill-triggers |
| M-6 | P2 | Revised | Projection, system-validation, deviation 8; `Bash` read-only `NOT VERIFIED` |
| M-7 | P3 | Revised | Deviation 2; model precedence and absolute binding `NOT VERIFIED` |
| M-8 | P2 | Revised | `0003` note, deviation 1; four-agent rule attributed to `0002` |
| M-9 | P2 | Revised | `workflows/system-validation.md` code-audit binding |
| M-10 | P3 | Revised | `workflows/system-validation.md` R-43 scope; N-1 completes the sweep |
| M-11 | P3 | Revised | `agents/explore.md` boundaries include `skill` |
| M-12 | P3 | Revised | Proxy/SRP citations; N-3 completes the precision |
| M-13 | P3 | Revised | Adapter consumers cite the three emitters |
| M-14 | P3 | Revised | Portability citation `134-136` (pre-existing) |
| M-15 | P3 | Revised | Strategy force reworded; default selection, not hardcode |
| N-1 | P2 | Revised | `agents/backend.md`, `agents/frontend.md`, `agents/mobile.md` |
| N-2 | P3 | Revised | OCP row cites the contract fields and catalog |
| N-3 | P3 | Revised | SRP/ISP rows cite coder boundaries and deviation 11 |
| N-4 | P3 | Revised | Proxy row cites description, `skill`, role, and steps |

**Gate status.** No P1 is open. Review 1 is `PASS`; Review 2 is `NOT VERIFIED`
only for the runtime items above, whose environment limit is named, and all its
design findings are resolved. The commit approval gate is the user's decision
on (a) `coder` as a third client-level agent, which supersedes the earlier
count, and (b) accepting the recorded runtime limits.
