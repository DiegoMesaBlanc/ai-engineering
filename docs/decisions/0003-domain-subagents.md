# 0003. Domain subagents over role agents

**Status:** proposed, pending the user's approval of the commit.
**Scope:** the agent catalog, the two PR subagents, the external auditor's audit
allowlist, the domain subagent definitions, the orchestrator dispatch section,
the bootstrap agent catalog, the routing manifests, and every procedure
reference that names an agent id.
**Base:** `13c3812` on `main`; all changes on `feat/agent-teams`; `ff88bfe` is
the branch head before this change set.

## Context

The branch `feat/agent-teams` added fifteen finely sliced role agents on top of
a repository that already carried every one of those capabilities as a skill.
Git evidence collected in this session:

- `git ls-tree -r --name-only main -- skills | grep '/SKILL.md$'` returns 31
  skills, including `architecture-advisor`, `task-context`,
  `complexity-advisor`, `discovery`, `project-analysis`, `stack-detection`,
  `design-analysis`, `api-contract-design`, `repository-strategy`,
  `engineering-review`, `best-practices`, `database-engineering`,
  `internationalization`, `security-review`, `vulnerability-catalog`,
  `testing-strategy`, `test-driven-development`, `accessibility`,
  `core-web-vitals`, `performance`, `seo` and `pull-request`.
- `git ls-tree -r --name-only main -- agents` returns four agents:
  `orchestrator`, `external-auditor`, `auditor`, `reviewer`.
- `git diff --name-status main...HEAD -- agents` adds fifteen role agents and
  modifies `external-auditor`, `orchestrator` and `reviewer`.
- `git diff --name-status main...HEAD -- skills` returns no change: the branch
  did not move, rename, or remove any skill.

The user corrected the direction twice: this is a **reduction** of subagents by
development domain, not the addition of a new layer. The user then supplied the
preservation rule verbatim: "Cuidado que no es borrar los agentes porque estos
antes eran skills. No se si borrarlos o volver a tenerlos como skills. Ya tú
decides que opción va mejor con la finalidad que queremos y que todo siga
funcionando como está en la rama main de este proyecto". The delegation of the
write-capability question was resolved with "Como sea mejor para lograr nuestra
finalidad".

The next direction was given verbatim: "Aprobado, pero verifica que el
sistema, por ejemplo: en mobile puede llamar la misma skill de front para la
creación de arquitectura de carpetas. Es solo un ejemplo para que te bases en el
y saques tus propias y mejoradas conclusiones. Crea lo que tengas que crear,
ajusta lo que tengas que ajustar. Al final debe haber un orquestador y
subagentes por cada flanco que llame las skills necesarias para ejecutar
ciertas determinadas tareas y que se vea una entrega bien profesional de
acuerdo a ser Senior/Staff, Teach Lead y Architec".

The following final constraint was given verbatim: "Aprobado, pero
recuerda que los agente que deben quedar es el Orchestrator para orquestar los
demás subagentes y External-Auditor para hacer auditorias".

The first delivery was reopened. The user challenged it verbatim: "Ten cuidado
porque el "writer-pr" es para crear los PR cuando se termina una tarea. Estás
seguro que se debe borrar?" and then "Verificalos todos los que borraste. Por
eso te habái dicho. Maldita sea". That challenge was correct: the earlier audit
checked that skill files survived, not that each deleted agent's capability had
a real executing home. `git show HEAD:agents/writer-pr.md` proved that the agent
wrote copy-ready PR content to the neutral state root using `tools: write`,
while the proposed replacement kept no write step at all.

The correction was directed verbatim with: "Es mejor lo que nosotros tenemos o
lo que esta en El catálogo de ECC?. Si es mejor lo nuestro, continuemos con lo
que nos falta. Además, revisa si sobra algo, si hay que modificar algo o si
tenemos que agregar algo y haz los ajustes necesarios. El "reviewer" cambiale el
nombre a reviewer-pr. Cambiale el nombre al que revise el PR para saber cual el
reviewer enfocado al PR y no a otra cosa". The follow-up asked: "También valida
si el "writer-pr" es un agente o solo un comando como el "reviewer-pr" o
"reviewer" ya que hacen practicamente lo mismo con los PRs".

