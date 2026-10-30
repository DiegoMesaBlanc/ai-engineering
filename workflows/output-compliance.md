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

# Local re-export of a name imported in the same file (no `from` clause)
grep -rnE "^export (type )?\{[^}]+\};?[[:space:]]*$" <changed-paths> \
  --include='*.ts' --include='*.tsx' \
  --exclude-dir=node_modules --exclude-dir=dist --exclude-dir=build |
while IFS=: read -r file line rest; do
  names="$(printf '%s' "$rest" |
    sed -E 's/^export (type )?\{//; s/\};?[[:space:]]*$//' |
    tr ',' '\n' | sed 's/^ *//; s/ *$//')"
  printf '%s\n' "$names" | while read -r name; do
    [ -n "$name" ] || continue
    grep -qE "^import[[:space:]].*\b$name\b" "$file" &&
      echo "  FAIL $file:$line — TYPE-003 — local pass-through of $name"
  done
done
```

This is a **candidate generator**, not a verdict. For each match, classify
it:

- The module **also imports** from the same source, or imports the name and
  then re-exports it locally → pass-through.
- The source is a **contract file** (`types`, `contracts`, `schemas`,
  `interfaces`) → pass-through.
- The file re-exports from **two or more** distinct sources → it is an
  aggregating barrel, which is exempt. A local re-export of an imported name
  is not a barrel: it adds no new module path, it only forwards.

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

### STRUCT-001 — One folder per unit inside its responsibility folder

A responsibility folder never holds a flat pile of implementation files and
tests mixed together. A unit that owns an implementation and a test gets its
own folder inside `components/`, `hooks/`, or `services/`, with the test next
to the implementation. The check looks at the responsibility level only
(depth 3 under `features/`); a test inside a unit folder (depth 4) is the
expected layout and is not flagged.

```sh
# Implementation and test as siblings directly inside a responsibility folder
# depth 3 under features/ = features/<feature>/<responsibility>/<file>
for root in $(find . -type d -iname features -not -path '*/node_modules/*' 2>/dev/null); do
  find "$root" -mindepth 3 -maxdepth 3 -type f \
    \( -name '*.test.ts' -o -name '*.test.tsx' -o -name '*.test.js' \
       -o -name '*.test.jsx' -o -name '*.spec.ts' \) |
  while read -r test; do
    dir="$(dirname "$test")"
    base="$(basename "$test")"
    unit="${base%%.test*}"
    unit="${unit%%.spec*}"
    for ext in ts tsx js jsx; do
      impl="$dir/$unit.$ext"
      if [ -f "$impl" ]; then
        echo "FAIL $test:1 sits next to $impl: the unit needs its own folder"
      fi
    done
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

In a feature-driven architecture the feature exposes one public entry point.
The form is the ecosystem's: `index.ts` for TypeScript and JavaScript,
`__init__.py` for Python, `mod.rs` for Rust, `package-info.java` for Java, a
`*Module.cs`/`*Module.php` for C#/.NET and PHP, and the feature's own module
file for Ruby. Test mirrors (`src/test/`, `tests/`, `spec/`) are not feature
roots and are skipped.

