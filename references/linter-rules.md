# Linter Rules for Structural Compliance

## Purpose

Canonical, copy-ready linter configuration and rules that enforce the
structural policy of the system **at lint time**, inside the target project.

Prose in a Skill is not enforcement. A rule installed in the project's lint
configuration runs on every lint, every commit, and every CI job. This
reference is the artifact the system materializes in the target project
during the mandatory formatting and linting setup.

Authority: `skills/implementation-recipes/SKILL.md` (rules),
`skills/architecture-advisor/SKILL.md` (placement),
`references/official-documentation.md` (rule documentation).

## What each rule enforces

| Rule | Enforces | Applies to |
|---|---|---|
| `func-style: ["error", "expression"]` | No `function` declarations; arrow functions preferred | TypeScript, JavaScript |
| `no-restricted-syntax` (`VariableDeclarator > FunctionExpression`) | No `function` expression assigned to a variable | TypeScript, JavaScript |
| `structural/no-mixed-types-and-runtime` (local rule) | A file declares types **or** runtime behavior, never both | TypeScript, JavaScript |
| `structural/one-contract-file-per-consumer` (local script) | Contract files group types by consumer | TypeScript, JavaScript |
| `react/no-multi-comp` | One component per file, only when `eslint-plugin-react` is already installed | React, Next.js |
| Checkstyle `OneTopLevelClass` | One top-level type per file | Java |
| StyleCop `SA1402` | One type per file | C#, .NET |
| PHP_CodeSniffer `PSR1.Classes.ClassDeclaration` | One class per file | PHP |

The type-per-file rule is language agnostic. The arrow function rule is a
TypeScript and JavaScript convention and has no counterpart elsewhere. See
**Rule applicability by ecosystem** for the full coverage matrix.

## TypeScript / JavaScript

### 1. Arrow functions

`func-style` with the `"expression"` option is the correct core rule:

- Arrow functions are **always allowed**; the option does not restrict them.
- A callback passed as an argument to another function is **not** checked.
- TypeScript **overload signatures** are not reported.
- `export default function` is not reported.
- A **generator** (`function*`) **is** reported. Keep it and justify it with an
  inline disable:

  ```ts
  // eslint-disable-next-line func-style -- generator requires the function keyword
  export function* streamEvents(): Generator<Event> {
    /* ... */
  }
  ```

  A recursive function rarely needs the keyword: `const f = () => { f(); }` is
  valid because the call happens after initialization. Use `function` only for
  a real `this` binding, the `arguments` object, or `new.target`.

`func-style` is **not auto-fixable**: ESLint provides no fixer for it. The
transformation is mechanical and is applied by the agent or the developer:

```ts
// before
export function deriveSelectedMethod(methods: SavedPaymentMethod[]) {
  return methods.find((method) => method.isDefault);
}

// after
export const deriveSelectedMethod = (methods: SavedPaymentMethod[]) => {
  return methods.find((method) => method.isDefault);
};
```

Watch the return type: a `function` declaration carries it after the
parameter list, while an arrow assigned to a variable carries it after the
arrow (`const f = (a: A): R => {}`).

### 2. No `function` expression assigned to a variable

`func-style` alone permits `const fn = function () {}`. Forbid it explicitly:

```js
"no-restricted-syntax": [
  "error",
  {
    selector: "VariableDeclarator > FunctionExpression",
    message: "Prefer an arrow function: const fn = () => {}.",
  },
],
```

The selector targets only a `function` expression directly assigned to a
variable. It does not match class methods, object methods, or callbacks.

### 3. Local rule — no mixed types and runtime

ESLint core has no rule for this. Add a local rule to the project.

Create `eslint-rules/structural-compliance.mjs`:

```js
/**
 * Local ESLint rules that enforce the structural policy of the project.
 *
 * - no-mixed-types-and-runtime: a file declares type declarations or runtime
 *   behavior, never both. Types belong in a dedicated contract file.
 *
 * A contract file may contain type declarations, schema declarations that
 * pair with an inferred type, and literal-union constants (`as const`).
 */

const TYPE_DECLARATIONS = new Set([
  "TSInterfaceDeclaration",
  "TSTypeAliasDeclaration",
]);

const RUNTIME_DECLARATIONS = new Set([
  "FunctionDeclaration",
  "ClassDeclaration",
  "TSEnumDeclaration",
  "TSModuleDeclaration",
]);

const SCHEMA_CALLS = [
  "z.object",
  "z.array",
  "z.string",
  "z.number",
  "z.boolean",
  "z.enum",
  "z.union",
  "z.discriminatedUnion",
  "z.literal",
  "z.record",
  "Joi.object",
  "Joi.array",
  "Joi.string",
  "Joi.number",
  "Joi.boolean",
  "yup.object",
  "yup.array",
  "yup.string",
  "yup.number",
  "yup.boolean",
];

const noMixedTypesAndRuntime = {
  meta: {
    type: "problem",
    docs: {
      description:
        "Disallow files that declare types and runtime behavior together.",
    },
    schema: [],
    messages: {
      mixed:
        "File mixes type declarations with runtime behavior. Extract the types to a dedicated contract file (types.ts / contracts.ts) in the owning layer folder.",
    },
  },
  create(context) {
    const sourceCode = context.sourceCode;

    const unwrap = (statement) => {
      if (
        statement.type === "ExportNamedDeclaration" ||
        statement.type === "ExportDefaultDeclaration"
      ) {
        return statement.declaration ?? null;
      }
      return statement;
    };

    const isContractSafeVariable = (declarator) => {
      const text = sourceCode.getText(declarator);
      if (text.includes("as const")) {
        return true;
      }
      const init = declarator.init;
      if (!init || init.type !== "CallExpression") {
        return false;
      }
      const callee = sourceCode.getText(init.callee);
      return SCHEMA_CALLS.some((schema) => callee.startsWith(schema));
    };

    return {
      "Program:exit"(program) {
        const typeNodes = [];
        const runtimeNodes = [];

        for (const statement of program.body) {
          const node = unwrap(statement);
          if (!node) {
            continue;
          }

          if (TYPE_DECLARATIONS.has(node.type)) {
            typeNodes.push(node);
            continue;
          }

          if (RUNTIME_DECLARATIONS.has(node.type)) {
            runtimeNodes.push(node);
            continue;
          }

          if (node.type === "VariableDeclaration") {
            const allContractSafe = node.declarations.every(
              isContractSafeVariable,
            );
            const target = allContractSafe ? typeNodes : runtimeNodes;
            target.push(node);
            continue;
          }

          if (node.type === "TSDeclareFunction") {
            typeNodes.push(node);
          }
        }

        if (typeNodes.length > 0 && runtimeNodes.length > 0) {
          context.report({ node: typeNodes[0], messageId: "mixed" });
        }
      },
    };
  },
};

export default {
  rules: {
    "no-mixed-types-and-runtime": noMixedTypesAndRuntime,
  },
};
```

Wire it in the flat config (`eslint.config.mjs` for ESLint 9 and later):

```js
import { defineConfig, globalIgnores } from "eslint/config";
import structuralCompliance from "./eslint-rules/structural-compliance.mjs";

export default defineConfig([
  {
    files: ["**/*.{ts,tsx,js,jsx,mjs,cjs}"],
    plugins: {
      structural: structuralCompliance,
    },
    rules: {
      "func-style": ["error", "expression"],
      "no-restricted-syntax": [
        "error",
        {
          selector: "VariableDeclarator > FunctionExpression",
          message: "Prefer an arrow function: const fn = () => {}.",
        },
      ],
      "structural/no-mixed-types-and-runtime": "error",
    },
  },
  globalIgnores([".next/**", "dist/**", "build/**", "coverage/**"]),
]);
```

When the project uses the legacy `.eslintrc` format, use the equivalent
`plugins` and `rules` entries. Keep the format the detected ESLint major
expects.