**Approval-attribution limit (M-92 / APPROVAL-001).** The user messages above
are quoted verbatim, but their conversation turn indices are not retained in
this repository. Their attribution as approvals is therefore `NOT VERIFIED`
from the change set alone; they are recorded as direction received, not as an
audit-grade approval record. No gate in this repository depends on those
attributions.

ECC (`https://github.com/affaan-m/ECC/tree/main`) was read as evidence only. Its
catalog reports 72 agents and 328 skills and keeps dedicated surfaces such as
`a11y-architect`, `seo-specialist`, `performance-optimizer`,
`security-reviewer`, `code-reviewer`, `database-reviewer`, `pr-test-analyzer`,
`planner` and `architect`. The decision after comparing both models is to keep
this repository's orchestration model — one composition root, shared skills,
per-step seniority, canonical workflows and gates, no flat catalog duplicated
per language — and to import ECC's useful idea: independent review and audit
surfaces are not absorbed into the units that produce the work. ECC is not
copied and is not the source of truth for this repository.

## Design patterns selected

Selection follows `references/design-patterns.md`. Each row names the force,
its signal, its translation into this Markdown stack, and two consumers.

| Pattern | Force | Signal | Consumers (`file:line`) | Stack translation |
|---|---|---|---|---|
| Facade | Fifteen role agents fragment dispatch; one entry point is needed | ST5 | `agents/orchestrator.md:265-266`, `manifests/skill-triggers@1.md:23-26` | One orchestrator dispatch plus named subagent targets in the routing manifests |
| Strategy | Seniority changes per step inside one domain without changing the subagent | BE1 | `manifests/skill-seniority@1.md:31`, `references/domain-execution-protocol.md:20` | A manifest row resolved per step and recorded in the returned checklist |
| Template Method | Six domains share one phase skeleton with different skill allowlists | BE10 | `references/domain-execution-protocol.md:6-7`, `schemas/subagent-contract.md:33` | Fixed phases in one shared reference; variable `context.skills` per definition |
| Plugin | Skills load from configuration without editing the agent definition | CR2 | `schemas/subagent-contract.md:47`, `manifests/skill-seniority@1.md:41` | `context.skills` ids resolved against the registry and the seniority catalog |
| Adapter | One canonical definition is projected to several clients | ST1 | `schemas/subagent-contract.md:11`, `references/subagent-projection.md:17` | The bootstrap emitter maps tools; unchanged |
| Proxy | The large skill catalog is loaded lazily | ST3 | `manifests/skill-triggers@1.md:11-13`, `manifests/skill-registry@1.md:11-14` | The trigger manifest loads only the matched skills |

Count: 6 selected. Rejected: Chain of Responsibility, `no signal`; Mediator,
`force already covered by Facade`; State, `force already covered by Strategy`;
Factory Method, `no signal`; Decorator, `no signal`; Composite, `no signal`;
Singleton, `no signal`.

## SOLID

| Principle | Status | Unit or mechanism | Evidence or reason |
|---|---|---|---|
| SRP | APPLIES | `orchestrator` composes and dispatches only; `external-auditor` owns independent audits; `reviewer-pr` owns PR review only; `writer-pr` owns PR content only; six domains each own one development domain | `schemas/subagent-contract.md:32`, `references/domain-execution-protocol.md:6-8`; each definition carries one `responsibility` string |
| OCP | APPLIES | A new subagent is a new definition plus a registry row; the generic dispatch section is not edited | `manifests/skill-registry@1.md:13`, `schemas/subagent-contract.md:47` |
| LSP | PRESERVED | Every definition satisfies `ai-engineering.subagent.v1` | `schemas/subagent-contract.md:27`, `references/subagent-projection.md:114` |
| ISP | APPLIES | Per-subagent skill allowlist and reader-only tools, except the single `write` capability of `writer-pr` | `schemas/subagent-contract.md:47`, `schemas/subagent-contract.md:57` |
| DIP | APPLIES | Dispatch root is `agents/orchestrator.md:265-266`; projection construction root is the bootstrap block; subagents receive skills through context | `references/subagent-projection.md:23`, `agents/orchestrator.md:265-266` |

