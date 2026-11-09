# Web Interface Checklist

A grep-able review checklist for web UI code. Use it in a review or in the
pre-delivery checks of `skills/ui-ux-design/SKILL.md`. Report each finding as
`file:line` with the rule and a short fix.

## Source and adaptation

- Source: `vercel-labs/web-interface-guidelines`, file `command.md`.
- Pinned commit: `434b7f91364665f2f733b310ec54809bf8f37937` (2026-10-05).
  Verified 2026-11-09: the pinned `command.md` is byte-identical to `main`
  (SHA-256 `d246b026f4f29b5823a9cc857f9edf3d2507002e055e32040cadeaf3b38e0234`).
- License: MIT, Copyright (c) 2025 Vercel Labs. The rules are adapted into this
  file, not copied verbatim.
- Rules that overlap `skills/accessibility/SKILL.md` (focus, ARIA, keyboard,
  reduced motion, target size, labels) are not repeated here. Apply that skill
  for them.
- The source is not fetched at review time. A new upstream revision is adopted
  only on request, through the same procedure as the `ui-ux-design` pin.

## Conflicts resolved

- **Title Case for headings and buttons** (upstream) is **not adopted**. The
  Engineering System uses sentence case (`skills/frontend-design/SKILL.md`).
- **React-specific examples** (`<Link>`, `forwardRef`, `useState`) are
  translated to the detected stack. The rule applies to every stack; the
  wording is not React-only.

## Forms

- Inputs have a meaningful `name` and an `autocomplete` value.
- Use the correct input type (`email`, `tel`, `url`, `number`) and `inputmode`.
- Never block paste (no `preventDefault` on paste events).
- Turn off spellcheck on email, code, and username fields.
- Use `autocomplete="off"` on non-authentication fields that trigger password
  managers.
- Keep a submit button enabled until the request starts; show a progress
  indicator while it runs.
- Show errors inline next to the field and move focus to the first error on
  submit.
- Placeholders end with an ellipsis and show the expected pattern.
- Warn before leaving a page with unsaved changes.

## Animation and rendering

- Animate `transform` and `opacity` only.
- Never use `transition: all`; list the properties.
- Set `transform-origin` to the element that really moves.
- Animations can be interrupted by user input.
- Any autoplaying motion longer than 5 seconds next to other content has a
  pause, stop, or hide control.

## Typography

- Use the ellipsis character (`…`), not three dots.
- Use typographic quotes, not straight quotes, in running text.
- Use a non-breaking space between a number and its unit (`10 MB`) and in
  keyboard shortcuts.
- Loading states end with an ellipsis (`Loading…`, `Saving…`).
- Use tabular numerals in number columns and comparisons.
- Use `text-wrap: balance` or `text-pretty` on headings.

## Content handling

- Text containers handle long content: truncate, clamp, or break words.
- Flex children that must truncate allow shrinking below their content width.
- Empty lists and empty strings render an intentional empty state, not a broken
  layout.
- User-generated content is checked for short, average, and very long values.

## Images and performance

- Every `<img>` has explicit `width` and `height` (prevents layout shift).
- Images below the fold use lazy loading; the critical above-the-fold image is
  prioritized.
- Lists above about 50 items are virtualized or use `content-visibility`.
- No layout reads (`getBoundingClientRect`, `offsetHeight`, `scrollTop`) inside
  render.
- Animated GIFs are replaced by a compressed video with a still fallback.
- Critical fonts are preloaded with a display strategy that avoids invisible
  text.

## Navigation and state

- State that a user can share (filters, tabs, pagination, expanded panels) is
  reflected in the URL.
- Navigation uses real links, so cmd/ctrl-click and middle-click work.
- Destructive actions require a confirmation or an undo window; they are never
  immediate.

## Touch and layout

- Interactive controls set `touch-action: manipulation` to avoid double-tap
  delay.
- Modals, drawers, and sheets set `overscroll-behavior: contain`.
- Gestures (drag, swipe, pinch) have a tap or keyboard alternative.
- Full-bleed layouts respect `env(safe-area-inset-*)`.
- No unwanted horizontal scrollbar: fix the overflow instead of hiding it.

## Theming and locale

- Dark themes set `color-scheme: dark` on the root element.
- Native `<select>` elements set an explicit background and text color.
- Dates, times, numbers, and currency use the locale formatters
  (`Intl.DateTimeFormat`, `Intl.NumberFormat`), not hardcoded formats.
- The language is detected from the browser language preferences, not from the
  IP address.
- Brand names and code identifiers are marked `translate="no"`.
- See `skills/internationalization/SKILL.md` for the full locale rules.

## Interaction states

- Buttons and links have a hover state.
- Hover, active, and focus states have more contrast than the resting state.

## Copy

- Use the active voice: "Install the CLI", not "The CLI will be installed".
- Use sentence case for headings and buttons (see Conflicts resolved).
- Use numerals for counts: "8 deployments", not "eight deployments".
- Use specific button labels: "Save API key", not "Continue".
- Error messages say what to do next, not only what failed.
- Address the user in the second person; avoid first person.
- Use "&" instead of "and" where space is tight.

## Anti-patterns to flag

- Zoom disabled (`user-scalable=no`, `maximum-scale=1`).
- Paste blocked (`preventDefault` on a paste handler).
- `transition: all`.
- Clickable `<div>` or `<span>` instead of a button or link.
- Images without dimensions.
- Long lists rendered without virtualization.
- Hardcoded date or number formats.
- `autoFocus` without a clear reason, or on mobile.
- Gesture-only actions with no tap or keyboard alternative.
- Animated GIF where a compressed video fits.