```sh
for root in $(find . -type d -iname features -not -path '*/node_modules/*' 2>/dev/null); do
  case "$root" in
    */src/test/*|*/tests/*|*/spec/*) continue ;;
  esac
  for feature in $(find "$root" -mindepth 1 -maxdepth 1 -type d 2>/dev/null); do
    name="$(basename "$feature")"
    found=0
    for api in index.ts index.tsx index.js index.jsx index.vue index.svelte \
               __init__.py mod.rs package-info.java doc.go "$name.rb"; do
      if [ -f "$feature/$api" ]; then found=1; break; fi
    done
    for module in "$feature"/*Module.cs "$feature"/*Module.php; do
      if [ -f "$module" ]; then found=1; break; fi
    done
    if [ "$found" -ne 1 ]; then
      echo "FAIL $feature:1 has no public API for its ecosystem"
    fi
  done
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

### STRUCT-006 — The feature root holds only responsibility folders and its public API

A feature is not a folder for "the feature's files"; it is the boundary that
contains the folders. A source file directly inside `features/<feature>/` is
a `FAIL` (P2): it belongs to `components/`, `hooks/`, `services/`, `types/`,
`context/`, or the ecosystem equivalent. The public API of the feature is the
only allowed file at the root, plus the ecosystem's package or module
marker (`__init__.py`, `mod.rs`, `package-info.java`, `doc.go`, a
`*Module.cs`/`*Module.php`, or the feature's own Ruby module file).

```sh
# Files directly inside a feature folder, excluding the public API and markers
for root in $(find . -type d -iname features -not -path '*/node_modules/*' 2>/dev/null); do
  case "$root" in
    */src/test/*|*/tests/*|*/spec/*) continue ;;
  esac
  find "$root" -mindepth 2 -maxdepth 2 -type f \
    \( -name '*.ts' -o -name '*.tsx' -o -name '*.js' -o -name '*.jsx' \
       -o -name '*.vue' -o -name '*.svelte' -o -name '*.java' -o -name '*.kt' \
       -o -name '*.py' -o -name '*.go' -o -name '*.rs' -o -name '*.cs' \
       -o -name '*.php' -o -name '*.rb' \) |
  while read -r file; do
    base="$(basename "$file")"
    feature="$(basename "$(dirname "$file")")"
    case "$base" in
      index.*|__init__.py|mod.rs|package-info.java|doc.go|*Module.cs|*Module.php)
        continue ;;
    esac
    if [ "$base" = "$feature.rb" ]; then
      continue
    fi
    echo "FAIL $file:1 sits at the feature root: it belongs to a responsibility folder"
  done
done
```

### STRUCT-007 — Contracts are not mixed with components

A contract file lives in `types/` (or the ecosystem's contract folder,
`domain/` or `contracts/`), or in the folder of the single unit that consumes
it. It never sits at the feature root and never directly inside
`components/`, `hooks/`, or `services/` next to the units. A component and
the types it does not own are not piled in one folder.

```sh
# A contract at the feature root, or directly inside a responsibility folder
for root in $(find . -type d -iname features -not -path '*/node_modules/*' 2>/dev/null); do
  case "$root" in
    */src/test/*|*/tests/*|*/spec/*) continue ;;
  esac
  find "$root" -mindepth 2 -maxdepth 2 -type f \
    \( -name '*.types.*' -o -name '*.contracts.*' \) |
  while read -r file; do
    echo "FAIL $file:1 sits at the feature root: a contract belongs to types/ or to its unit folder"
  done
  find "$root" -mindepth 3 -maxdepth 3 -type f \
    \( -name '*.types.*' -o -name '*.contracts.*' \) |
  while read -r file; do
    parent="$(basename "$(dirname "$file")")"
    case "$parent" in
      types|contracts|domain|models) continue ;;
    esac
    echo "FAIL $file:1 is mixed with the units of its responsibility folder: move it to types/ or to the unit folder"
  done
done
```

A contract inside a unit folder (`components/<unit>/<unit>.types.ts`) is the
single-consumer case and is allowed. A contract inside `types/`, `domain/`,
or `contracts/` is the shared case and is allowed.

### CONFIG-001 — The target project carries configuration, not system tooling

The structural policy is verified here, not installed in the target project.
The project receives its standard `eslint.config.*`, its Prettier
configuration, and the ignore files those tools need. A file that implements a
rule of this system — a local ESLint plugin under `eslint-rules/` or
`.eslint-local/`, or a checker script such as
`scripts/check-contract-consumers.mjs` — is a `FAIL` (P2): it is system
tooling the project must not carry, and **no custom rule file**, plugin, or
checker script belongs there.

```sh
{
  git diff --name-only HEAD
  git ls-files --others --exclude-standard
} | sort -u |
  grep -E '(^|/)(eslint-rules/|\.eslint-local/)|check-contract-consumers\.mjs$' |
  sed 's/^/  FAIL system tooling inside the target project: /'
```

An ecosystem's standard rule configured in the project's normal configuration
(for example Checkstyle `OneTopLevelClass`) is configuration and is not
flagged. A shared, versioned configuration package published outside the
project is also not flagged; what is flagged is a copy of system tooling
inside the project.

### LAYER-001 — Dependency direction

Dependencies point inward: `domain` imports nothing from `application`,
`infrastructure`, `api`, `ui`, or a framework; `application` may import
`domain`; `infrastructure` implements abstractions declared by the inner
layer; UI calls the application or API layer, never infrastructure. A
framework annotation in a domain model is a documented exception, not a
default.

| From | Forbidden |
|---|---|
| `domain/` | `application/`, `infrastructure/`, `api/`, `components/`, framework SDK, vendor SDK |
| `application/` | `infrastructure/`, `components/` |
| `components/` or `api/` | `infrastructure/` |
| `infrastructure/` | nothing forbidden; it may depend on `domain/` and `application/` |

```sh
# TypeScript / JavaScript
find src -type d \( -name domain -o -name application \) -print0 2>/dev/null |
xargs -0 grep -rnE "from ['\"][^'\"]*(infrastructure|/db|/storage|/stripe|@prisma|express|fastify)" 2>/dev/null |
  sed 's/^/  FAIL LAYER-001: /'
find src -type d \( -name components -o -name api \) -print0 2>/dev/null |
xargs -0 grep -rnE "from ['\"][^'\"]*infrastructure" 2>/dev/null |
  sed 's/^/  FAIL LAYER-001: /'

# Java / Kotlin
find . -type d -name domain -print0 2>/dev/null |
xargs -0 grep -rnE '^import .*(infrastructure|javax\.persistence|org\.springframework)' 2>/dev/null |
  sed 's/^/  FAIL LAYER-001: /'

# Python
find . -type d -name domain -print0 2>/dev/null |
xargs -0 grep -rnE '^(from|import) .*(infrastructure|django|fastapi|sqlalchemy)' 2>/dev/null |
  sed 's/^/  FAIL LAYER-001: /'

# C# / .NET
find . -type d \( -name Domain -o -name domain \) -print0 2>/dev/null |
xargs -0 grep -rnE '^using .*(Infrastructure|Microsoft\.EntityFrameworkCore)' 2>/dev/null |
  sed 's/^/  FAIL LAYER-001: /'

# PHP
find . -type d \( -name Domain -o -name domain \) -print0 2>/dev/null |
xargs -0 grep -rnE '^use .*(Infrastructure|Doctrine|Illuminate)' 2>/dev/null |
  sed 's/^/  FAIL LAYER-001: /'

# Ruby
find . -type d \( -name domain -o -name Domain \) -print0 2>/dev/null |
xargs -0 grep -rnE '^(require|include|extend) .*(infrastructure|active_record|rails)' 2>/dev/null |
  sed 's/^/  FAIL LAYER-001: /'
```

The patterns are candidate generators: a framework import in `domain/` is a
finding unless the project documents the exception and the review records it.
A `FAIL` is P1 when it crosses a module boundary with runtime risk, P2
otherwise. The check reports `file:line`; it never rewrites code.

### CHECK-001 — Phase checklist format and derivation

A phase checklist is valid when every line uses the state format and every
step names the artifact it produces. This check is repeatable and does not
depend on the language or the framework.

```sh
# The format: [ x ], [   ] or [ ! ] followed by the step.
# One to three spaces is accepted for the pending state, so a spacing
# difference is not reported as a defect.
checklist() {
  printf '%s\n' "$@" | grep -nvE '^\[( {1,3}| x | ! )\] .+' |
    sed 's/^/  FAIL invalid checklist line: /'
}

# The canonical phase names, read from the workflow that defines them, so the
# check never drifts from its source of truth.
phase_names() {
  for candidate in \
    "${AI_ENGINEERING_ROOT:+$AI_ENGINEERING_ROOT/workflows/task-to-code.md}" \
    "workflows/task-to-code.md" \
    "../../workflows/task-to-code.md"; do
    [ -n "$candidate" ] && [ -f "$candidate" ] || continue
    grep -hoE '^## Phase [0-9]+ .+' "$candidate" |
      sed -E 's/^## Phase [0-9]+[[:space:]]*[^[:alnum:]]*[[:space:]]*//' |
      tr '[:upper:]' '[:lower:]' | paste -sd'|' - && return 0
  done
  return 1
}

