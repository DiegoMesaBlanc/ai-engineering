# Styling Options

## Purpose

The styling approach of a frontend or mobile project is a decision, and the
options that can be offered depend on the framework. Styled Components is a
React and React Native library; an Angular project does not use it, and a
React Native project does not use CSS files. This reference lists, per
framework, the styling options that framework supports, the options it
excludes, the signals that detect the framework, and the source each option
was verified against.

It is the reference for the styling approach of a new frontend or mobile
project (`references/project-startup-decisions.md`, "Frontend Decisions" and
"Mobile Decisions") and for detecting the approach of an existing project
(`skills/stack-detection/SKILL.md`).

## How to use

The styling approach is settled at the **start** of a project's creation, not
in the middle of it, because a scaffold tool can fix a styling mechanism on
its own (the Next.js scaffold, for example, includes Tailwind CSS by default).

1. **New frontend or mobile project**: once the framework is confirmed in
   Phase 2 of `workflows/project-initialization.md`, ask the styling question
   **first**, before the startup decision set and before any scaffold step.
   Present the options of the detected framework from "Options by framework",
   with the recommendation, the trade-off, and the excluded options with their
   reason. Never offer an option the framework does not support.
2. **Existing project**: do not ask. Detect the styling approach the repository
   already uses, from the evidence in its manifests and source, record it with
   that evidence, and continue. When the repository mixes two approaches,
   report the second one as a finding with its files; keep following the one
   the project's UI uses most, and state that choice in the report.
3. **A framework not listed here**: consult its official documentation for the
   detected version, offer only the options it describes, and record the
   source. When no source can be consulted, report the options as `NOT
   VERIFIED` and ask the user.
4. Record the answer, or the detected approach, in the startup decision set,
   with the source of each option.

## Detection

The signals are the files and dependencies a repository contains. Check the
signals in this order; the first match decides, and a later match is only a
secondary signal (for example, `react` is present in React Native and Next.js
projects, but it does not decide them).

| Order | Framework | Signals |
|---|---|---|
| 1 | Ionic | `@ionic/angular`, `@ionic/react`, or `@ionic/vue` in `package.json`; the underlying framework is then detected from the same manifest |
| 2 | Angular | `angular.json`, or `@angular/core` in `package.json` |
| 3 | React Native | `react-native` in `package.json`, or `expo` with an `app.json` or `app.config.*` |
| 4 | Next.js | `next` in `package.json`, or `next.config.*` |
| 5 | Vue | `vue` in `package.json`, or `.vue` single-file components |
| 6 | React (web) | `react` and `react-dom` in `package.json`, without the signals above |
| 7 | Svelte, Solid, and other frameworks | Detected from their manifests; their options are not listed here, so the rule for other frameworks applies |

## Options by framework

Verification columns: `Verified` means the official documentation was read
for this table; `NOT VERIFIED` means the option is listed from common usage and
must be checked against the official documentation before it is offered as a
fact.

### Angular

Verified against the Angular "Styling components" guide (Angular v22, consulted
2026-11-05).

| Option | Mechanism | Trade-off | Verified |
|---|---|---|---|
| Component styles (CSS) | `styles` or `styleUrl` on the component; emulated view encapsulation by default | Built in, no dependency; scoped by default, with `ViewEncapsulation` modes when needed | Verified |
| Component styles in Sass, Less, or Stylus | The same mechanism, with a preprocessor that outputs CSS | Adds a preprocessor dependency; the guide states that Angular works with any tool that outputs CSS | Verified |
| Tailwind CSS | Utility classes in templates, through the Tailwind Angular guide | Adds a build dependency; utility classes in templates | NOT VERIFIED (the Angular guide index links it; read it before offering) |

Recommendation for a new Angular project: component styles in CSS or SCSS.

### React (web)

No React-specific styling guide was read for this table. The options below
are the ones the Next.js styling guide documents for React applications; a
plain React project must confirm them against its bundler's documentation.

| Option | Mechanism | Trade-off | Verified |
|---|---|---|---|
| CSS Modules | `*.module.css` imported as an object of class names | Local scope with no runtime; needs the bundler's CSS Modules support | Verified through the Next.js guide; NOT VERIFIED for a plain React bundler |
| Tailwind CSS | Utility classes | Fast for most layouts; a build dependency | Verified through the Next.js guide |
| Sass | Preprocessor that outputs CSS | Adds a preprocessor dependency | Verified through the Next.js guide |
| CSS-in-JS (styled-components, Emotion) | Styles written in JavaScript | Runtime cost and a library dependency; check its compatibility with the framework and rendering mode | CSS-in-JS as a category is listed by the Next.js guide; the named libraries and their versions are NOT VERIFIED |

Recommendation for a new React project: CSS Modules or Tailwind CSS.

### Next.js

Verified against the Next.js CSS guide (last updated 2026-08-25, consulted
2026-11-05).

