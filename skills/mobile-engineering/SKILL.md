---
name: mobile-engineering
description: Apply mobile-specific lifecycle, platform, storage, navigation, and test constraints when building or reviewing a native or cross-platform mobile application.
---

# Mobile Engineering

## Purpose

Carry the mobile-specific engineering constraints that a generic web or
backend skill does not express. This skill is a shared capability: a `mobile`
subagent loads it at `senior-engineer` seniority during the `build` phase, and
any other domain may load it when its change touches a mobile surface.

## When to load

Load when the detected stack is native Android, native iOS, React Native,
Flutter, Ionic, or another mobile target, and when a change affects a mobile
screen, platform API, or mobile build.

## Procedure

1. Detect the concrete target and version before choosing guidance:
   native iOS, native Android, React Native, Flutter, or Ionic. Consult
   `references/official-documentation.md` for that version and record the
   consultation.
2. For folder structure, load `architecture-advisor` at `architect` seniority
   and follow `references/folder-architectures.md`. That reference defines the
   React Native feature tree and the `android/` and `ios/` root projects.
   Flutter is not present in that reference: use the official Flutter structure
   guidance and record the deviation in the returned checklist.
3. Decide the styling approach from `references/styling-options.md` for the
   detected framework. A new mobile project asks this before any scaffold. An
   existing project's approach is detected and followed, never asked.
4. Apply platform capabilities through a port. A permission request, system
   settings, a system alert, or a device or accessibility state read is a
   dependency the composition root binds; a hook, feature, or screen does not
   import the platform module directly. A component that only renders platform
   primitives is exempt.
5. Apply storage rules: credentials only in the secure store; personal and
   payment data never in plain storage; identity-owned caches partitioned by
   identity and cleared on sign-out; sensitive files written owner-only;
   uploaded images stored with metadata stripped.
6. Apply backup rules: personal data excluded from device and cloud backup by
   `allowBackup="false"` or data-extraction rules on Android and the iOS
   equivalent. Verify in the build configuration or built manifest; without a
   build, report `NOT VERIFIED`.
7. Apply lifecycle rules: cancel subscriptions, listeners, timers, and requests
   on unmount; check mounted state after an await before navigation or state
   updates; handle loading, success, error, empty, and offline states.
8. Apply navigation and deep-link rules: validate and sanitize external URLs
   before acting on them, and keep protected routes behind an auth guard that
   reads the verified session.
9. Apply locale rules when the requirement names them: resolve direction from
   the active locale or `I18nManager.isRTL` where the app disables automatic
   mirroring, use locale-aware formatters with an explicit locale, and assert
   each direction-dependent unit under an RTL locale.
10. Run the platform test tree named in `references/folder-architectures.md`.
    A test case is never placed at the source root.

## Boundaries

- Never weaken a gate to fit a platform constraint. Report the conflict.
- Never claim an announcement or screen-reader behavior is `PASS` on a platform
  the check did not exercise. Report it `NOT VERIFIED` with the platform named.
- Every external call has a client-enforced timeout; otherwise report it
  `NOT VERIFIED`.

## Output

Return the detected target, the structure decision, the platform constraints
applied, deviations with their reasons, and the validation commands for that
target. End with `TERMINADO` or `DETENIDO` with the reason.

## References

- `architecture-advisor`, `internationalization`, `accessibility`,
  `design-analysis`, `stack-detection`
- `references/folder-architectures.md`, `references/styling-options.md`,
  `references/official-documentation.md`
- `workflows/output-compliance.md`
