# Linter Rules for Structural Compliance

## Purpose

Define the standard formatter and linter configuration a target project
receives, and state where the structural policy of this system is verified.

The project receives **configuration only**: `eslint.config.*`, the Prettier
configuration file, and their ignore files. The structural policy lives here,
in `ai-engineering`, and is verified here by `workflows/output-compliance.md`.
**No custom rule file, plugin, or checker script is created in a target
project**, and the policy adds no dependency to it.

This reference is the canonical guidance; the target project does not carry a
copy of it.

## What lives where

| Concern | Lives in |
|---|---|
| The policy and its wording | `AGENTS.md` in this system |
| The standard configuration | This reference |
| The verification of structural placement | `workflows/output-compliance.md` in this system |
| The project's lint and format configuration | The target project: `eslint.config.*`, `.prettierrc.*` or `prettier.config.*`, ignore files |

Anything beyond those configuration files is out of scope for a target
project. A project that contains `eslint-rules/`, a local plugin, or a checker
script under `scripts/` is carrying system tooling it should not carry, and
`CONFIG-001` reports it.

## TypeScript / JavaScript — standard configuration

`eslint.config.mjs` for the current ESLint major (flat config), adapted to the
detected major and to the framework plugins the stack already uses:

```js
import { defineConfig, globalIgnores } from "eslint/config";
import tseslint from "typescript-eslint";

export default defineConfig([
  globalIgnores(["node_modules/**", "dist/**", "build/**", "coverage/**"]),
  ...tseslint.configs.recommended,
  {
    files: ["**/*.{ts,tsx,js,jsx,mjs,cjs}"],
    rules: {
      "func-style": ["error", "expression"],
      "no-restricted-syntax": [
        "error",
        {
          selector: "VariableDeclarator > FunctionExpression",
          message: "Prefer an arrow function: const fn = () => {}.",
        },
      ],
      "@typescript-eslint/consistent-type-imports": "error",
    },
  },
]);
```

For a JavaScript-only project, drop the `typescript-eslint` import and the
rule that depends on it; keep `func-style` and `no-restricted-syntax`, which
are core rules.

- `func-style` and `no-restricted-syntax` are **ESLint core rules**. They
  express the arrow function convention and need no project code.
- `@typescript-eslint/consistent-type-imports` is part of `typescript-eslint`,
  which a TypeScript project already installs for its standard linting. When
  the project prefers the compiler to own the rule, `verbatimModuleSyntax` in
  `tsconfig.json` covers the same ground. Enable one, not both, to avoid
  duplicate reports.
- `typescript-eslint` is used only when the detected setup already includes
  it. It is never added for the structural policy.
- Prettier owns formatting. Do not re-enable ESLint rules that overlap with
  Prettier; configure the formatter in `.prettierrc.json` or
  `prettier.config.*` and leave formatting there.

For a legacy `.eslintrc` project, use the equivalent `rules` entries in the
format the detected ESLint major expects.

## What the standard configuration can and cannot express

| Rule | Expressable by standard configuration? | Mechanism |
|---|---|---|
| Arrow functions | Yes | ESLint core `func-style` + `no-restricted-syntax` |
| Type-only imports | Yes | `@typescript-eslint/consistent-type-imports` or `verbatimModuleSyntax` |
| One type per file, types separated from runtime behavior | No | Verified by `TYPE-001` in `workflows/output-compliance.md` |
| Contract grouping by consumer | No | Verified by `TYPE-002` |
| No pass-through re-export | No | Verified by `TYPE-003` |
| Unit folders and co-located tests | No | Verified by `FILE-001` and `workflows/code-audit.md` |

ESLint core has no rule for "a file declares types or runtime behavior", and
that is the point: expressing it at lint time requires a custom plugin
distributed as a package and unit-tested with `RuleTester`, which is a
different artifact from a configuration file. Copying a rule file into each
project is not that artifact, it is an unversioned fork of it. The system
verifies the rule itself, over the change set, and reports `file:line`
evidence.

## Rule applicability by ecosystem

Two rules are involved, and they do not apply to the same ecosystems.

**Rule A — types separated from runtime behavior, one contract file per
consumer.** Language agnostic, a policy of this system, verified in every
supported ecosystem by `workflows/output-compliance.md`. It is not installed
in the project.

**Rule B — prefer arrow functions.** A TypeScript and JavaScript convention
with no counterpart in Python, Java, Kotlin, Go, Rust, C#, PHP, or Ruby. Those
languages have no arrow syntax and their own conventions govern function
declarations. The system never imposes an alien convention on an ecosystem
that has its own.

### Coverage matrix

| Ecosystem | Rule A | Rule B |
|---|---|---|
| TypeScript / JavaScript | Verified by the system | Configured with `func-style` + `no-restricted-syntax` |
| React, Next.js, Angular, Vue, Svelte, Solid, React Native, Ionic, Capacitor, Node.js, Express, NestJS | Verified by the system | Configured, inherited from TypeScript / JavaScript |
| Go, Rust, Python, Django, FastAPI, Ruby, Kotlin, Spring (Kotlin) | Verified by the system | Not applicable |
| PHP, C# / .NET, Java, Spring (Java) | Verified by the system | Not applicable |
| SQL (PostgreSQL, MySQL, Oracle, MongoDB) | Not applicable — no type or module declarations | Not applicable |

Where an ecosystem linter ships a documented standard rule for Rule A —
Checkstyle `OneTopLevelClass`, StyleCop `SA1402`, PHP_CodeSniffer
`PSR1.Classes.ClassDeclaration` — the project may enable it in its normal
configuration, because that is configuration, not a new artifact. It is
optional: the system's check is the authority and runs whether or not the
project enables the rule.

## Installation requirements

- The formatter and the linter, with the plugins their detected stack already
  needs, are the mandatory exception in the dependency policy of `AGENTS.md`.
- The structural policy adds **no** dependency, file, plugin, or script to the
  project.
- A software project without a configured formatter or linter is
  `SETUP_REQUIRED`.
- A rule that no linter can express is covered by
  `workflows/output-compliance.md` and reported as covered by verification,
  never as an installed rule.

## Sources

- ESLint, *Custom Rules* — a custom rule is for a use case core rules do not
  cover, and it is distributed as a plugin with unit tests (`RuleTester`).
  Consulted 2026-10-27 for ESLint 10.
- ESLint, *func-style* — a frozen core rule; the documentation states that the
  choice is a preference, not a correctness rule. Consulted 2026-10-27 for
  ESLint 10.
- typescript-eslint, *consistent-type-imports* — a configuration rule that
  enforces type-only imports and overlaps with `verbatimModuleSyntax`.
  Consulted 2026-10-27 for typescript-eslint 8.
- Prettier, *Rationale* — the formatter prints code and does not transform it;
  formatting and linting are separate concerns. Consulted 2026-10-27 for
  Prettier 3.