# The derivation: a step is not the name of a phase
checklist_not_phase_names() {
  names="$(phase_names)" || {
    echo "  FAIL canonical phase names not found: set AI_ENGINEERING_ROOT"
    return 1
  }
  printf '%s\n' "$@" |
    grep -iE "^\[( {1,3}| x | ! )\][[:space:]]*($names)[[:space:]]*$" |
    sed 's/^/  FAIL step is a phase name: /'
}

# A phase plan is not a phase checklist: the two artifacts are not the same
checklist_not_a_plan() {
  total=$(printf '%s\n' "$@" | grep -cE '^\[( {1,3}| x | ! )\] ')
  plans=$(printf '%s\n' "$@" | grep -cE '^\[( {1,3}| x | ! )\] [0-9]+\. ')
  if [ "$total" -gt 0 ] && [ "$total" -eq "$plans" ]; then
    echo "  FAIL this is a phase plan, not a phase checklist"
  fi
}
```

A compliant checklist passes all three checks. The pending state accepts one
to three spaces, so `[ ]` and `[   ]` are the same state and a spacing
difference is never the finding; the canonical rendering stays `[   ]`. A
checklist of phase names, such as "Implement", fails the second one, because
the names are read from `workflows/task-to-code.md` and not from a fixed
list. A phase plan passed as a checklist fails the third one, because the two
artifacts are not the same thing. A step states the
concrete thing it creates or verifies — "the store reducers", "the MSAL
sign-in flow", "the cart view" — and its artifact comes from the plan the
step was derived from, so the step text does not repeat the path.

The scenarios the derivation must cover are listed in
`workflows/session-resilience.md` ("Scenarios the checklist must cover").

### CHECK-002 — Phase plan and phase boundary gate

The phase plan lists the phases the task will go through, each with its
outcome. The boundary report states what was produced, what remains, what the
next phase adds, and the resume point; when another phase remains, it asks for
approval to start it. Both use the same states as the checklist.

```sh
# The phase plan format: [   ], [ x ] or [ ! ] followed by the phase
check_phase_plan() {
  printf '%s\n' "$@" | grep -nvE '^\[(   | x | ! )\] [0-9]+\. .+' |
    sed 's/^/  FAIL invalid phase plan line: /'
}

