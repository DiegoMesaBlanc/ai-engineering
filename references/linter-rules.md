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

| Rule | Enforces |
|---|---|
| `func-style: ["error", "expression"]` | No `function` declarations; arrow functions preferred |
| `no-restricted-syntax` (`VariableDeclarator > FunctionExpression`) | No `function` expression assigned to a variable |
| `structural/no-mixed-types-and-runtime` (local rule) | A file declares types **or** runtime behavior, never both |
| `structural/one-contract-file-per-consumer` (local script) | Contract files group types by consumer |
| `react/no-multi-comp` | One component per file, only when `eslint-plugin-react` is already installed |

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

## Other ecosystems

| Ecosystem | Enforceable now | Mechanism |
|---|---|---|
| Python | Partial | Ruff `D` rules for docstrings; type/runtime separation is not expressible in Ruff — verify with the compliance check |
| Java | Partial | Checkstyle `OneTopLevelClass` enforces one public type per file; the contract/runtime split is verified by the compliance check |
| Kotlin | No | Kotlin allows multiple declarations per file by design |
| Go | Partial | `golint`/`revive` `type-name`; the interface location rule is verified by the compliance check |
| Rust | Partial | `clippy::module_inception`, `clippy::self_named_module_files`; the contract split is verified by the compliance check |
| C#/.NET | Partial | Roslyn analyzers `SA1402` (one type per file) |
| PHP | Partial | PSR-4 already enforces one class per file |
| Ruby | No | Sorbet/RBS carry the types; no lint rule separates them |

Where the ecosystem linter cannot express the rule, the structural
compliance check in `workflows/output-compliance.md` remains the
verification mechanism.

## Installation requirements

- No new runtime dependency. `func-style`, `no-restricted-syntax`, and the
  local rule use ESLint core.
- `eslint-plugin-react` is used only when it is already installed.
- The local rule file and the script are plain project files under the
  existing source tree.
