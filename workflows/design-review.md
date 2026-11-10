# Pre-Code Design Review Workflow

## Purpose

Find the unstated assumptions, contradictions, and design violations in a
proposal **before the first file is written**, while changing the design is
still cheap.

The idea is adopted from the "Elephant and Goldfish" model of the
google/elephant-goldfish-model project (Apache-2.0): the agent that drafts the design
is not the agent that reviews it, and the reviewer receives the design and its
evidence without the earlier conversation. Only the idea is adopted. No plugin,
skill, or prompt file of that project is installed or copied.

The producer drafts. A fresh-context reviewer challenges. The human decides.

## When It Runs

It runs in Phase 9 of `workflows/task-to-code.md`, after the task plan and the
proposal are drafted and **before the proposal is presented for approval**, so
the user sees the review findings together with the proposal.

- Required for software implementation tasks at rigor R2 (Medium), R3 (High),
  and R4 (Critical), as classified in `workflows/task-planning.md`.
- For Trivial and Low tasks it is skipped, and the skip is recorded with the
  reason (proportional process rigor; the code quality baseline still applies).
- Never runs for architecture-only, estimation-only, or read-only requests:
  they write no file and have nothing to implement.

## Independence

The review follows the Independence Contract of `workflows/code-audit.md`
(Independent Mode), rules I-1, I-2, I-3, and I-7. It is not restated here.

- **I-1 Separate session.** A fresh subagent or a new session. Running it in
  the session that wrote the proposal does not create independence.
- **I-2 Cold input.** The reviewer receives the inputs listed below, nothing
  else.
- **I-3 Read-only.** No edits, no commits, no branches, no remote writes.
- **I-7 Declared limit.** The same model keeps the same blind spots. The report
  states this.

**Allowed input:**

- the requirement wording and its acceptance criteria, in the original words;
- the proposal: the presentation record of the design patterns, the Folder
  Architecture Map, the File Responsibility Map, the contracts and types, the
  dependency direction, the approved scope manifest, and the pending decisions;
- the repository paths the proposal cites, read-only;
- the canonical rules (`AGENTS.md`, `references/audit-pass-criteria.md`).

**Denied input:** the implementation conversation; the reasoning the producer
gives for a choice; the producer's justification text beyond the proposal
itself; any validation claim not yet executed. If the reviewer's context already
holds the conversation, independence is `NOT ESTABLISHED`: stop, and request a
fresh session. Reading the proposal in the change set is not a breach.

## Procedure

1. **Input Manifest.** List what was received and what was withheld. Record the
   client, the session, and whether the session is fresh.
2. **Acceptance check.** Map every acceptance criterion to a planned unit and to
   the test or observation that proves it. A criterion with no planned unit or
   no proof is a finding. A criterion the requirement does not settle is
   `AMBIGUOUS`, and the reviewer does not interpret it.
3. **Assumption check.** List every assumption the proposal relies on that the
   requirement does not state. Each one cites the proposal line it comes from.
4. **Design check.** Trace the presentation record to the proposal: each
   selected pattern has at least two planned consumers with `file:line` or
   planned path; each SOLID row names the unit that implements it or the reason
   it does not apply; the composition root is named; a concrete import is
   planned only in the composition root (`DIP-001`).
5. **Scope check.** Every planned file and every changed value of existing
   behavior appears in the approved scope manifest. An unlisted one is a finding
   (`SCOPE-001`, `SCOPE-002`).
6. **Boundary check.** Every external call in the plan names its timeout and,
   when it retries, its retry budget; every boundary contract names its error
   codes and the test that asserts each one.
7. **Consistency check.** Contradictions between sections of the proposal are
   findings, citing both lines.

Each check either records a finding or states `no finding` with the evidence
it read. A check that was not run is `NOT VERIFIED`, never a pass.

## Findings and Dispositions

Each finding has an identifier, a severity, the proposal line, the evidence,
and a proposed resolution.

| Severity | Meaning | Gate |
|---|---|---|
| P1 | The proposal cannot satisfy a requirement, breaks a mandatory rule, or relies on an unstated grant, scope, or behavior change | Blocks implementation until resolved |
| P2 | A design violation or an unverifiable criterion that the proposal can fix | Presented with the proposal; the user decides |
| P3 | A clarity or naming observation | Presented; no gate |

Every finding receives exactly one disposition before implementation starts:

- **Revised**: the proposal changes, and the change is shown.
- **Accepted by the user**: quoted with the verbatim user message and its turn
  (M-92).
- **Rejected**: with evidence that the finding does not hold.

A finding is never dropped silently. A disposition that changes the approved
scope, an architecture, a permission, or a dependency is a human decision gate
in `AGENTS.md`, and it is raised with the user, not decided by the producer.

After a P1 is revised, the review runs again on the revised proposal in a
**new** fresh context, not the reviewer that raised it. Only a re-review that
finds no open P1 lets the proposal reach the approval question.

## Output

The review is shown to the user with the proposal, in this form:

- the Input Manifest (received, withheld, independence declaration);
- the verdict: `PASS` (no open P1, every check run), `BLOCKED` (an open P1),
  or `NOT VERIFIED` (a check was not run, or independence was not established);
- the findings table with identifier, severity, line, evidence, and disposition;
- the limit: independence is contextual, not epistemic (I-7).

The review record lives in the task checkpoint (`workflows/session-resilience.md`),
never in the repository, like the scope manifest.

## Related

- `workflows/task-to-code.md` — Phase 9 runs this review before the approval
  question.
- `workflows/code-audit.md` — the Independent Mode this workflow reuses for
  independence; it audits the code after it is written.
- `workflows/task-planning.md` — the rigor level that decides whether the
  review is required.
- `references/audit-pass-criteria.md` — the criteria the design check applies.