### 4. One contract file per consumer

ESLint analyzes one file at a time, so the consumer grouping is verified by a
small script with no dependencies. Create `scripts/check-contract-consumers.mjs`:

```js
/**
 * Verifies that every contract file groups types with the same consumer.
 *
 * A type consumed by a different set of files than its siblings belongs in
 * a different contract file. Exit code 1 means a violation was found.
 */

import { readFileSync, readdirSync, statSync } from "node:fs";
import { join, extname, relative } from "node:path";

const ROOT = process.cwd();
const SOURCE_EXTENSIONS = new Set([".ts", ".tsx", ".js", ".jsx", ".mjs"]);
const IGNORED = new Set([
  "node_modules",
  ".next",
  "dist",
  "build",
  "coverage",
  "generated",
  "__generated__",
]);

const isGenerated = (path) =>
  path.includes("generated") || /\.generated\.[cm]?[jt]sx?$/.test(path);

const walk = (directory) => {
  const files = [];
  for (const entry of readdirSync(directory)) {
    if (IGNORED.has(entry)) {
      continue;
    }
    const path = join(directory, entry);
    if (statSync(path).isDirectory()) {
      files.push(...walk(path));
      continue;
    }
    if (SOURCE_EXTENSIONS.has(extname(path)) && !isGenerated(path)) {
      files.push(path);
    }
  }
  return files;
};

const files = walk(ROOT);

const declaredTypes = new Map();
const usedIn = new Map();

for (const file of files) {
  const source = readFileSync(file, "utf8");
  const relativePath = relative(ROOT, file);

  const declarationPattern =
    /^\s*export\s+(?:declare\s+)?(interface|type|enum|class)\s+([A-Za-z_$][\w$]*)/gm;
  for (const match of source.matchAll(declarationPattern)) {
    declaredTypes.set(match[2], relativePath);
  }

  const identifierPattern = /\b([A-Za-z_$][\w$]*)\b/g;
  for (const match of source.matchAll(identifierPattern)) {
    const name = match[1];
    if (!usedIn.has(name)) {
      usedIn.set(name, new Set());
    }
    usedIn.get(name).add(relativePath);
  }
}

const byFile = new Map();

for (const [typeName, declaringFile] of declaredTypes) {
  if (!byFile.has(declaringFile)) {
    byFile.set(declaringFile, []);
  }
  const consumers = new Set(usedIn.get(typeName) ?? []);
  consumers.delete(declaringFile);
  byFile.get(declaringFile).push({ typeName, consumers });
}

let violations = 0;

const shareAPlace = (left, right) => {
  if (left.consumers.size === 0 && right.consumers.size === 0) {
    return true;
  }
  if (left.consumers.size === 0 || right.consumers.size === 0) {
    return false;
  }
  return [...left.consumers].some((consumer) => right.consumers.has(consumer));
};

for (const [file, members] of byFile) {
  if (members.length < 2) {
    continue;
  }
  for (let i = 0; i < members.length; i += 1) {
    for (let j = i + 1; j < members.length; j += 1) {
      const left = members[i];
      const right = members[j];
      if (!shareAPlace(left, right)) {
        violations += 1;
        console.error(
          `${file}: "${left.typeName}" and "${right.typeName}" are used in different places and must not share one contract file.`,
        );
      }
    }
  }
}

if (violations > 0) {
  console.error(`${violations} contract grouping violation(s) found.`);
  process.exit(1);
}

console.log("Contract consumer grouping OK.");
```

Add it to the project scripts and run it from the lint pipeline:

```json
{
  "scripts": {
    "lint": "eslint .",
    "check:contracts": "node scripts/check-contract-consumers.mjs",
    "check:structure": "npm run lint && npm run check:contracts"
  }
}
```

## Rule applicability by ecosystem

Two rules are involved, and they do not apply to the same ecosystems.

**Rule A — one type per file, separated from runtime behavior.** Language
agnostic. It applies to every supported ecosystem, and it is *installed* in
the ones whose linter can express it.

