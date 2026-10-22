# Output Compliance Workflow

## Purpose

Mechanically verify that generated code complies with the structural rules of
`skills/implementation-recipes/SKILL.md` and `skills/architecture-advisor/SKILL.md`.

This workflow exists because prose rules do not prevent violations. It runs
deterministic checks over the change set and returns `PASS` / `FAIL` with
`file:line` evidence. It is read-only and never modifies code.

## Scope

- The change set under validation: staged, unstaged, and untracked source
  files of the target project.
- Files added, modified, or created by the current task.
- Excludes generated output (`*.generated.*`, `generated/`, `dist/`,
  `build/`, `coverage/`, `node_modules/`, lockfiles, minified assets).

## Canonical Rules Checked

### FUNC-001 — Prefer arrow functions (TypeScript/JavaScript)

A source file must not declare a function with the `function` keyword when
an arrow function is equivalent. `function` is allowed only with a
legitimate reason: generators (`function*`), recursion that needs
self-reference by name, a required `this` binding that arrow syntax cannot
provide, use of `arguments`, or use of `new.target`.

**Detection:**

```sh
grep -rnE '(^|[[:space:]])(export[[:space:]]+)?(default[[:space:]]+)?(async[[:space:]]+)?function[[:space:]]+[A-Za-z_$]' \
  <changed-paths> --include='*.ts' --include='*.tsx' --include='*.js' \
  --include='*.jsx' --include='*.mts' --include='*.cts' \
  --exclude-dir=node_modules --exclude-dir=dist --exclude-dir=build
```

The pattern does not match `function*`, so generators are not flagged.
Each match is then reviewed for the allowed reasons. A match without a
legitimate reason is `FAIL` (P3). A match with a legitimate reason is
`PASS` with the reason recorded.

### TYPE-001 — No file mixes type declarations and runtime behavior

A source file declares either **type declarations** or **runtime behavior**,
never both. Types live in a dedicated contract file
(`types.ts`, `contracts.ts`, `schemas.ts`, or the ecosystem equivalent).
Type-only re-export barrels and `import type` statements are not runtime
behavior.

**Type declaration patterns per ecosystem:**

| Ecosystem | Pattern |
|---|---|
| TypeScript/JavaScript | `interface`, `type`, `enum`, `declare` |
| Python | `class` deriving from `Protocol`, `TypedDict`, `BaseModel`, `dataclass` |
| Java/Kotlin | `interface`, `record`, `enum`, `data class` |
| Go | `type ... interface`, `type ... struct` |
| Rust | `trait`, `enum` |
| C#/.NET | `interface`, `record`, `struct` |
| PHP | `interface`, `enum` |
| Ruby | Sorbet `sig` blocks, RBS declarations |

**Runtime behavior patterns per ecosystem:**

| Ecosystem | Pattern |
|---|---|
| TypeScript/JavaScript | `function`, `=>`, `class` with a body, `const`/`let` with a value |
| Python | `def`, `class` with behavior |
| Java/Kotlin | method declarations, class bodies with behavior |
| Go | `func` |
| Rust | `fn` |
| C#/.NET | methods |
| PHP | functions, class bodies with behavior |
| Ruby | `def` |

**Contract file contents.** A contract file may contain:

- type declarations: `interface`, `type`, `enum`, `declare`;
- schema declarations that pair with an inferred type: `z.object(...)`,
  `Joi.object(...)`, `yup.object(...)` and their `infer` counterparts;
- literal-union constants: `export const X = [...] as const` when the constant
  backs a type.

A contract file must not contain:

- `function` declarations;
- `class` declarations with a body, including `class X extends Error`;
- runtime constants that are catalogs or configuration rather than contracts
  (i18n catalogs, configuration maps, lookup tables);
- service or transport logic.

**Detection (TypeScript/JavaScript):**

