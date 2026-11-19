---
name: replica
description: Clean-room replication of a website or mobile app. Maps public references into a screen inventory, user flows and a feature matrix, verifies parity, and implements through the canonical task-to-code workflow. Use when the user asks to clone, replicate, rebuild or match an existing web or mobile app, or supplies its URL or screenshots as the target.
license: Policy and artifact shapes adapted from replica-skill, MIT (Jake Schincariol); see Sources and attribution.
---

# Replica

## Purpose

Build a new implementation that does the same job as an existing web or mobile app without taking anything the original owns. The deliverable is working software plus a recon map and a parity report that say honestly what was rebuilt and what was left out.

This Skill supplies constraints, artifact shapes and verification. It does **not** replace `workflows/task-to-code.md`: task alignment, branch policy, phase gates, implementation, validation, review and delivery follow the canonical workflow. When a task is only an analysis or an estimate, no file is written in any project.

## Sources and attribution

Adapted from [replica-skill](https://github.com/Jakeschincariol/replica-skill) by Jake Schincariol ([opusjake.ai](https://opusjake.ai)), MIT License, Copyright (c) 2026 Jake Schincariol.

- Pinned commit: `77c9436fb3d18c3d58169efb8caf4fe906b0dc51`
- Upstream file: `README.md` and the `SKILL.md` files of `replica-recon`, `replica-design`, `replica-build` and `replica-diff`.
- Local changes: selected the clean-room rules, the recon and feature-matrix shapes, the per-screen definition of done and the parity procedure; rewrote them to fit this system's workflows and gates; did not copy any upstream file, Python tool, template or test.
- The upstream tools are **optional** and are not installed or copied here. If the user has installed the upstream pack, its standard-library Python scripts may be used; installing anything is a separate, authorized dependency decision.

## Scope and evidence boundaries

- The target project is the only writable scope. The referenced app, its pages, screenshots and store listings are evidence only and stay read-only.
- Write `replica/` artifacts only when the approved scope manifest names those paths. Otherwise keep the notes in the task checkpoint or the user-level handoff directory, never in the repository.
- Public sources and the user's own account only. Never log into another person's account, ask for a password, get past a paywall, or scrape behind a login. Read at human speed with the user present; no crawlers, bulk downloads or automation against the target.
- Do not read or save the target's bundles, source maps, binary, private APIs or network calls in order to copy them. Public API documentation is allowed.
- Check the target's terms when the user's own account is used. If those terms forbid building a competing product, say so and work from public sources only.

## Hard rules: clean room

- Rebuild **functionality and UX patterns**, not owned material. Write every line of code, label, empty state and email fresh.
- Never take the target's source code, logos, icons, illustrations, photos, sounds, fonts, copy, content or data. Use an open icon set and the project's own assets.
- Matching what a control does is parity. Matching its exact sentence is copying.
- Record licensed content, the target's network and users, data it owns, partner integrations, hardware and regulated licences as out of scope with a reason.
- Rebrand before shipping: a new name, palette, mark and words. A check for remaining traces of the original is required before any delivery.
- Do not chase pixel parity. The exact look is the original's trade dress and changes before launch.
- Do not invent reviews, quotes, metrics or user research. Mark estimates as estimates and inaccessible evidence as `NOT AVAILABLE`.

## Human decision gates

Ask before proceeding when the decision materially changes the outcome:

- The platform and the slice to replicate. "All of X" is not a project; propose the core loop the users pay for and confirm it.
- Whether the goal is internal study, a product to sell, or a client deliverable. The legal and rebranding obligations differ.
- Terms of service that restrict the intended use, and any trademark or domain question before launch.
- New dependencies, paid services, credentials, permissions, or any deployment. Delivery follows `workflows/secure-delivery.md` when CI/CD is involved.

## Artifacts and procedure

### 1. Scope and size

Record the app, platform, slice and audience. Inventory screens, flows, entities and the hard parts (realtime, sync, payments, calendar or mail integrations, offline). Give an honest size: S, M, L or XL — rescope XL. No promise of a perfect clone.

### 2. Sources table

Build a table with one URL per row before reading anything, in this order of value: help centre/docs, pricing page, changelog, store listing, public walkthroughs, marketing site, the user's own account, public API docs. Mark unavailable sources explicitly. Use `skills/knowledge-graph/SKILL.md` when the collected material is large enough that reading it whole would be wasteful, and `skills/context-compression/SKILL.md` when a single output is too large.

### 3. Recon map

Write `replica/recon.md` with stable identifiers that every other artifact uses:

- `S01`, `S02`… — screen, route or how to reach it, purpose, key components, and states seen. Record empty, loading, filled, error, permission-denied and mobile states; an unrecorded state is a state that will not be built.
- `F01`, `F02`… — a user goal and the screens it passes through, with the edge cases on the happy path. Count the clicks to finish; that is the number to beat.
- Components — name, variants, states, and which screens use them.
- Inferred data model — entities, fields, relationships, evidence and confidence. Guesses are marked as guesses.
- Out of scope — each exclusion with its reason.
- Size and the three hardest parts.

Use `skills/design-analysis/SKILL.md` to interpret screenshots, URLs or design artifacts. It stays the authority for explicit versus inferred requirements and for token extraction.

### 4. Feature matrix

Write `replica/features.csv` with columns `feature, area, priority, original, clone, notes`. Priority is `must`, `should` or `could`; `clone` starts at `no` and becomes `yes`, `partial` with a note, or `skip` with a reason. The matrix is the input to parity verification and to build ordering.

### 5. Design system and states

Measure roles, not raw values: colour roles, type scale, spacing base, radius, shadow, motion, content width, grid and breakpoints. Map them to the project's tokens first; a value with no token is reported as a gap, not hardcoded in a component. Specify every component with default, hover, active, focus-visible, disabled, loading, error and empty states, including keyboard and screen-reader behaviour.

Visual direction comes from `skills/frontend-design/SKILL.md`; polish and symmetry from its `references/visual-rhythm.md`. Accessibility, RTL and locale come from their owning skills. `skills/ui-ux-design/SKILL.md` supplies the pre-delivery checklist.

### 6. Implementation

Follow `workflows/task-to-code.md` with the approved phase plan and its gates. Build the shell and routing, then the core loop end to end as a vertical slice, then the remaining screens. For each screen: read its recon row, use the reference only for layout and hierarchy, build with existing primitives, cover every state, and update the feature matrix. Use fake data with the same contract as the real data layer when the backend is not yet in scope.

### 7. Parity verification

Parity means the same job, not the same look. Report four things:

1. **Feature parity** — from the current matrix. Weight `must` 3, `should` 2, `could` 1; count `partial` as half; exclude `skip` rows and features the original does not have. A missing must-have means not shippable.
2. **Layout parity** — compare screenshots at the same viewport and in the same state, using the project's supported desktop and mobile sizes. Record the method and the tool used. If no comparison tool was executed, report the layout score as `NOT VERIFIED`, not as a guessed number.
3. **Behaviour diff** — walk each flow in both apps: clicks to finish, error handling, what is remembered, and what notifications arrive. Record each difference as flow, original, clone, and fix or keep.
4. **Verdict** — `not shippable`, `shippable`, or `better than the original`, with the evidence that supports it. Give honest numbers.

### 8. Delivery

Report the screens built, must-haves done, parity evidence, out-of-scope items and the next step. Check the codebase for any remaining name, asset or copy of the original before presenting the result as shippable. Delivery and commit follow their own approval gates in this system.

## Per-screen definition of done

- [ ] Every state from the recon map, plus empty, loading and error.
- [ ] Works at the project's supported desktop and mobile widths.
- [ ] The flow can be completed with the keyboard alone; focus is visible.
- [ ] Semantic markup, labels on inputs, alt text, and no console errors.
- [ ] No copied copy, asset or identifier from the original.
- [ ] Feature matrix updated and a same-viewport screenshot saved for comparison.

## Not adopted from upstream

- The `replica-architect`, `replica-backend`, `replica-entrepreneur`, `replica-brand`, `replica-launch` and `replica-deploy` skills: architecture, data, legal, business and delivery decisions already have canonical owners in this system.
- The six Python tools and their tests: use the project's existing tooling or the optional upstream install instead of copying them here.
- The rigid eleven-step sequence and one-commit-per-screen rule: the canonical workflow owns phase boundaries and commit policy.
