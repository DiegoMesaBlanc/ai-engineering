# Self-Learning Workflow

## Goal

Let the AI Engineering System improve its own effectiveness over time by
turning observations from real work into durable knowledge: capture small
lessons, consolidate them into a trustworthy set, and promote the stable,
general ones into the canonical knowledge layer through an explicit human
approval.

This workflow is provider-neutral. It is not tied to OpenCode, Claude,
Codex, Aider, or any single client; clients only resolve the store path and
follow the phases below.

This workflow is the canonical definition of capture, consolidation, and
promotion behavior. It is never a retrospective of people or teams; it is
about the system's own engineering knowledge.

---

## Neutral Store

Learnings are written to a user-level directory **outside every target
project's source tree** and outside the canonical ai-engineering repository,
using a provider-neutral root:

```
${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}/learnings/<scope>/<entry>.md
```

Rules:

- `AI_ENGINEERING_STATE_HOME` overrides the root when configured and valid;
  otherwise the default `$HOME/.ai-engineering` is used.
- `<scope>` is either `global` for system-wide lessons or `<repo-slug>` for
  project-specific ones, derived from the repository root directory name.
- The store holds only learning metadata and prose. Never store
  credentials, tokens, secrets, cookies, private keys, personal data, or
  private content values.
- Never write learnings inside a target project's source tree or into the
  canonical repository's tracked files.
- Normalize every identifier used as a path component: accept only ASCII
  letters, digits, `-`, `_`, and `.`, with a bounded length; reject empty
  values, path separators, `.`/`..`, and other unsafe characters. After
  building a path, resolve it and verify it still starts with the neutral
  root; reject writes that escape it or follow a symlink outside it.

---

## Phase 1 — Capture

Record observations while they are fresh, as small independent entries.
Capture sources include:

- recurring corrections or preferences from the user
- project-specific conventions, quirks, or constraints
- implementation or design patterns that worked, or failed, and why
- recurring friction in the system's own workflows, Skills, or prompts
- validation surprises, tooling gotchas, and provider behavior

Each entry records at least:

- scope (`global` or `<repo-slug>`)
- a short, specific statement of the lesson
- the context in which it was observed
- the supporting evidence actually observed
- confidence and whether it is confirmed or a hypothesis
- the date and, when available, the task and commit it came from

Do not record speculation as fact. Do not record secrets or personal data.
Do not create a learning for a one-off event with no reusable signal.

---

## Phase 2 — Consolidate

Merge, deduplicate, and refine captured entries so the store stays small and
trustworthy:

- merge entries that state the same lesson; prefer the clearest wording
- resolve contradictions, preferring the better-evidenced entry and marking
  the weaker one superseded
- discard stale or disproven entries
- keep the scope correct; promote project-specific lessons to `global` only
  when evidence shows they are not project-specific
- keep the store compact; it is a curated set, not a log

---

## Phase 3 — Promote (Proposal Only)

A lesson is promotable when it is stable, general enough to matter beyond
one context, and supported by evidence.

Promotion means proposing a change to the canonical knowledge layer — a
Skill, a workflow, `AGENTS.md`, or a reference. Promotion is **always a
proposal that requires human approval**:

- never edit the canonical repository automatically
- present the proposed change, the lesson it encodes, and the evidence
- follow the normal branch, dependency, architecture, and validation
  policies for any canonical change
- record the outcome; when a proposal is rejected, keep or revise the
  learning entry rather than silently discarding the observation

Never promote a lesson as a change that bypasses the system's own
guardrails, and never weaken a safety, quality, or cost rule in the name of
learning.

---

## Phase 4 — Maturity Model

The system's maturity describes demonstrated reliability and autonomy. It
is evidence-based and descriptive; it never grants authority and is never
self-declared.

- **L1 Guided** — needs explicit, detailed instructions; frequent
  correction; low autonomy.
- **L2 Assisted** — handles routine tasks with close review; occasional
  correction.
- **L3 Reliable** — autonomous on routine work; reviews significant
  changes; rare correction.
- **L4 Trusted** — autonomous within policy; surfaces risks and decisions;
  consistent, evidence-based validation.
- **L5 Adaptive** — improves the system itself by capturing, consolidating,
  and promoting knowledge safely, without weakening guardrails.

Maturity for a scope rises or falls on evidence such as: validation and
review pass rates, the frequency and severity of corrections, adherence to
policies and decision gates, and the honest reporting of unverified or
blocked work. Report the current level with its evidence; do not claim a
level the evidence does not support.

---

## Safety

- Provider-neutral; never depend on a single client's path or feature.
- Read-only with respect to target project source trees; the only write is
  the learning entry in the neutral store.
- Never store secrets, credentials, or personal data.
- Never overwrite or delete unrelated entries or another scope's store.
- Never edit the canonical repository automatically; promotion is always a
  proposal with human approval.
- Never weaken safety, quality, or cost guardrails in the name of learning.
- Never present a hypothesis, or an unverified result, as established
  knowledge.
