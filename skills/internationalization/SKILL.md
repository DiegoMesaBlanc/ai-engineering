---
name: internationalization
description: Implement internationalization and localization correctly so user-facing text, formatting, direction, and translation scale without rewrites.
---

# Internationalization

## Purpose

Build software that works correctly in more than one language and locale,
instead of retrofitting translation into hardcoded text. Internationalization
(i18n) is the structure; localization (l10n) is the per-locale content.

## When to Apply

Apply when the project or feature has user-facing text and any of these is
true:

- it must support more than one language
- it needs locale-aware dates, times, numbers, or currency
- it targets right-to-left (RTL) languages
- content will be translated later, even if only one language ships now
- user-generated or external content must be displayed in mixed directions

Do not force i18n on code with no user-facing text, such as a purely internal
service or a documentation-only change.

## Detect Before Implementing

Inspect the real project first:

- the existing i18n library, message format, and locale configuration
- how strings are currently stored and loaded
- the translation workflow and where catalogs live
- the default locale, supported locales, and fallback behavior
- existing date, number, and currency formatting
- whether the interface supports RTL today

Reuse the established approach. Do not introduce a second i18n library or
message format without a justified reason and human approval.

## Externalize All User-Facing Text

- Never hardcode user-facing strings in components, templates, or code.
- Use stable message keys with namespaces; do not reuse one key for
  different meanings.
- Never build sentences by concatenating translated fragments; word order
  differs across languages. Use a message with placeholders instead.
- Give translators context: what the text means, where it appears, and any
  length or tone constraints.
- Do not translate code identifiers, log messages, protocol values, or
  technical tokens that are not user-facing.

## Message Format

- Support pluralization through the existing message format (for example ICU
  MessageFormat or the project's equivalent); never assume a single plural
  form.
- Support select/gender variants where the language requires them.
- Keep placeholders typed and escaped correctly; never interpolate raw HTML
  or user input unsafely.
- Handle missing translations explicitly with a defined fallback instead of
  rendering a blank string or a raw key.

## Locale-Aware Formatting

- Format dates, times, numbers, currency, and units with the platform's
  locale-aware APIs (for example `Intl` and CLDR data) or the project's
  library; never hand-format these.
- Keep time zone and locale as separate concerns; a locale does not imply a
  time zone.
- Parse localized user input (numbers, dates, currency) through the same
  locale-aware layer, not with ad hoc splitting.
- Display currency with its locale conventions and an explicit currency code.

## Direction and Layout

- Write layout with logical properties (start/end) rather than hardcoded
  left/right so RTL mirrors correctly.
- Mirror directional icons and layout where the meaning is directional;
  never mirror universal symbols.
- Set the correct `lang` and `dir` attributes on content, especially for
  mixed-language or user-generated text.
- Allow text to expand; avoid fixed widths and truncation that break longer
  translations.

## Sorting, Case, and Comparison

- Sort and collate with locale-aware comparison, not raw code-unit order.
- Do not assume ASCII casing, word boundaries, or text length.
- Treat case-insensitive matching as locale-sensitive where the platform
  supports it.

## Translation Workflow

- Keep one source of truth for message catalogs.
- Keep keys stable; changing a key is a migration, not a rename.
- Version and review catalogs like code; never silently drop or overwrite
  translations.
- Keep the source language authoritative and handle missing or stale
  translations predictably.
- Do not machine-translate sensitive, legal, or safety-critical text without
  human review.

## Testing

- Test at least one non-default locale, and an RTL locale when RTL is
  supported.
- Use a pseudo-locale to surface hardcoded strings, truncation, and layout
  overflow.
- Assert that user-facing text comes from catalogs rather than literals.
- Cover plural, select, missing-translation, and formatting behavior.
- Include localization in accessibility and SEO checks: `lang`, `dir`, and
  `hreflang` where relevant.

## Performance and Delivery

- Load only the locales needed; avoid bundling every catalog into the client.
- Cache catalogs and avoid refetching on every navigation.
- Prefer the project's existing loading and caching strategy.

## Module Layout

Separate i18n concerns into distinct files within the module:

- **Types/Contracts** (`types.ts`): `Locale`, `LocaleMeta`, message shape
  interfaces (`PaymentSelectorTranslations`).
- **Configuration** (`config.ts`): `SUPPORTED_LOCALES`, `LOCALE_META`,
  accessor functions (`getLocaleMeta`, `getCurrencyForLocale`,
  `getDirectionForLocale`).
- **Message Catalogs** (`messages/<locale>.ts`): one file per locale
  (`en.ts`, `es.ts`, `ar.ts`) containing the translation dictionary.
  This enables lazy loading and independent translation workflows.
  If lazy loading is not required, a single `messages.ts` is acceptable.
- **Formatters** (`format.ts`): `formatExpiry`, `formatCurrency`,
  `formatNumber`, `formatPercent`, and other locale-aware formatters.
- **Public API (optional)** (`index.ts`): re-exports `getTranslations`,
  formatters, and types only when multiple entry points or consumers
  justify a barrel; never add one to re-export a single file.

Do not mix contracts, catalogs, config, and formatters in one file
(e.g., a monolithic `i18n.ts`). Split when the file would change for
unrelated reasons (translations vs. formatting vs. locale config) or a
consumer cannot import the contract without pulling in data/runtime.

## Relation to Other Components

- `architecture-advisor` decides where localization belongs in the structure.
- `implementation-recipes` implements the surrounding stack conventions.
- `design-analysis` and `frontend-design` shape the interface; this Skill
  makes it locale-aware.
- `accessibility` covers `lang`/`dir` and screen-reader behavior.
- `seo` covers `hreflang` and localized metadata.
- `testing-strategy` selects the depth of locale tests.
- `dependency-assessment` governs any new i18n library.

## Decision Gates

Introduce a new i18n library or message format; change the default locale or
fallback strategy; or add locales that materially affect scope, content, or
cost only with human approval.

## Output

### I18N PLAN

- Detected library, message format, and locale configuration
- Supported locales, default locale, and fallback behavior
- Text externalization approach and key conventions
- Plural, select, and formatting approach
- Direction and RTL support
- Translation workflow and catalog ownership
- Testing approach across locales and RTL
- Accessibility and SEO considerations
- New dependencies and required approvals
- Risks and exclusions

## Safety

- Never hardcode user-facing text or concatenate translated strings.
- Never invent translations or present machine output as reviewed content.
- Never break existing locales, keys, or catalogs silently.
- Never store secrets or sensitive data in message catalogs.