```sh
# Files that declare types
grep -rlE '(^|[[:space:]])(export[[:space:]]+)?(declare[[:space:]]+)?(interface|type|enum)[[:space:]]+[A-Za-z_$]' \
  <changed-paths> --include='*.ts' --include='*.tsx' \
  --exclude-dir=node_modules --exclude-dir=dist --exclude-dir=build

# For each candidate file, check whether it also contains runtime behavior
grep -nE '(^|[[:space:]])(export[[:space:]]+)?(default[[:space:]]+)?(async[[:space:]]+)?function[[:space:]]+[A-Za-z_$]|(^|[[:space:]])(export[[:space:]]+)?(const|let|var)[[:space:]]+[A-Za-z_$][A-Za-z0-9_$]*[[:space:]]*=|(^|[[:space:]])(export[[:space:]]+)?(abstract[[:space:]]+)?class[[:space:]]+[A-Za-z_$]' \
  <candidate-file> \
  | grep -vE 'as[[:space:]]+const|z\.(object|string|number|boolean|array|enum|union|discriminatedUnion|literal|record)\(|Joi\.object|yup\.object'
```

The runtime pattern matches top-level declarations, not callbacks nested
inside a function body. The second filter removes schema declarations and
literal-union constants, which are allowed in a contract file. A file that
still matches is `FAIL` (P2): the offending declarations must be extracted to
the file that owns them, and the types must live in a dedicated contract file
in the owning layer's folder.

### TYPE-002 — One contract file per consumer group

A contract file may contain several type declarations **only** when all of
them are consumed by the same component, service, or logical unit. Types
with different consumers are separated by ownership, following the layer or
module that owns each contract.

**Detection:**

1. For each type declared in a contract file, list the files that import it:

   ```sh
   grep -rn "import[[:space:]].*\b<TypeName>\b" <project-root>/src \
     --include='*.ts' --include='*.tsx'
   ```

2. Compare the consumer sets. Types with identical consumer sets may share a
   file. Types with different consumer sets must be separated.
3. A type with no external consumer that lives in a file with runtime
   behavior is already covered by `TYPE-001`.

### TYPE-003 — No pass-through re-export of a consumed contract

A module that consumes a contract does not re-export it. The owning file is
the single public path of the contract. A second path — a component that
re-exports the props type it imports from `./types`, a service that
re-exports the contracts it imports from `./contracts` — is ambiguity, not
convenience, and it is removed.

A barrel that aggregates several modules (`index.ts` re-exporting from more
than one source) is a public API surface and is exempt.

**Detection per ecosystem:**

| Ecosystem | Pass-through form | Pattern |
|---|---|---|
| TypeScript / JavaScript | `export … from "./x"` | `^export (type )?\{[^}]+\} from "\./` |
| Go | type alias | `^type [A-Za-z_][A-Za-z0-9_]* = [A-Za-z_][A-Za-z0-9_]*\.[A-Za-z_]` |
| Rust | `pub use` of a sibling | `^pub use (super\|crate\|self)::` |
| Python | sibling import forwarded through `__all__` | `^from \.[A-Za-z0-9_.]+ import` together with `__all__` containing the name |
| Ruby | constant alias of an imported constant | `^[A-Z][A-Za-z0-9_]* = [A-Z][A-Za-z0-9_:]*` |
| Kotlin | `typealias` | `^typealias [A-Za-z_][A-Za-z0-9_]* = ` |
| PHP | `class_alias()` | `class_alias\(\s*['"]` |
| C# / .NET | `TypeForwardedTo` | `\[assembly:\s*TypeForwardedTo` |
| Java | none — the language has no forwarding mechanism | Not applicable |

**Detection (TypeScript/JavaScript):**

```sh
# Re-export candidates: any module specifier, relative or aliased
grep -rnE "^export (type )?\{[^}]+\} from \"[^\"]+\"" <changed-paths> \
  --include='*.ts' --include='*.tsx' \
  --exclude-dir=node_modules --exclude-dir=dist --exclude-dir=build
```

This is a **candidate generator**, not a verdict. For each match, classify
it:

- The module **also imports** from the same source → pass-through.
- The source is a **contract file** (`types`, `contracts`, `schemas`,
  `interfaces`) → pass-through.
- The file re-exports from **two or more** distinct sources → it is an
  aggregating barrel, which is exempt.

The module specifier may be relative (`./types`) or aliased
(`@/contracts/types`); both are covered.

**Detection (other ecosystems):**