**Rule B — prefer arrow functions.** A TypeScript and JavaScript
convention. It has no counterpart in Python, Java, Kotlin, Go, Rust, C#, PHP,
or Ruby, because those languages have no arrow syntax and their own
conventions govern function declarations. The system never imposes an alien
convention on an ecosystem that has its own. Where an ecosystem does define
a function-style convention, the project convention prevails.

### Coverage matrix

| Ecosystem | Rule A installed | Rule A verified | Rule B |
|---|---|---|---|
| TypeScript / JavaScript | Yes — ESLint `structural/no-mixed-types-and-runtime` + `func-style` | Yes | Yes |
| Java | Yes — Checkstyle `OneTopLevelClass` | Yes | Not applicable |
| C# / .NET | Yes — StyleCop `SA1402` | Yes | Not applicable |
| PHP | Yes — PHP_CodeSniffer `PSR1.Classes.ClassDeclaration` | Yes | Not applicable |
| Python | No — Ruff cannot express it | Yes | Not applicable |
| Go | No — `golangci-lint` has no equivalent | Yes | Not applicable |
| Rust | No — `clippy` has no equivalent | Yes | Not applicable |
| Kotlin | No — multiple declarations per file are idiomatic | Yes | Not applicable |
| Ruby | No — Sorbet/RBS carry the types | Yes | Not applicable |

"Verified" means the rule is checked by the deterministic compliance check in
`workflows/output-compliance.md`, which runs over the change set and reports
`file:line` evidence. It is enforced by the agent and the audit, not by the
project's linter, because the ecosystem offers no rule to install.

### Java — Checkstyle

`OneTopLevelClass` enforces one top-level type per file, so an interface and
a class in the same file are a violation.

```xml
<?xml version="1.0"?>
<!-- config/checkstyle/checkstyle.xml -->
<!DOCTYPE module PUBLIC
  "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
  "https://checkstyle.org/dtds/configuration_1_3.dtd">
<module name="Checker">
  <module name="TreeWalker">
    <module name="OneTopLevelClass">
      <property name="severity" value="error"/>
    </module>
    <module name="OuterTypeFilename">
      <property name="severity" value="error"/>
    </module>
  </module>
</module>
```

### C# / .NET — StyleCop analyzers

`SA1402` (FileMayOnlyContainOneType) and `SA1401` (fields must be private)
enforce the type-per-file rule.

```ini
; .editorconfig
dotnet_diagnostic.SA1402.severity = error
dotnet_diagnostic.SA1401.severity = error
dotnet_diagnostic.SA1201.severity = error
```

`SA1201` requires elements to appear in the correct order, which keeps
types before members and supports the separation of contract declarations
from behavior.

### PHP — PHP_CodeSniffer with the PSR-12 standard

`PSR1.Classes.ClassDeclaration` enforces one class per file, and PSR-4
autoloading already depends on it.

```xml
<?xml version="1.0"?>
<!-- phpcs.xml -->
<ruleset name="Project">
  <rule ref="PSR12"/>
  <rule ref="PSR1.Classes.ClassDeclaration">
    <severity>5</severity>
  </rule>
</ruleset>
```

### Python, Go, Rust, Kotlin, Ruby

No linter in these ecosystems can express the rule, so the deterministic
compliance check is the mechanism. Do not invent a custom linter plugin for
them, and do not report the rule as unenforced without stating that the
compliance check covers it.

## Installation requirements

- No new runtime dependency. `func-style`, `no-restricted-syntax`, and the
  local rule use ESLint core.
- `eslint-plugin-react` is used only when it is already installed.
- The local rule file and the script are plain project files under the
  existing source tree.
- Checkstyle, StyleCop analyzers, PHP_CodeSniffer, Ruff, and
  `golangci-lint` are the linters already mandated by the formatting and
  linting policy in `AGENTS.md`. They are added as project-local development
  dependencies, never globally.
- When the ecosystem linter cannot express a rule, the compliance check
  covers it and the report states that fact explicitly.