The dispatch root is `agents/orchestrator.md`. The projection construction root
is the single POSIX bootstrap block in `workflows/bootstrap.md`. No domain,
review or audit subagent writes to the target project. `writer-pr` is the only
writable subagent and writes only to the neutral state root.

## Final catalog: ten agents

| id | Client mode | Responsibility |
|---|---|---|
| `orchestrator` | primary | Coordinate task intake, dispatch subagents, and apply the artifacts they return. |
| `external-auditor` | all | Independent fresh-context audit: whole-cycle, pre-code design, web dimensions, security, gates, compliance and system validation; also the client Auditor mode. |
| `frontend` | subagent | Web client structure, UI, accessibility, performance, SEO and visual craft. |
| `backend` | subagent | Service contracts, implementation, dependency packages and engineering review. |
| `data` | subagent | Schema, migrations, indexes, queries and transaction safety. |
| `devops` | subagent | Pipelines, environments, artifact promotion and delivery gates. |
| `mobile` | subagent | Native or cross-platform mobile structure, platform ports, storage and lifecycle. |
| `security` | subagent | Threat modeling, security review, vulnerability catalog and secure boundaries. |
| `writer-pr` | subagent | Draft copy-ready PR/MR content from cold evidence and write it only to the neutral state root. |
| `reviewer-pr` | subagent | Review a PR or proposed change set from cold inputs; return copy-ready file-and-line findings. PR review only. |

`writer-pr` and `reviewer-pr` are subagents, not commands. Evidence: both carry
`schema: ai-engineering.subagent.v1`, `mode: underneath`, `tools`,
`invariants`, `completion.checklist` and an `activity` block; `/generate-pr` and
`/review-pr` carry only `description` and `agent: orchestrator`; the bootstrap
emits projections exclusively from `agents/*.md`. Their responsibilities are
deliberately different: `writer-pr` has `write` to the neutral state root,
while `reviewer-pr` is `readOnly: true`. Merging them would recreate the
self-review that this correction removes.

## Capability preservation of the seventeen removed agent files

The first proposal was classified with an explicit rule: PRESERVED means the
capability and its distinctive invariant have a real executing home; DEGRADADA
means the skill or workflow survived but its independent context, tool or
writing behavior was lost; PERDIDA means no home exists. The audit found 6
PRESERVADAS, 11 DEGRADADAS and 0 PERDIDAS. Every degraded capability was then
given a real home in this change set.

| Removed id | Distinctive capability | First verdict | Current home after correction |
|---|---|---|---|
| `advisor-architecture` | Architecture and pattern diagnosis | PRESERVADA | `architecture-advisor` and `repository-strategy` in the domain allowlists; Phase 8 Architecture |
| `advisor-complexity` | Proportional rigor classification | PRESERVADA | `complexity-advisor` in root; Phase 8 Complexity |
| `analyst-project` | Project state, stack and convention analysis | PRESERVADA | `discovery`, `project-analysis`, `stack-detection` in root; Phases 1 and 4 |
| `analyst-task-context` | Requirement and design normalization | PRESERVADA | `task-context` in root, `design-analysis` in frontend/mobile; Phases 1 and 2 |
| `audit-a11y` | Independent WCAG 2.2 verdict | DEGRADADA | `accessibility` routed to `external-auditor` with an explicit audit skill list and web-audit trigger |
| `audit-cwv` | Independent measured LCP/INP/CLS verdict | DEGRADADA | `core-web-vitals` routed to `external-auditor` |
| `audit-performance` | Independent before/after performance verdict | DEGRADADA | `performance` routed to `external-auditor` |
| `audit-seo` | Independent search-visibility verdict | DEGRADADA | `seo` routed to `external-auditor` |
| `auditor-design` | Fresh pre-code proposal review | PRESERVADA | `workflows/design-review.md`, Phase 9 to `external-auditor` |
| `compliance-runner` | Independent mechanical compliance verdict | DEGRADADA | Orchestrator runs the sweep; `external-auditor` owns `REPORT-001` and returns an independent verdict |
| `designer-contracts` | Versioned boundary contract | PRESERVADA | `api-contract-design` in backend/data; Phase 8 Contracts |
| `reviewer-engineering` | Independent engineering findings | DEGRADADA | `engineering-review` and `review-memory` routed to `reviewer-pr`; `external-auditor` also loads `engineering-review` for audits |
| `reviewer-security` | Independent weakness verdicts | DEGRADADA | `security-review` and `vulnerability-catalog` routed to `external-auditor`; security domain keeps its implementation copies |
| `reviewer-testing` | Independent gate execution and verdict | DEGRADADA | `testing-strategy` routed to `external-auditor` with the gate-audit trigger and procedure assignment |
| `writer-pr` | PR content, written to the neutral state root | DEGRADADA | Restored as `agents/writer-pr.md` with `tools: write`, step 4, and Phase 14 delegation |
| `auditor` (legacy) | Read-only system-validation audit | DEGRADADA | `external-auditor` is the client Auditor mode and now owns `workflows/system-validation.md` explicitly |
| `reviewer` (legacy) | PR content and PR review, read-only | DEGRADADA | Split into `writer-pr` for content and `reviewer-pr` for PR review; whole-cycle audit stays with `external-auditor` |