```sh
# Go: type alias
grep -rnE "^type [A-Za-z_][A-Za-z0-9_]* = [A-Za-z_][A-Za-z0-9_]*\.[A-Za-z_]" \
  <changed-paths> --include='*.go'

# Rust: pub use forwarding a sibling module
grep -rnE "^pub use (super|crate|self)::" <changed-paths> --include='*.rs'

# Python: sibling import forwarded through __all__
grep -rnE "^from \.[A-Za-z0-9_.]+ import" <changed-paths> --include='*.py'

# Kotlin: type alias
grep -rnE "^typealias [A-Za-z_][A-Za-z0-9_]* = " <changed-paths> --include='*.kt'

# PHP: class alias
grep -rnE "class_alias\(\s*['\"]" <changed-paths> --include='*.php'

# C#/.NET: assembly type forwarding
grep -rnE "\[assembly:\s*TypeForwardedTo" <changed-paths> --include='*.cs'

# Ruby: constant alias
grep -rnE "^[A-Z][A-Za-z0-9_]* = [A-Z][A-Za-z0-9_:]*" <changed-paths> --include='*.rb'
```

For Python, Ruby, and Kotlin, confirm the forwarded name is only
re-exposed and not part of that module's own API surface. A forwarding
mechanism used with a documented purpose — an assembly migration in C#/.NET,
a conditional class alias in PHP — is not a pass-through.

### STRUCT-001 — One folder per unit inside a feature

A feature folder never holds a flat pile of implementation files, tests, and
types mixed together. A unit that owns an implementation and a test gets its
own folder, with the test next to the implementation.

```sh
# Implementation and test as siblings inside a feature folder
for feature in $(find src/features -mindepth 1 -maxdepth 1 -type d 2>/dev/null); do
  find "$feature" -maxdepth 1 -type f \
    \( -name '*.test.ts' -o -name '*.test.tsx' -o -name '*.spec.ts' \) |
  while read -r test; do
    base="$(basename "$test")"
    impl="${base%%.test*}"
    [ -f "$feature/$impl.ts" ] || [ -f "$feature/$impl.tsx" ] ||
      [ -f "$feature/$impl.js" ] || [ -f "$feature/$impl.jsx" ] &&
      echo "FAIL $test sits next to its implementation: the unit needs its own folder"
  done
done
```

### STRUCT-002 — No imports that reach past a feature's public API

A feature may import from `app/` and from `shared/`. It must never import
the internals of another feature. Cross-feature interaction goes through the
other feature's public API, or through the global state in `app/`.

```sh
# An import that reaches into another feature's internals
grep -rnE "from ['\"][^'\"]*features/[^'\"/]+/" src/ \
  --include='*.ts' --include='*.tsx' 2>/dev/null |
  grep -vE "features/[^/]+/?['\"]" |
  sed 's/^/  /'
```

An import of `features/<other>/` is the public API and is allowed. An import
of `features/<other>/<internals>` is a `FAIL`.

### STRUCT-003 — Every feature exposes a public API

In a feature-driven architecture, `features/<feature>/index.ts` is the only
entry point other parts of the application may use.

```sh
for feature in $(find src/features -mindepth 1 -maxdepth 1 -type d 2>/dev/null); do
  [ -f "$feature/index.ts" ] ||
    echo "FAIL $feature has no public API (index.ts)"
done
```

### STRUCT-004 — The created tree matches the chosen architecture

The folders that exist must be the folders that were decided. Compare the
top-level folders of the project against the tree of the agreed style, in
`references/folder-architectures.md`, translated to the detected ecosystem.

```sh
# The agreed top-level folders, for example for a feature-driven web app
for folder in app shared features; do
  [ -d "src/$folder" ] ||
    echo "FAIL src/$folder is missing: it belongs to the agreed tree"
done
```

A missing or an unexpected top-level folder is a `FAIL` (P2). The check
does not judge whether the tree is a good one; it verifies that the tree is
the one that was agreed.

### STRUCT-005 — A monorepo requires a recorded concrete benefit

A monorepo is justified by a concrete benefit: shared code or types,
atomic changes across projects, common tooling, or consistent validation.
One team and a shared release train are not that benefit.

When the repository is a monorepo, the decision that created it must name
the benefit. A repository with several packages and no such record is a
`FAIL` (P2): the topology was adopted by default.

