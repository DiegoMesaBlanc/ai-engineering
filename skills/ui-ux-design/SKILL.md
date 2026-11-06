---
name: ui-ux-design
description: UX and UI design intelligence for frontend, mobile, and desktop projects, delivered by the external skill ui-ux-pro-max (pinned commit). Use when designing or reviewing a user interface, choosing a visual direction for a product type, or checking UX rules for a specific framework, combined with this system's own skills.
license: External skill, MIT (nextlevelbuilder/ui-ux-pro-max-skill). Referenced, not copied; see the registry entry.
---

# UI/UX Design

This skill is a **reference**, not a copy. The design data and search engine live in the external skill `ui-ux-pro-max`. This file records which version to use, how it works with our skills, and where its guidance must be corrected.

## Source and pin

- Repository: https://github.com/nextlevelbuilder/ui-ux-pro-max-skill
- Pinned commit: `b7d5353e0130feb8c5574ef1251f8a1db1873ccb` (2026-11-05), MIT.
- The installed copy must match the pinned commit. This system never installs it: the user installs it with the upstream CLI or plugin in their own environment.
- When the installed copy is missing, do not use its guidance. State that the design data is NOT AVAILABLE and continue with the skills of this system.
- Its search script needs Python 3 on the user's machine. If Python is missing, ask the user to install it. Never install software for this skill.

## How it works with this system

Use the external skill for **data and reasoning about product type, style, palette, typography, and stack-specific UX rules**. Use this system's skills for everything else:

| Need | Skill to use |
|---|---|
| Aesthetic direction, avoiding templated visuals | `frontend-design` |
| WCAG 2.2, keyboard, screen readers | `accessibility` (its rules prevail over the external data) |
| Measured performance (LCP, INP, CLS) | `core-web-vitals`, `performance` |
| Code structure and stack conventions | `implementation-recipes`, `styling-options` |
| Turning a Figma, screenshot, or brief into requirements | `design-analysis` |
| Localization and RTL | `internationalization` |

Procedure:

1. Detect the stack with `stack-detection`. Use the external skill's stack guidance for that stack only.
2. Ask the external skill for a design starting point for the product type. Treat its answer as a proposal, then confirm the visual direction with `frontend-design`.
3. Apply UX rules by platform. Then run the pre-delivery checks in this file.
4. Record the external skill's version (commit) in the task context.

## Corrections to the external guidance

These rules are not applied as written. They override the external output wherever it conflicts. Each one comes from a checked source or from an observed contradiction.

- **React RSC security.** The external rule "Require React 19.2.1+ for RSC code paths" is incomplete. The advisory at https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components lists later CVEs with newer patched versions. Use the **latest patched release** in the 19.x line, and only for apps that use React Server Components or server functions.
- **Next.js server functions and proxy.** Use `proxy.ts`, not `middleware.ts`. Server functions are not routes in the proxy chain, so a proxy matcher that excludes a path also skips its server functions. Authorize inside each server function; never rely on proxy alone. Source: https://nextjs.org/docs/app/api-reference/file-conventions/proxy
- **Forwarding refs in React 19.** Pass `ref` as a prop. Do not use `forwardRef` in new code. Source: https://react.dev/reference/react/forwardRef
- **Web-only rules.** Hover-only interaction, `cursor-pointer`, and similar pointer rules do not apply to touch-first mobile targets. On touch devices, every action must also be reachable by tap.
- **Universal durations.** Do not apply one transition duration to every element. Duration depends on distance, element, platform, and user preference, and must respect reduced motion.
- **Accent color in restricted domains.** The generator may recommend a purple accent and also list purple gradients as an anti-pattern. Choose the accent from the product's mood and the contrast check, not from the first output.
- **Angular loops (to verify).** The external stack rule for Angular says `trackBy` in `@for`. Current Angular control flow uses `track`. This is not yet checked against https://angular.dev; check it before applying the rule.
- **Version labels.** The external data labels some stacks with older minor versions (for example React 19.2.x, Next.js 16.2, Nuxt 4.5, Astro 7.1.6). Confirm the current version with the official registry or documentation before using a version-specific rule.

## Pre-delivery checklist

Mark each item PASS, FAIL, or NOT APPLICABLE with a reason.

- [ ] Icons are SVG from a single library. No emoji are used as functional icons.
- [ ] Text contrast meets WCAG 2.2 AA (4.5:1 normal text, 3:1 large text and UI components).
- [ ] Focus is visible for keyboard users, and focus order matches visual order.
- [ ] Every interactive element has a target size appropriate to its platform.
- [ ] Motion respects the reduced-motion preference.
- [ ] Text reflows without clipping at narrow widths, browser zoom, and text scaling.
- [ ] Layout is checked at the project's supported breakpoints or device sizes.
- [ ] Status and error states are not conveyed by color alone.
- [ ] Loading, empty, and error states exist for each asynchronous view.

## Writing rules

- The external skill's `--persist` option writes a `design-system/` folder into a project. Use it only when the user names the project path and authorizes that write.
- Code examples follow `AGENTS.md`: arrow functions in TypeScript and JavaScript, and types in their own file.
- Search results are guidance. They do not replace user testing, accessibility testing on real devices, or the project's design system.

## Updating the pin (on demand)

This check runs only when someone asks for it, or before a task that relies on this skill's design data. Nothing updates automatically. The steps are read-only until step 5.

1. **Compare the pin with the upstream head** (read-only):

   ```sh
   git ls-remote https://github.com/nextlevelbuilder/ui-ux-pro-max-skill refs/heads/main
   ```

   If the hash equals the pinned commit, the pin is current and the check ends here.

2. **List the upstream changes since the pin** (read-only):

   ```sh
   curl -s -H "Accept: application/vnd.github+json" \
     "https://api.github.com/repos/nextlevelbuilder/ui-ux-pro-max-skill/compare/b7d5353e0130feb8c5574ef1251f8a1db1873ccb...main" \
     | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['status'], d['behind_by'], 'behind'); [print(f['filename']) for f in d['files']]"
   ```

   Pay attention to changes under `src/ui-ux-pro-max/data/` (rules and stack guidance), `src/ui-ux-pro-max/scripts/`, and `src/ui-ux-pro-max/templates/base/skill-content.md` (the instructions the agent reads).

3. **Re-check the corrections above** against their sources. For each correction, record whether the upstream change fixes the problem, keeps it, or changes it.

4. **Review the changed rows.** For changed stack guidance, check the cited official documentation before accepting the new rule.

5. **Move the pin only after the user approves.** Update the commit in this file and in the registry entry, and record the date and the review result. Remove a correction only when the upstream data no longer has the problem.

If the upstream has no changes in these paths, record "no relevant change" and keep the pin.