No capability is unhoused. Six were already preserved; eleven now have an
explicit executing home, tool and invariant. No `skills/<id>/SKILL.md` is
removed.

## Decisions

1. **Kernel rules stay intact.** Token reduction comes from fewer definitions
   and lazy skill retrieval, not from weakening `AGENTS.md`.
2. **One canonical definition, thin projections.** `agents/<id>.md` remains the
   single source of truth; the bootstrap emitter still writes Codex TOML,
   OpenCode Markdown and Claude Markdown outside the repository, stages the
   whole catalog, gates it by exact id set and count, and publishes by atomic
   symlink swap.
3. **Ten agents, not nineteen.** Two are agents (`orchestrator`,
   `external-auditor`) and eight are subagents: six domains, `writer-pr` and
   `reviewer-pr`. The legacy `reviewer` deviation from `main` is superseded by
   the renamed PR-focused `reviewer-pr`; no other client mode is changed.
4. **Budgets are report-only.** `lean@1` and `full@1` keep
   `budget.enforcement: report`. ECC's blocking mode is rejected.
5. **`implementation-recipes` stays in root** for tasks that write code.
6. **Visibility is a client status bar only**, written by the orchestrator in
   the neutral state root.
7. **`AGENTS.md` carries no status-bar rule.**
8. **AC1 is measured, not claimed.** The earlier P50 token figure remains
   `NOT VERIFIED` until three completed tasks exist.
9. **Domains, review and audit are readOnly; only `writer-pr` writes.** The v1
   contract fields `readOnly`, `noCommit`, `noHomeWrite`, the tools array and
   the absence of `delegate` are unchanged for every other subagent. The
   contract and projection now state the single neutral-state-root exception.
10. **Seniority is data, not identity.** `manifests/skill-seniority@1.md` resolves
    one seniority per skill; `references/domain-execution-protocol.md` fixes the
    six-phase skeleton and the checklist line format. The same subagent is an
    `architect` at one step and a `staff-reviewer` at another.
11. **Skills are shared capabilities.** A registry `target` is the orchestrator's
    default route, not an ownership lock. Every domain whose `architect` phase
    needs a structure decision lists `architecture-advisor`; `mobile` therefore
    loads the same folder-architecture skill as `frontend`, backed by
    `references/folder-architectures.md:330,358,377-381` for React Native.
12. **Every removed capability has a real home.** The seventeen removed agent
    files were mapped one by one to a current skill route, workflow assignment,
    tool and invariant. Six were preserved unchanged; eleven were corrected in
    this change set. Skill existence alone is never accepted as proof.
13. **PR preparation and PR review are separate subagents.** `writer-pr` drafts
    and writes; `reviewer-pr` reviews and never writes. The commands
    `/generate-pr` and `/review-pr` remain thin adapters bound to the
    orchestrator. This separation is the imported ECC insight.