```sh
# A monorepo with more than one package and no recorded benefit
packages=$(find . -maxdepth 3 -name package.json -not -path "*/node_modules/*" | wc -l)
if [ "$packages" -gt 1 ]; then
  grep -rqiE "shar(e|es|ed|ing)[^.]*(code|type|librar)|atomic|atomically|common tooling|consistent validation|single commit" \
    --include='*.md' . --exclude-dir=node_modules ||
    echo "FAIL monorepo without a recorded concrete benefit"
fi
```

### CHECK-001 — Phase checklist format and derivation

A phase checklist is valid when every line uses the state format and every
step names the artifact it produces. This check is repeatable and does not
depend on the language or the framework.

```sh
# The format: [ x ], [   ] or [ ! ] followed by the step
checklist() {
  printf '%s\n' "$@" | grep -nvE '^\[( x|   | !)\] .+' |
    sed 's/^/  FAIL invalid checklist line: /'
}

# The derivation: a step is not the name of a phase
checklist_not_phase_names() {
  printf '%s\n' "$@" |
    grep -iE '^\[( x|   | !)\] (requirements and approach|implementation milestone|new finding|validation results|final diff|alignment|validation|review|implementation|planning|design|analysis|delivery|documentation)\b' |
    sed 's/^/  FAIL step is a phase name: /'
}
```

A compliant checklist passes both checks. A checklist of phase names, such
as "Implementation milestone", fails the second one. A step states the
concrete thing it creates or verifies — "the store reducers", "the MSAL
sign-in flow", "the cart view" — and its artifact comes from the plan the
step was derived from, so the step text does not repeat the path.

The scenarios the derivation must cover are listed in
`workflows/session-resilience.md` ("Scenarios the checklist must cover").

### GATE-001 — No installation path bypasses the approval gate

Packages are installed as an **installation package**, evaluated once. An
instruction that installs a package without the package evaluation is a
`FAIL`, whatever step it belongs to.

```sh
# Instrucciones que instalan paquetes sin pasar por la puerta
grep -rnE '\b(npm|pnpm|yarn|bun|pip|pip3|poetry|uv|cargo|gem|composer|dotnet|mvn|gradle)\s+(install|add|i|get)\s+[@<a-zA-Z0-9_./-]+' \
  --include='*.md' AGENTS.md workflows/ commands/ skills/ references/ 2>/dev/null |
  grep -viE 'approval|approve|authorized|gate|installation package|installation list|package|never|without|only after|npx ' |
  sed 's/^/  /'
```

A line that installs a package and does not reference the approval gate is a
finding. The expected result is no output.

### FILE-001 — Unit folder and test co-location

Where the ecosystem supports it, a unit with multiple related artifacts
(implementation, test, styles, stories, subcomponents) lives in a dedicated
unit folder with the test next to the unit. `Component.tsx` +
`Component.test.tsx` + `Component/` as siblings is `FAIL` (P2).

## Procedure

1. Resolve the target project root and the change set (staged, unstaged,
   untracked).
2. Exclude generated output, dependencies, and build artifacts.
3. Run `FUNC-001` over the change set.
3. Run `FUNC-001` over the change set.
4. Run `TYPE-001` over the change set.
5. Run `TYPE-002` over the contract files identified in `TYPE-001`.
6. Run `TYPE-003` over the change set, in the form the detected ecosystem
   uses to forward a contract.
7. Run `FILE-001` over the change set.
8. Review each match against the allowed reasons before reporting it.
9. Report every finding with `file:line`, the rule id, the severity, and the
   concrete fix.

## Reporting

```
### OUTPUT COMPLIANCE

- Scope: <paths, branch, commit>
- Checks executed: FUNC-001, TYPE-001, TYPE-002, TYPE-003, FILE-001
- Result: PASS | FAIL
- Findings:
  - <file>:<line> — <RULE> — <severity> — <reason> — <fix>
- Skipped checks: <check> — <reason>
```

Report `FAIL` when any check fails. Do not report `PASS` for a check that
was not executed, and do not silence a finding because the surrounding task
is complete. A finding inside the declared scope is the task's own defect,
not pre-existing noise.

## Integration

- `workflows/task-to-code.md` Phase 10 runs this workflow before the task is
  considered implemented.
- `workflows/project-validation.md` includes it as a mandatory check for
  software projects with source code.
- `workflows/code-audit.md` uses it as deterministic evidence for
  `FILE_COHESION`, `TYPE_OWNERSHIP`, and `TS_FUNCTION_STYLE` findings.