| Option | Mechanism | Trade-off | Verified |
|---|---|---|---|
| Tailwind CSS | Utility classes, configured through PostCSS | The guide recommends it for most styling; a build dependency | Verified |
| CSS Modules | `*.module.css` next to the component | Local scope; the guide recommends it when utilities are not enough | Verified |
| Global CSS | A stylesheet imported in the root layout | Applies to every route; for truly global styles only | Verified |
| External stylesheets | Stylesheets from packages, imported in the app | Depends on the package | Verified |
| Sass | Preprocessor, documented in its own guide | Adds a preprocessor dependency | Verified |
| CSS-in-JS | Documented in its own guide | Confirm the library's support for the App Router before choosing it | Verified as an option; library details NOT VERIFIED |

Recommendation for a new Next.js project: Tailwind CSS, with CSS Modules for
component-specific styles that utilities do not cover.

### Vue

Verified against the Vue "SFC CSS Features" page (consulted 2026-11-05).

| Option | Mechanism | Trade-off | Verified |
|---|---|---|---|
| Scoped styles | `<style scoped>` in the single-file component | Built in, no dependency; child root elements are affected by the parent's scoped styles | Verified |
| CSS Modules | `<style module>`, read with `$style` or `useCssModule` | Built in; hashed class names | Verified |
| Dynamic values in CSS | `v-bind()` inside `<style>` | Built in; reactive values in styles | Verified |
| Tailwind CSS or Sass | Through the project's build tooling | Adds a build dependency | NOT VERIFIED in this run |

Recommendation for a new Vue project: scoped styles in the single-file component.

### React Native (Expo or bare)

Verified against the React Native "Style" page (version 0.88, consulted
2026-11-05) and the NativeWind homepage (consulted 2026-11-05).

| Option | Mechanism | Trade-off | Verified |
|---|---|---|---|
| `StyleSheet` | `StyleSheet.create` in the component file, or a styles file next to it | Built in, no dependency; the documentation recommends it as components grow | Verified |
| NativeWind | Tailwind utility classes in the `className` prop | Adds a dependency and build configuration; the homepage shows a pre-release v5 line, so the version must be confirmed before adoption | Verified (homepage); version NOT VERIFIED |
| Styled Components for React Native | Styles written in JavaScript | Runtime cost and a library dependency | NOT VERIFIED in this run |
| Other native CSS-in-JS libraries (for example, Unistyles, Tamagui) | Library-specific | Library dependency and native configuration | NOT VERIFIED in this run |

Recommendation for a new React Native project: `StyleSheet`, which needs no
dependency. Offer NativeWind when the team already uses Tailwind.

### Ionic

Ionic uses the styling options of its underlying framework: Ionic Angular
follows the Angular list, Ionic React follows the React list, and Ionic Vue
follows the Vue list. Ionic's own CSS variables and utilities are an addition
that is NOT VERIFIED in this run.

## Excluded options

The options below are not offered for the framework on the left. Offering one
of them is a defect of the proposal, not a choice.

| Framework | Excluded option | Reason |
|---|---|---|
| Angular | Styled Components, Emotion, NativeWind | React and React Native libraries; Angular has its own component styles mechanism |
| Angular | CSS Modules as a built-in mechanism | Not part of the Angular styling mechanism; use component styles |
| Vue | Styled Components, Emotion, NativeWind | React and React Native libraries |
| Vue | Angular component styles | Angular-specific mechanism |
| React (web) | Angular component styles, Vue `<style scoped>` | Framework-specific mechanisms |
| Next.js | Angular component styles, Vue `<style scoped>`, React Native `StyleSheet` | Framework-specific mechanisms |
| React Native | CSS files, Sass, Less, Stylus, CSS Modules, Tailwind CSS for the web | Native rendering does not read CSS files or web stylesheets |
| React Native | Angular component styles, Vue `<style>` blocks | Framework-specific mechanisms |

## Source kinds

Every option names its source. The source kind is one of:

- an ecosystem official rule or guide (Angular, Next.js, Vue, React Native);
- a library's own documentation (NativeWind, the CSS-in-JS libraries);
- a policy of this system (the recommendations, the exclusions, and the
  question rule).

The recommendations are a policy of this system, not a framework requirement.
A framework's own documentation wins when the two differ.

## Asking rule

- The styling approach of a **new** frontend or mobile project is the first
  question of its creation, asked once the framework is confirmed, before any
  other decision and before any scaffold step.
- The question lists only the options of the detected framework, with the
  recommendation, the trade-off, and the excluded options with their reasons.
- An **existing** project is never asked. Its styling approach is detected and
  followed, and mixed approaches are reported as a finding.
- The rule does not apply to a backend or a library without a UI.

Version rule: the options are checked against the version the project targets
(`references/project-startup-decisions.md`, "Version and documentation"). An
option whose version guide was not consulted is reported as `NOT VERIFIED`.