14. **Audit independence is explicit.** Web, security, gate, compliance, design
    and whole-cycle audits are dispatched to `external-auditor` from a fresh
    context; a domain's `staff-reviewer` step is not an independent audit.

## M-99 pre-code design review

Two fresh design-review subagent sessions were recorded in the neutral activity
state root (outside this repository) under the session keys
`auditor-design-precode` and `auditor-design-rereview`. The client session ids
and transcripts were not retained in the change set, so the invocation detail
required by EXEC-001 is `NOT VERIFIED` from this repository alone; the outcomes
below are the recorded activity entries, not an independently re-executed
review.

- Review #1, fresh design-review session (`auditor-design-precode`, heartbeat
  2026-11-23T16:12:44Z): **BLOCKED** by P1 DR-01 — the files that carry
  mandatory invariants had no exact edit list or post-change assertion.
  Findings F-01..F-09 were also recorded.
- Dispositions: DR-01, F-01, F-02, F-03, F-04, F-05, F-07, F-08 and F-09 were
  revised; F-06 was resolved by the producer decision recorded above.
- Review #2, a different fresh design-review session (`auditor-design-rereview`,
  heartbeat 2026-11-23T16:20:37Z): **PASS**, no open P1. DR-01 was confirmed
  closed. RR-01..RR-06 and RR-08 were folded into this record; RR-07 was
  verified as a `no-op` with a grep over the five invariant workflows, which
  returned no obsolete agent id.
- Independence is contextual, not epistemic (I-7). No enforced subagent timeout
  is exposed by this client, so boundedness is `NOT VERIFIED`.
- Those reviews covered the earlier eight-agent proposal. This correction
  changes the catalog, the PR surfaces and the audit routes, so it requires its
  own fresh audit before delivery; that audit is reported in the validation
  evidence, not assumed here.

## Deviations

This record cites `schemas/subagent-contract.md`, `workflows/bootstrap.md` and
`workflows/task-to-code.md`; the fields and artifacts present in the repository
that matter to this change are named here.

1. `reviewer` existed on `main` with client `mode: primary`; the branch changed
   it to `subagent`. That file is removed. Its PR-content capability is
   restored in `writer-pr` and its PR-review capability in `reviewer-pr`; the
   whole-cycle audit remains in `external-auditor`.
2. `auditor` existed on `main` as a pass-through client mode. It is removed;
   `external-auditor` keeps `mode: all`, is selectable as the Auditor mode, and
   now explicitly owns `workflows/system-validation.md`.
3. `compliance-runner` existed only on the branch. It is removed; the
   orchestrator runs `workflows/output-compliance.md`, and `external-auditor`
   returns the independent `REPORT-001` verdict.
4. `writer-pr` existed on the branch and was removed in the first proposal. It
   is restored with `tools: read, write`, `readOnly: false` and a step that
   writes copy-ready content to the neutral state root. The bootstrap emitter
   projects it like any other v1 subagent.
5. The v1 contract fields `schema`, `id`, `name`, `description`, `mode`,
   `responsibility`, `context.profile`, `context.skills`, `context.references`,
   `tools`, `body`, `invariants.readOnly`, `invariants.scope`,
   `invariants.noCommit`, `invariants.noHomeWrite`, `invariants.timeout.source`,
   `invariants.timeout.seconds`, `completion.returns`,
   `completion.terminalStates`, `completion.checklist`, `activity.states`,
   `activity.statusFile` and `activity.lifecycleAuthority` are unchanged in
   meaning; the prose of `noHomeWrite` now names the `writer-pr` state-root
   exception. `body` and `timeout.seconds` are named here because the cited
   schema declares them: `body` is the non-empty Markdown body after the
   frontmatter and `timeout.seconds` is required only when
   `timeout.source` is `client-enforced`.
6. The bootstrap gate emits `mode: primary` only for `orchestrator`,
   `mode: all` only for `external-auditor`, and `mode: subagent` for the eight
   subagents. The `.v2` ready record and `AI_ENGINEERING_STATE_HOME` resolution
   are unchanged.
7. No precedence claim is made for this record: its introducing commit is not
   asserted to precede the governed code by ancestry alone. The record is
   written as the first increment of the change set.