# The boundary report states produced, remaining, next, resume, and asks
check_boundary_report() {
  report="$(printf '%s\n' "$@")"
  printf '%s\n' "$report" | grep -qiE 'hecho|done|produced' ||
    echo "  FAIL boundary report does not say what was produced"
  printf '%s\n' "$report" | grep -qiE 'pendiente|remaining|falta' ||
    echo "  FAIL boundary report does not say what remains"
  printf '%s\n' "$report" | grep -qiE 'reanuda|resume' ||
    echo "  FAIL boundary report does not give the resume point"
  printf '%s\n' "$report" | grep -qiE 'siguiente fase|next phase' ||
    echo "  FAIL boundary report does not say what the next phase adds"

  # When another phase remains, the boundary asks for approval to start it.
  if ! printf '%s\n' "$report" |
       grep -qiE 'última fase|ultima fase|last phase|no further phase'; then
    printf '%s\n' "$report" |
      grep -qiE 'aprob|approve starting|do you approve' ||
      echo "  FAIL boundary report does not ask for approval to start the next phase"
  fi
}
```

A compliant plan passes the first check, and a compliant boundary report
passes the second one. A plan whose phases are phase names without an
outcome, or a boundary report that omits what the next phase adds or the
approval question, is a finding. A report for the last phase is exempt from
the question, because there is no transition to approve. The phases the plan
lists are the canonical ones of `workflows/task-to-code.md`, restricted to
the ones the task needs, so the sequence is derived from the task and never
fixed.

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

### FILE-001 — Unit folder, test co-location, and test-support separation

Where the ecosystem supports it, a unit with multiple related artifacts
(implementation, test, styles, stories, subcomponents) lives in a dedicated
unit folder with the test next to the unit. `Component.tsx` +
`Component.test.tsx` + `Component/` as siblings is `FAIL` (P2).

A test case never sits at the root of the source tree or of a layer folder,
and a support location never contains test cases. TypeScript and JavaScript:
`foo.ts` and `foo.test.ts` share a folder; an API-surface test that no single
unit owns is named `*.integration.test.*` or lives under `tests/integration/`;
shared support lives in `testing/`, and a `test/` or `tests/` folder directly
under `src/` is not a support location.

```sh
# (a) A test case at the root of the source tree
find src -mindepth 1 -maxdepth 1 -type f \
  \( -name '*.test.*' -o -name '*.spec.*' \) |
  sed 's/^/  FAIL test at the source root: /'

# (b) A support location that holds test cases
for dir in src/testing src/test src/tests test/support spec/support; do
  [ -d "$dir" ] || continue
  find "$dir" -type f \( -name '*.test.*' -o -name '*.spec.*' \) |
    sed "s|^|  FAIL test case inside the support location $dir: |"
done

# (c) A src/test folder that holds only support files
if [ -d src/test ] &&
   ! find src/test -type f \( -name '*.test.*' -o -name '*.spec.*' \) | grep -q .; then
  echo "  FAIL src/test holds no test case: use src/testing for support"
fi

# (d) A test inside src/ with no sibling unit and no integration mark
find src -type f \
  \( -name '*.test.ts' -o -name '*.test.tsx' \
     -o -name '*.spec.ts' -o -name '*.spec.tsx' \) |
