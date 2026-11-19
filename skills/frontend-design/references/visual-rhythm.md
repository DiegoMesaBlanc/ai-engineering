---
name: visual-rhythm
description: Measurable symmetry and rhythm redlines for polishing an existing screen — spacing on one grid, equal things at equal sizes, aligned columns, nested radii and quiet chrome — plus a bounded audit-to-check sequence of passes. Load when a UI must become more symmetric, when polishing a screen, or when a redesign needs an ordered set of passes.
license: Adapted from unslop by Damir Sokolovsky (@heydamir), CC BY 4.0. Attribution and local changes are recorded below.
---

# Visual Rhythm and Symmetry

## Source and attribution

Adapted from **unslop** by Damir Sokolovsky ([@heydamir](https://x.com/heydamir)), licensed CC BY 4.0.

- Upstream: https://github.com/heydamir/unslop
- Pinned commit: `620163eedb3529d9cd23a3994ed8d287d06f760d`
- License: https://creativecommons.org/licenses/by/4.0/
- Local changes: selected the symmetry and rhythm rules, rewrote them as stack-neutral checks, added the ordered pass sequence, and removed the upstream scripts, story video, image-generation guidance, numeric token tables and RTL rules (RTL is owned by `skills/internationalization/SKILL.md`). No upstream file is copied here.

## Relation to this system

This reference extends `skills/frontend-design/SKILL.md`. Load it on demand for a redline pass over an existing screen; it does not need to be loaded for every UI task.

- Project tokens and a deliberate brand win over the starting values below. These values are a measuring stick for a pass, not a house style.
- `skills/accessibility/SKILL.md` owns contrast, focus, keyboard and reduced motion. `skills/internationalization/SKILL.md` owns direction, locale and formatting. `skills/ui-ux-design/SKILL.md` owns the pre-delivery checklist. Where they differ, they win.
- Change presentation only. Keep the product's features, content and section order unless the task explicitly authorizes changing them.

## Symmetry redlines

Inspect the rendered screen when possible; otherwise inspect the code that renders it. Record each violation with `file:line`.

### One spacing grid

- Margins, paddings, gaps and fixed heights are multiples of 4 px. A 2 px nudge inside a small control and an odd value that exactly centres something are the only exceptions.
- Remove one-off values that break the rhythm, such as `mt-[13px]` or `py-[11px]`, and replace them with steps from the project's scale.
- Space shows grouping: the gap inside a group is smaller than the gap between groups.
- A section label sits closer to its own items than to the section above it. Starting values: 24 above, 8 below.

### Equal things, equal sizes

- All rows in a list share one height and one gap; all cards in a row share one padding.
- Sibling cards do not get different padding or one-off offsets.
- Counts and meta are quiet text, not glowing pills. A card that holds a number carries a label, the figure and a comparison — not a sparkline added for decoration.

### Three alignment columns

- Icons occupy one column with the same box width in every row.
- Labels start on one line, with a fixed gap of 8 or 10 px after the icon.
- Numbers align in one column, right aligned, with one format and tabular figures across the screen.
- Nothing is nudged off the line with a stray translate or an odd margin.

### Density by role

- Navigation is the densest part of an app: rows 32–36 px tall, 2–4 px between them. Section labels 24 above and 8 below.
- Cards use 20–24 px padding; page edges 24–32 px. Compact panels and sidebars are tighter.
- Row heights of 40–44 px are for touch targets, which `skills/accessibility/SKILL.md` also requires.
- Measure the sidebar after the pass: it is where alignment breaks most often.

### Nested radius

- An inner radius equals the outer radius minus the padding between them, never below 4 px. A 16 px card with 8 px padding holds items with an 8 px radius.
- Starting values: controls 8–10, cards 12–16, panels 14–20. Not everything is a pill.
- Buttons share the control radius; a square corner against the component style is a violation.

### App proportions

- An app fills the viewport: a sidebar of 240–280 px with fluid content, capped around 1440–1600 px. Do not shrink an app into a narrow centred column.
- The first screen is composed as a whole, with no dead empty band. On small screens everything stacks in reading order and the dense preview gets simpler instead of being zoomed down.

### Quiet chrome, one bright element

- Each screen has one polished thing that carries the richness. Navigation, buttons and text stay in one tinted neutral ramp with at most one accent.
- In apps and dashboards, backgrounds, surfaces, borders and text stay neutral; colour lives in the content, the selected state and the moments that mean something.
- The accent marks the primary action, the selection or one key figure, and covers a few percent of the screen.
- Depth comes from surface steps and hairlines, not from glows or coloured shadows. A gradient is only a small tone shift inside one neutral.

## Audit before editing

Walk the screen and list **every** sign, one line per sign with its location and its fix. Group the lines by surfaces and colour, type, icons, spacing, copy, components, heroes, visuals and motion. Do not fix while listing.

Then decide how much to change:

- **Heavy slop** — signs across several groups: run the full sequence below.
- **Light slop** — a mostly designed screen with a few signs: polish only what the audit found, take the direction from the existing brand, and keep its palette, type, layout and visual. Put bigger ideas in the report as suggestions.
- **No slop** — say so plainly, suggest at most two or three refinements with exact values, and apply them only if the user agrees. Never change a good screen just to make it different.

## Direction: five lines

Write these before touching anything, derive them from what the product does and who it is for, and show them to the user before editing. If the direction will be wrong, everything after it has to be redone.

- **Product** — what it does and for whom, in one plain sentence.
- **Character** — two or three concrete words. Words that fit every template (`clean`, `modern`, `sleek`, `minimal`, `premium`) are not allowed.
- **World** — the place or material the product belongs to. This sets the tint of the neutrals, the accent and the visual.
- **Moments** — one or two places where the screen gets rich colour and detail, chosen from what the product actually shows.
- **Signature** — one small detail only this product would have, used consistently. One, not five.

A real brand — a logo, colour or typeface deliberately chosen — is kept and built around. A hue-to-hue gradient, sparkles or glow are not a brand colour unless the user says they are.

## Nine passes, in order

Finish a pass across the whole screen before starting the next one.

1. **Spacing** — grid, equal sizes, grouping and alignment first; everything else reads differently once the rhythm is right.
2. **Colour** — one tinted neutral ramp, three text roles, one accent; remove hue gradients, glows, coloured shadows and blobs.
3. **Icons** — one line-icon set, one size and stroke (16 or 18 px at 1.5), the colour of their text role; replace emoji used as icons or decoration.
4. **Copy** — plain, specific labels; no greeting headline, exclamation marks, hype words or unverifiable claims; comparisons say what they compare to.
5. **Type** — chosen for the character, a few sizes, weights 400–500 with 600 for rare emphasis; sentence case, no tracked uppercase labels, no italic emphasis, no negative tracking on small text.
6. **Layout** — keep sections and order; align and re-proportion inside them; one primary action and at most one secondary; a hero has a real visual or a product preview with one moment.
7. **Details** — nested radii, quiet meta, honest charts with one accent, distinct table-row states, visible hover and focus, and the signature detail in its one place.
8. **Visuals** — remove decorative filler; show the product or one truthful scene from its world. Generated images are a separate, user-authorized action and are not required by this pass.
9. **Motion** — remove loops and bouncy or catch-all transitions; keep motion that explains a change, name the animated properties, and respect reduced motion.

Keep the pass bounded: at most three rendering checks unless the project's validation requires more. When nearly every line changes, plan the passes in order and then write the new screen in one go, as long as every pass is applied.

## Check

- [ ] The direction is written and every pass follows it.
- [ ] Every spacing value is on the project's scale and equal things have equal sizes.
- [ ] Icons, labels and numbers each sit in one aligned column.
- [ ] One neutral ramp, three text roles, one accent with verified contrast.
- [ ] No emoji used as icons or label decoration; one icon set, size and stroke.
- [ ] Numbers use one format with tabular figures.
- [ ] Nested radii add up; buttons share the control radius.
- [ ] Table header, value in use, selection and keyboard focus are each recognizable.
- [ ] The first screen is composed as a whole, with no dead empty band.
- [ ] Template check: if another product's name replaced this one, would the screen still fit? If yes, push the direction harder in copy, visual and signature.

Contrast, focus order, touch targets, reduced motion, RTL and locale formatting are checked with their owning skills, not inferred from this list.

## Not adopted from upstream

- `snapshot.mjs` and `story.mjs`: they require Playwright and ffmpeg and are not installed by this system.
- Image generation and the listed paid image services: generation runs only when the user asks and authorizes the cost.
- The numeric token tables as a house style: project tokens win.
- RTL rules: `skills/internationalization/SKILL.md` is the canonical source.
