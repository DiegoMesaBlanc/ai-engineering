# Task Planning Workflow

## Purpose

Define, proportionally to the task, the expected behavior and the
implementation plan before implementation begins, so that simple changes
stay lightweight and complex changes do not start without a defined
behavior.

This is the single canonical planning procedure. Complexity is classified
and a rigor level is recommended in `skills/complexity-advisor/SKILL.md`;
the planning result is retained in `skills/task-context/SKILL.md`. The
procedure itself is defined only here.

## Position in the Lifecycle

This workflow is invoked from Phase 9 (Plan) of `workflows/task-to-code.md`.
It is not a competing lifecycle:

- analysis (Phases 1–8) precedes it;
- implementation (Phase 10) follows it, only once the plan is sufficient and
  any required approval has been obtained.

Planning does not implement and does not modify application code.

## Inputs

Reuse results already obtained; do not repeat analysis:

- task context and working interpretation (`skills/task-context/SKILL.md`)
- discovery, project state, and stack (`skills/discovery/SKILL.md`,
  `skills/stack-detection/SKILL.md`)
- definitive complexity classification (`skills/complexity-advisor/SKILL.md`)
- architecture diagnosis and the architecture decision gate
  (`skills/architecture-advisor/SKILL.md`, Phase 8)
- dependency outcome (`workflows/dependency-assessment.md`)

A preliminary complexity estimate may exist earlier, but the definitive
classification comes from Phase 8 and is not fixed before sufficient
evidence is available.

## Step 1 — Determine the Rigor Level

Derive the rigor level from the definitive complexity classification using
the mapping in `skills/complexity-advisor/SKILL.md`. Do not reclassify
complexity here.

If the available information is insufficient to select a level, resolve the
missing evidence first. Do not force a level, and do not record a definitive
rigor before the classification is supported by evidence.

## Step 2 — Produce the Proportional Plan

Apply only the depth required by the selected level:

| Level | Complexity | Required planning content |
|---|---|---|
| R1 Brief | Trivial, Low | objective; in-scope and out-of-scope; one to three verifiable acceptance criteria; validation approach |
| R2 Moderate | Medium | acceptance criteria; proposed solution design (components, contracts, placement); planned tests; risks and assumptions |
| R3 Complex | High | explicit expected behavior (states, flows, edge cases, error behavior); technical decisions with alternatives and trade-offs when a decision is not inferable; test strategy; risks |
| R4 High-risk | Critical | everything in R3 plus a validation and rollback strategy and explicit attention to security, data, and concurrency impact |

Common content for every level:

- files and modules affected
- existing abstractions to reuse
- the dependency outcome and its evidence
- the selected Secure Delivery Workflow when CI/CD, security, or delivery
  controls are involved
- validation and risks

Do not add content that the level does not require. Do not produce a heavy
specification for a simple task.

## Step 3 — Verifiable Acceptance Criteria

State each acceptance criterion so it can be verified: observable behavior,
or a specific artifact or command. Derive criteria from the task and from
repository evidence. Never invent acceptance criteria.

## Step 4 — Assumptions, Risks, and Unknowns

Separate facts from assumptions. Make material assumptions, risks, and
unresolved unknowns visible in the plan. An unknown that could change the
solution materially becomes a question or a decision gate, not a silent
guess.

## Step 5 — Persistence Decision

By default the plan is session state, retained in the working
interpretation and persisted through `workflows/session-resilience.md` only
when continuity requires it.

Create a durable planning artifact only when at least one of these holds:

- cross-session traceability is required;
- an approval record must be retained;
- the user requests a durable artifact;
- a task or provider requires one.

Never create a Markdown specification for a trivial task. Never duplicate
the code's behavior as a second specification. Record the decision and its
reason.

## Step 6 — Human Decision Gates

Apply the Human Decision Gates in AGENTS.md. Approval is triggered by risk,
irreversible or high-impact operations, security or data impact, cost,
scope or backward-compatibility changes, shared or production resources,
and project rules — not by the classification label.

A High (R3) task therefore does not require approval merely because it is
classified High; it requires approval only when a decision gate applies. A
Critical (R4) task activates reinforced controls and requires approval
before high-impact operations.

When a gate applies, present the recommendation, alternatives, and
trade-offs, and wait for the user's decision before implementing. Do not
turn a recommendation into an approved decision.

## Step 7 — Handoff

Present the plan and the selected rigor level. Continue to implementation
(Phase 10) only when the plan is sufficient for the level and any required
approval has been granted. If implementation must stop for a user answer,
stop before changing any code.

## Output

Return a concise plan:

### TASK PLAN

- Rigor level:
- Complexity (from Phase 8):
- Objective:
- In scope:
- Out of scope:
- Acceptance criteria (verifiable):
- Solution design (R2+):
- Expected behavior (R3+):
- Validation and test strategy:
- Dependency outcome:
- Risks and assumptions:
- Decision gates and approval status:
- Artifact decision:
- Next step:

Keep the plan proportional to the task. A brief R1 plan is complete for a
trivial change.

## Boundaries

- Planning does not implement, commit, push, or open a pull request.
- Planning does not define a competing lifecycle order.
- Planning reuses the complexity, architecture, and dependency results
  instead of repeating them.
- Planning adds no new dependency and no new Skill; it applies the
  existing ones.
- The plan is proportional; missing depth is added by moving to a higher
  level, only when evidence justifies it.

## Safety

- Never invent requirements, acceptance criteria, behavior, or evidence.
- Never require an elaborate specification for a trivial task.
- Never present an assumption as a fact.
- Never start implementing a complex or high-risk change without a defined
  expected behavior and, when a gate applies, an approval.
- Never create redundant planning artifacts or a second specification of
  the code.
- Never treat a recommendation as an approved decision.