grep -v '\.integration\.test\.' |
while read -r test_file; do
  dir="$(dirname "$test_file")"
  base="$(basename "$test_file")"
  stem="${base%%.test.*}"
  stem="${stem%%.spec.*}"
  found=0
  for ext in ts tsx js jsx mts cts; do
    [ -f "$dir/$stem.$ext" ] && found=1
  done
  [ "$found" -eq 0 ] &&
    echo "  FAIL detached test: $test_file has no sibling unit"
done
```

**Per-ecosystem test location.** A test file outside its allowed location is
a `FAIL`. The allowed locations are the ones the applicability matrix in
`skills/implementation-recipes/SKILL.md` defines:

| Ecosystem | Allowed test location | Flagged when |
|---|---|---|
| TypeScript, JavaScript, React, Next.js, Angular, Vue, Svelte, Solid, React Native, Ionic, Capacitor | Next to the unit (`*.test.*`, `*.spec.ts`), or `tests/integration/`, or `*.integration.test.*` | A test at the source root or with no sibling unit |
| Python | `test_*.py` next to the module, or the project's `tests/` tree | A `test_*.py` at the source root |
| Go | `_test.go` in the package directory | A `_test.go` whose directory has no non-test `.go` file |
| Rust | `tests/` at the crate root, or an inline `#[cfg(test)]` module | A `*_test.rs` or `*_tests.rs` file inside `src/` |
| Java, Kotlin | `src/test/java` or `src/test/kotlin`, mirrored package | A `*Test.java`, `*Tests.java`, or `*Test.kt` under `src/main` |
| C#/.NET | A test project (`tests/`, `*.Tests/`) | A `*Tests.cs` under `src/` |
| PHP | `tests/`, mirrored namespace | A `*Test.php` under `src/` |
| Ruby | `spec/`, mirroring `lib/` | A `*_spec.rb` under `lib/` |
| SQL | Not applicable — no test artifact | — |

```sh
# Python: a test module at the source root
find src -mindepth 1 -maxdepth 1 -name 'test_*.py' |
  sed 's/^/  FAIL root-level Python test: /'

# Go: a test file whose directory has no non-test Go file
find . -name '*_test.go' -not -path '*/node_modules/*' | while read -r t; do
  d="$(dirname "$t")"
  [ -z "$(find "$d" -maxdepth 1 -name '*.go' ! -name '*_test.go' | head -1)" ] &&
    echo "  FAIL detached Go test: $t"
done

# Rust: a test file inside src/
find src -type f \( -name '*_test.rs' -o -name '*_tests.rs' \) |
  sed 's/^/  FAIL Rust test inside src: /'

# Java, Kotlin: a test under the production source root
find src/main -type f \
  \( -name '*Test.java' -o -name '*Tests.java' -o -name '*Test.kt' \) |
  sed 's/^/  FAIL test under src/main: /'

# C#/.NET and PHP: a test under the production source root
find src -type f \( -name '*Tests.cs' -o -name '*Test.php' \) \
  -not -path '*/test*' |
  sed 's/^/  FAIL test under src: /'

# Ruby: a spec under lib/
find lib -type f -name '*_spec.rb' | sed 's/^/  FAIL spec under lib: /'
```

A `FAIL` is P2: move the test next to its unit or into the mirrored test
tree, give the integration test its explicit name or tree, or rename the
support location. The check never reorganizes code; it reports `file:line`
evidence.

## Procedure

1. Resolve the target project root and the change set (staged, unstaged,
   untracked).
2. Exclude generated output, dependencies, and build artifacts.
3. Run `FUNC-001` over the change set.
4. Run `TYPE-001` over the change set.
5. Run `TYPE-002` over the contract files identified in `TYPE-001`.
6. Run `TYPE-003` over the change set, in the form the detected ecosystem
   uses to forward a contract.
7. Run `FILE-001` over the change set and the source tree: unit folders, test
   placement, and support locations.
8. Run `STRUCT-001` and `STRUCT-006` over every feature folder.
9. Run `STRUCT-007` over every contract file in a feature folder.
10. Run `STRUCT-002` and `STRUCT-003` when the project is feature-driven.
11. Run `LAYER-001` over the source tree when the architecture declares layers.
12. Review each match against the allowed reasons before reporting it.
13. Report every finding with `file:line`, the rule id, the severity, and the
    concrete fix.

## Reporting

```
### OUTPUT COMPLIANCE

- Scope: <paths, branch, commit>
- Checks executed: FUNC-001, TYPE-001, TYPE-002, TYPE-003, FILE-001, STRUCT-001, STRUCT-006, STRUCT-007, LAYER-001
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
