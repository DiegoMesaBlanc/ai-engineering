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

Code the change set **creates** must not declare a function with the
`function` keyword when an arrow function is equivalent. `function` is
allowed only with a legitimate reason: generators (`function*`), recursion
that needs self-reference by name, a required `this` binding that arrow syntax
cannot provide, use of `arguments`, or use of `new.target`.

**Scope.** The check applies to the functions the change set creates. A
declaration that already exists in the base revision is **pre-existing**: it
is a finding recorded with its file and line, never a `FAIL`, and never a
reason to convert it inside this task. A wholesale reformat that makes
pre-existing lines appear as additions does not change that — the base
revision decides.

**Detection** — added lines of the change set only:

```sh
git diff -U0 --diff-filter=ACM "$BASE" -- \
    '*.ts' '*.tsx' '*.js' '*.jsx' '*.mts' '*.cts' |
  grep -E '^\+' | grep -v '^+++' | sed 's/^+//' |
  grep -nE '(^|[[:space:]])(export[[:space:]]+)?(default[[:space:]]+)?(async[[:space:]]+)?function[[:space:]]+[A-Za-z_$]'
```

The pattern does not match `function*`, so generators are not flagged.
Each added declaration is then reviewed: if an equivalent declaration exists
in `$BASE`, it is pre-existing and becomes a finding; otherwise it is
reviewed for the allowed reasons. A new declaration without a legitimate
reason is `FAIL` (P3). A declaration with a legitimate reason is `PASS` with
the reason recorded.

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

**Detection.** The function reads one contract file and lists, for each type it
declares, the production files that import it. The consumer of a type is a source
file under `src/` that imports the name, excluding the contract file itself, the
barrels (`index.*`), and the test and specification files (a test consumes its
unit, it is not a second consumer of the contract). Types with identical consumer
sets may share the file. A type whose consumer set differs from the first type's
set is a `FAIL` at the line of its declaration. A type with no consumer is not
compared; it is covered by `TYPE-001` when it lives with runtime behavior.

```sh
# $1 = project root; $2 = contract file, relative to the root.
consumer_group_check() {
  root="$1"; f="$2"
  [ -d "$root/src" ] || { echo "NOT VERIFIED TYPE-002: $f: there is no src/ folder, so the consumers were not searched"; return 0; }
  work="$(mktemp -d)" || return 1
  awk '/^export (declare )?(interface|type|enum) [A-Za-z_$]/ {
    n = $0; sub(/^export (declare )?(interface|type|enum) /, "", n); sub(/[^A-Za-z0-9_$].*$/, "", n)
    print NR "\t" n }' "$root/$f" |
  while IFS="$(printf '\t')" read -r line name; do
    consumers="$(grep -rlE "import[^;]*[^A-Za-z0-9_\$]${name}[^A-Za-z0-9_\$]" "$root/src" \
      --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' 2>/dev/null |
      grep -vE "/index\.|\.(test|spec)\.|/$(basename "$f")\$" | sort | tr '\n' ' ')"
    [ -n "$consumers" ] || continue
    printf '%s\t%s\t%s\n' "$line" "$name" "$consumers"
  done > "$work/types.tsv"
  awk -F '\t' -v file="$f" '
    NR == 1 { first = $3 }
    $3 != first { print "FAIL TYPE-002: " file ":" $1 ": " $2 " is consumed by a different set of files than the first type of this file (TYPE-002)" }
  ' "$work/types.tsv"
  rm -rf "$work"
}
```

The `FAIL` line names each type whose consumer set differs from the first type of the
file. A file whose types share one consumer set passes, and a file with no consumed
type prints nothing. The check runs over the contract files named by `TYPE-001`; its
consumer list is taken from the source tree of the project, so a project without a
`src/` folder is `NOT VERIFIED` for this rule, not a pass.

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

### TYPE-004 — A contract name is declared in one file only

A contract has one owning declaration. The same contract name declared in
more than one file of the project is the duplicated contract that
`skills/engineering-review/SKILL.md` and `references/audit-pass-criteria.md`
(Dimension 3) report: two copies drift apart and make the owner ambiguous. It
is the declaration counterpart of `TYPE-003`, which covers re-exports.

The detection covers TypeScript and JavaScript declarations: `interface`,
`type`, and `enum` (exported), and JSDoc `@typedef` in `.js`, `.mjs`, and `.cjs`
files, where contracts are declared without TypeScript syntax. Declarations in
the same file are not counted, because they are one declaration.

**Detection** (a candidate generator over the files of the change set and the
source tree, excluding dependencies and build output):

```sh
duplicate_contract_check() {
  for f in "$@"; do
    awk -v file="$f" '
      /^export (declare )?(interface|type|enum) [A-Za-z_$]/ {
        name = $0
        sub(/^export (declare )?(interface|type|enum) /, "", name)
        sub(/[^A-Za-z0-9_$].*$/, "", name)
        print name "\t" file "\t" NR
      }
      /@typedef/ {
        name = $0
        sub(/.*@typedef[[:space:]]+(\{[^}]*\}[[:space:]]+)?/, "", name)
        sub(/[^A-Za-z0-9_$].*$/, "", name)
        if (name != "") print name "\t" file "\t" NR
      }' "$f"
  done | awk -F '\t' '
    {
      name[NR] = $1; file[NR] = $2; line[NR] = $3
      key = $1 SUBSEP $2
      if (!(key in seen)) { seen[key] = 1; files[$1]++ }
    }
    END {
      for (i = 1; i <= NR; i++)
        if (files[name[i]] > 1)
          printf "FAIL TYPE-004: %s:%s: %s is also declared in another file\n", file[i], line[i], name[i]
    }'
}
```

Run it over the source files of the project, for example:
`duplicate_contract_check $(find src -type f \( -name '*.ts' -o -name '*.tsx' -o -name '*.js' -o -name '*.mjs' -o -name '*.cjs' \) -not -path '*/node_modules/*')`.

**Classification.** Every `FAIL` line is a candidate, and each one is
classified before it is reported:

- The declarations are copies of one contract: `FAIL` (P2). Keep the declaration
  in the owning file, import it from there in every consumer, and remove the
  copy. The owning file is the one in the layer or module that owns the contract.
- A JSDoc alias `@typedef {import("...").Name} Name` in a consumer is a second
  path to the owner's contract, not a new contract. It is `FAIL` and is removed:
  the consumer writes `import("...").Name` inline, without the alias, so the
  owning file stays the only declaration.
- The same name names two different contracts with different consumers (for
  example two `Props` types of unrelated components): not a defect. Record the
  reason in the report, so the exception is visible and not silently ignored.

A check that reports no candidate is `PASS` for `TYPE-004`. Never report `PASS`
for a project whose files were not listed.

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

### TEST-001 and TEST-002 — Test APIs are imported explicitly; the case is declared with `test`

A TypeScript or JavaScript test file the change set creates or modifies
imports every test API it uses explicitly from the framework module that
declares it (`vitest`, `@jest/globals` for Jest, or `node:test` for Node's test runner), so the type checker
accepts it without a global test declaration (M-47, `AGENTS.md`). Without
the import the file fails with `Cannot find name 'describe'` (TS2593). A
test case is declared with `test`, not `it`.

**Scope.** The check reads every test file the change set **creates** in full,
and every test file it **modifies** only on the lines the change set adds. A
violation that already exists in the base revision of a modified file is printed
as `FINDING`, with its `file:line`, and is never a `FAIL` (`AGENTS.md`,
pre-existing violations). A `FINDING` is recorded in the report and does not block
the task; it is not dropped silently (M-47, R-145). A test file the task does not
touch is never read.

**Detection.** The function reads each file and reports every line where a
test API is called or read as a member, `(?<![.\w$])NAME(` or `NAME.identifier`,
and every `it(` case. The name must appear in an import from `vitest`, `node:test`, or
`@jest/globals`. Prose such as `the test. Next` does not match, because a member
access has no space after the dot. When a second argument names a file of
added line numbers, only those lines are reported.

```sh
test_api_check() {
  if [ "$#" -ge 2 ]; then filter=1; added="$2"; else filter=0; added=/dev/null; fi
  findings=$(perl -e '
    local $/;
    my ($f, $filter, $added) = @ARGV;
    my %on;
    if ($filter) { open my $a, "<", $added or die "$added: $!"; my @l = do { local $/ = "\n"; <$a> }; close $a; %on = map { my $x = $_; $x =~ s/\s+$//; ($x => 1) } @l; }
    open my $h, "<", $f or die "$f: $!"; my $s = <$h>; close $h;
    my %imp;
    while ($s =~ /import\s+(?:type\s+)?\{([^}]*)\}\s*from\s*["\x27](?:vitest|\@jest\/globals|node:test)["\x27]/g) {
      for my $n (split /,/, $1) { $n =~ s/^\s+|\s+$//g; $n =~ s/\s+as\s+\S+$//; $imp{$n} = 1 if length $n; }
    }
    for my $n (qw(describe test it expect vi beforeEach afterEach beforeAll afterAll)) {
      while ($s =~ /(?<![.\w\$])\Q$n\E(?:\s*\(|\.[A-Za-z_])/g) {
        my $line = 1 + (substr($s, 0, $-[0]) =~ tr/\n//);
        my $pre = $filter && !$on{$line};
        my $kind = $pre ? "FINDING" : "FAIL";
        my $note = $pre ? " (pre-existing in the base revision: a recorded finding, not a FAIL)" : "";
        print "$kind TEST-001: $f:$line: $n is used without an explicit import$note\n" unless $imp{$n};
      }
    }
    while ($s =~ /(?<![.\w\$])it\s*\(/g) {
      my $line = 1 + (substr($s, 0, $-[0]) =~ tr/\n//);
      my $pre = $filter && !$on{$line};
      my $kind = $pre ? "FINDING" : "FAIL";
      my $note = $pre ? " (pre-existing in the base revision: a recorded finding, not a FAIL)" : "";
      print "$kind TEST-002: $f:$line: the case is declared with it, not test$note\n";
    }
  ' "$1" "$filter" "$added")
  [ -n "$findings" ] && printf '%s\n' "$findings"
  printf '%s\n' "$findings" | grep -q '^FAIL' && return 1
  return 0
}

# Added line numbers of a modified file, from the change set against the base.
added_lines() {
  git diff -U0 "$BASE" -- "$1" | awk '/^@@/ {
    split($3, r, ","); start = substr(r[1], 2); n = (r[2] == "" ? 1 : r[2]);
    for (i = 0; i < n; i++) print start + i
  }'
}

created_tests=$({
  git diff --name-only --diff-filter=A "$BASE"
  git ls-files --others --exclude-standard
} | sort -u | grep -E '\.(test|spec)\.(ts|tsx|mts|cts|js|jsx|mjs|cjs)$')
modified_tests=$(git diff --name-only --diff-filter=M "$BASE" |
  grep -E '\.(test|spec)\.(ts|tsx|mts|cts|js|jsx|mjs|cjs)$')

printf '%s\n' "$created_tests" | sed '/^$/d' |
  while IFS= read -r f; do test_api_check "$f"; done
printf '%s\n' "$modified_tests" | sed '/^$/d' |
  while IFS= read -r f; do
    added=$(mktemp); added_lines "$f" > "$added"
    test_api_check "$f" "$added"; rm -f "$added"
  done
```

The check uses `perl`, present on macOS and Linux. Where `perl` is absent the
check is `NOT VERIFIED` with that reason, never a pass. The check does not
detect an aliased import (`import { test as t }`) or a test API hidden behind
a local wrapper; such a file is reported for review.

### TEST-003 — A test reaches its state by a controlled clock or an event

M-76: a test reaches its state through a controlled clock or an event, never through a real timer hold. A test or a test-support helper that holds a resource with a real timer, or waits
on a real delay, depends on wall-clock time. Such a test passes or fails with the load
of the machine, and it cannot prove a bound because the bound is the timer itself. The
detector reads every test file and every test-support file of the change set, and of the
sweep: a file that calls `setTimeout`, `setInterval`, or `sleep` and never uses fake
timers is a `FAIL` at each such line. The same holds for a wall-clock read (`Date.now()`,
`performance.now()`) and for a real timer passed as an option (`timeoutMs: 60`), because an
elapsed-time assertion built on them depends on the load of the machine (F-1 of the external
audit, M-76). A file that controls time with fake timers, or
that releases the resource on an event, passes. There is no exception marker: a real
timer in a test is replaced by a controlled clock or an event, and the guarantee that a
bounded operation stops is asserted under that control.

```sh
# $1 = repository root; $2... = test and test-support files, relative to it.
timer_hold_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    [ -f "$root/$f" ] || continue
    grep -qE 'useFakeTimers|advanceTimersByTime|fakeTimers|FakeTimers|sinon\.useFakeTimers' "$root/$f" && continue
    grep -nE '(^|[^A-Za-z0-9_.])(setTimeout|setInterval|sleep)\(' "$root/$f" 2>/dev/null |
    while IFS= read -r hit; do
      echo "FAIL TEST-003: $f:${hit%%:*}: a real timer or delay in a test with no controlled clock (M-76)"
    done
    # M-76: a wall-clock read or a real timer passed as an option is the same dependency on time,
    # and an elapsed-time assertion built on it is a timing race even with no literal setTimeout.
    grep -nE '(Date\.now\(\)|performance\.now\(\))' "$root/$f" 2>/dev/null |
    while IFS= read -r hit; do
      echo "FAIL TEST-003: $f:${hit%%:*}: a wall-clock read in a test with no controlled clock (M-76)"
    done
    grep -nE '(timeoutMs|delayMs|sleepMs|waitMs|intervalMs)[[:space:]]*[:=][[:space:]]*[0-9]' "$root/$f" 2>/dev/null |
    while IFS= read -r hit; do
      echo "FAIL TEST-003: $f:${hit%%:*}: a real timer passed as an option to a test with no controlled clock (M-76)"
    done
  done
}
```

### DEAD-EXPORT-001 — Every export the change set adds has a consumer

A TypeScript or JavaScript file the change set creates or modifies declares
its exports only when another module uses them (M-49, `AGENTS.md`). An export
that is re-exported by a barrel (`index.*`) but imported by no other module is
dead: the barrel makes it look public, and the control it names is never
reached. An export is live only when it is imported by another module. The check reads the declared exports of the changed non-barrel files
and searches the repository for an import of each name from a production module
other than the file itself and a barrel. **Test files do not count as
consumers**: a test, a test-support helper, or a file under a test or support
folder that imports the export is not a production use. An export whose only
importer is a test is dead for production code, and its fix is to move the
helper into the explicit support location (`testing/`, `test/support/`,
`spec/support/`, `src/testFixtures`) and import it from there. The exports of a
file in a support location are exempt: they exist for tests by design, and the
check does not read them. A dead export is
a `FAIL` when the change set introduced it; an export that already existed in
the base revision is a finding.

**Scope limit.** The check finds exports with no importer. It does not decide
whether a consumer is a real control on the trust path. That is M-48, verified
by `skills/security-review/SKILL.md` ("Control wiring").

```sh
# Prints true when the module $2 imports the identifier $1 from another module.
imports_name() {
  perl -0777 -e '
    my ($n, $c) = @ARGV;
    open my $h, "<", $c or exit 1; my $s = <$h>; close $h;
    exit(($s =~ /import\s+(?:type\s+)?(?:\{[^}]*?\b\Q$n\E\b[^}]*\}|\Q$n\E\b)[^;]*?from\s*["\x27]/s) ? 0 : 1);
  ' "$1" "$2"
}

# $1 = repository root (run from it with "."); $2... = changed non-barrel files, relative to it.
dead_export_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in
      testing/*|*/testing/*|test/support/*|*/test/support/*|spec/support/*|*/spec/support/*|*/testFixtures/*|testFixtures/*) continue ;;
    esac
    grep -nE '^export (const|let|function|class|type|interface|enum) ' "$f" | while IFS=: read -r line decl; do
      name=$(printf '%s\n' "$decl" | sed -E 's/^export (const|let|function|class|type|interface|enum) ([A-Za-z_$][A-Za-z0-9_$]*).*/\2/')
      used=0
      grep -rlw --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' --include='*.mjs' --include='*.cjs' \
        --exclude-dir=node_modules --exclude-dir=dist "$name" "$root" | sed 's#^\./##' |
        while IFS= read -r c; do
          case "$c" in
            "$f"|*/index.ts|*/index.tsx|*/index.js|*/index.jsx|*/index.mjs|*/index.cjs|index.*) continue ;;
            *.test.*|*.spec.*|__tests__/*|*/__tests__/*|__mocks__/*|*/__mocks__/*|test/*|*/test/*|tests/*|*/tests/*|testing/*|*/testing/*|spec/*|*/spec/*|*/testFixtures/*|testFixtures/*) continue ;;
          esac
          imports_name "$name" "$c" && { echo used; break; }
        done | grep -q used && used=1
      [ "$used" = 1 ] || echo "FAIL DEAD-EXPORT-001: $f:$line: $name is exported with no consumer outside a barrel"
    done
  done
}
```

The check uses `perl` and `grep`, present on macOS and Linux. Where `perl` is
absent it is `NOT VERIFIED` with that reason. It does not detect a consumer that
imports the name through a dynamic `import()` or `require()`; such a file is
reported for review.

### SAFE-001, SAFE-002, SAFE-003 — Timeouts, cleartext endpoints, hardcoded endpoints and temporary paths

These detections enforce the delivery-safety rules of `AGENTS.md`:

- **SAFE-001 (M-54).** Every external call has an explicit timeout. A call to
  `fetch`, an HTTP client (`axios`, `got`, `http.request`), or an equivalent in a
  TypeScript or JavaScript file the change set creates or modifies is a `FAIL`
  when no `signal` or `timeout` option appears inside the call. The call is read
  to its closing parenthesis, up to 40 lines, so an options object of any length
  is inspected: a fixed window of a few lines misses an option that follows a long
  header block and passes an unrelated `timeout` that belongs to another statement.
  A call that is not closed within 40 lines is `NOT VERIFIED`, never clean. The
  retry loop is judged by the same test on each attempt.
- **SAFE-002 (M-55).** `http://` is rejected for every non-local build. Every
  `http://` and `https://` URL in a changed file, of any type (source, shell,
  workflow YAML, configuration), is **parsed**, not substring-matched: the
  authority is the text up to the first `/`, `?`, or `#`. A URL whose authority
  carries userinfo (`user@host`, or `localhost:1@evil.com`) is a `FAIL`, because
  the host the reader sees is not the host the request reaches. An `http://` URL is
  accepted only when its authority is exactly `localhost`, `127.0.0.1`, or `10.0.2.2`
  (the Android emulator loopback), with an optional port, or the documentation
  domains `example.com` and `example.org`; any other host is a `FAIL`. An `https://`
  URL with no host (`https://`, `https:///path`) is a `FAIL`. The scheme and host
  are checked against the parsed authority, never against a substring of the line.
  A guard that matches a URL by a glob or an alternative pattern (`http://localhost:*`,
  `http://127.0.0.1:*|...`) does not parse the authority and is a `FAIL` with that reason,
  whatever the pattern accepts (M-55, F-A).
  Limit: the detector reads one URL per match on a line and does not decode
  percent-encoding; a host written in an encoded form is `NOT VERIFIED` for the
  reviewer under Dimension 6.
- **SAFE-003 (M-58).** No hardcoded environment endpoint and no fixed shared
  temporary path. A `localhost` or `127.0.0.1` URL in non-test TypeScript or
  JavaScript source is a `FAIL`, and a fixed temporary-directory path literal in a changed script is
  a `FAIL` unless the line also reads `TMPDIR` (the `${TMPDIR:-/tmp}` fallback of
  a `mktemp` call is not a fixed path).

**Scope limit.** The detectors read TypeScript and JavaScript and POSIX shell.
Other ecosystems are `NOT VERIFIED` for SAFE-001 and SAFE-002 by these detectors
and are reviewed under `references/audit-pass-criteria.md` Dimension 6 and
Dimension 8 instead. SAFE-001 reads HTTP clients only: a database busy or lock wait,
a broker, or a child process is a bound that `BOUND-001` requires a test to hold,
and Dimension 8 reviews its declared value. A detection is a candidate: the reviewer confirms each match
before reporting it, and a call wrapped by a helper that sets the timeout is
accepted after the reviewer reads the helper.

```sh
# $1 = repository root; $2... = changed TypeScript or JavaScript files, relative to it.
safe_timeout_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    awk -v file="$f" '
      { lines[NR] = $0 }
      /(^|[^A-Za-z0-9_.])(fetch|axios|axios\.(get|post|put|patch|delete|request)|got|https?\.request)[[:space:]]*\(/ {
        call[++n] = NR
      }
      END {
        for (i = 1; i <= n; i++) {
          s = call[i]; depth = 0; ok = 0; closed = 0
          for (k = s; k <= NR && k <= s + 40 && !closed; k++) {
            text = lines[k]
            if (k == s) text = substr(text, index(text, "("))
            if (text ~ /signal|timeout|Timeout/) ok = 1
            for (c = 1; c <= length(text) && !closed; c++) {
              ch = substr(text, c, 1)
              if (ch == "(") depth++
              else if (ch == ")" && --depth == 0) closed = 1
            }
          }
          if (ok) continue
          if (closed) printf "FAIL SAFE-001: %s:%d: external call with no timeout\n", file, s
          else printf "NOT VERIFIED SAFE-001: %s:%d: the call is not closed within 40 lines, so its options are not read\n", file, s
        }
      }' "$root/$f"
  done
}

# $1 = repository root; $2... = changed files of any type (source, shell, workflow YAML,
# configuration), relative to it. Markdown is excluded: documentation is not a request.
# M-55: the authority of each URL is parsed up to the first /, ?, or #; userinfo is a
# FAIL; http:// is accepted only for an exact local or documentation host; https:// needs a host.
safe_cleartext_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.md) continue ;; esac
    grep -noE "https?://[^\"'\` ),;<>]*" "$root/$f" 2>/dev/null |
    while IFS= read -r hit; do
      n="${hit%%:*}"; url="${hit#*:}"
      scheme="${url%%://*}"; rest="${url#*://}"
      auth="${rest%%[/?#]*}"
      if [ -z "$auth" ]; then
        # A bare scheme in a comparison (startsWith("https://")) is a scheme check, not an endpoint.
        case "$(sed -n "${n}p" "$root/$f")" in *startsWith*|*endsWith*|*includes*|*indexOf*) continue ;; esac
      fi
      case "$auth" in
        *@*) echo "FAIL SAFE-002: $f:$n: a URL with userinfo names a host the reader does not see (M-55)"; continue ;;
        *\**|*\|*|*\[*) echo "FAIL SAFE-002: $f:$n: a URL is matched by a glob or alternative pattern, not parsed; the guard must parse the authority (M-55)"; continue ;;
      esac
      case "$scheme" in
        https) [ -n "$auth" ] || echo "FAIL SAFE-002: $f:$n: an https URL with no host (M-55)" ;;
        http)
          case "$auth" in
            localhost|localhost:[0-9]*|127.0.0.1|127.0.0.1:[0-9]*|10.0.2.2|10.0.2.2:[0-9]*|example.com|example.com:[0-9]*|example.org|example.org:[0-9]*) ;;
            *) echo "FAIL SAFE-002: $f:$n: cleartext http:// to a host that is not local (M-55)" ;;
          esac ;;
      esac
    done
  done
}

# $1 = repository root; $2... = changed shell or workflow YAML files, relative to it.
# M-54 for shell and workflow steps: a curl or wget statement carries a time bound, and a
# workflow step that calls the network declares its own timeout-minutes. A job timeout
# bounds the job, not the step, so it does not satisfy this check.
safe_bound_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in
      *.sh|*.bash|*.zsh)
        awk -v file="$f" '
          /(^|[^A-Za-z0-9_.-])(curl|wget)([ \t]|$)/ && !/^[ \t]*#/ {
            if ($0 ~ /--max-time|--connect-timeout|(^|[ \t])-m[ \t]|--timeout|(^|[ \t])-T[ \t]/) next
            if ($0 ~ /\\[ \t]*$/) { printf "NOT VERIFIED SAFE-001: %s:%d: the time bound of this curl or wget statement continues on the next line; read that line (M-54)\n", file, NR; next }
            printf "FAIL SAFE-001: %s:%d: curl or wget with no time bound (M-54)\n", file, NR
          }' "$root/$f" ;;
      *.yml|*.yaml)
        awk -v file="$f" '
          /^[ \t]*-[ \t]/ {
            if (step && net && !tmo) printf "FAIL SAFE-001: %s:%d: a workflow step calls the network with no step timeout-minutes (M-54)\n", file, step
            step = NR; net = 0; tmo = 0
          }
          /timeout-minutes:/ { tmo = 1 }
          /(^|[^A-Za-z0-9_.-])(curl|wget|gh api|gh release|docker (pull|push))([ \t]|$)/ { net = 1 }
          END {
            if (step && net && !tmo) printf "FAIL SAFE-001: %s:%d: a workflow step calls the network with no step timeout-minutes (M-54)\n", file, step
          }' "$root/$f" ;;
    esac
  done
}

# $1 = repository root; $2... = changed files, relative to it.
safe_endpoint_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in
      *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs)
        case "$f" in *test*|*spec*|*__tests__*|*.d.ts) continue ;; esac
        grep -nE 'https?://(localhost|127\.0\.0\.1)' "$root/$f" 2>/dev/null |
          sed "s#^#FAIL SAFE-003: $f:#" ;;
      *.sh|*.bash|*.zsh)
        grep -nE '/[t]mp/' "$root/$f" 2>/dev/null | grep -v 'TMPDIR' |
          sed "s#^#FAIL SAFE-003: $f:#" ;;
    esac
  done
}
```

The detections use `awk`, `grep`, and `sed`, present on macOS and Linux. Where a
changed file is missing they report nothing for it, so the caller lists the
changed files from `git`, never by hand.

### SAFE-004 to SAFE-007, AUTH-001, DEP-001 — Encoding, Cache Partitions, Rejected Credentials, Backup, Identity Source, and Downgrade

These detections enforce the delivery-safety rules that the external audits found
missed, and they are candidate generators: the reviewer confirms each match before
reporting it.

- **SAFE-002 scheme form (M-55).** A file that validates a URL setting with `.url()`
  and has no scheme refinement accepts `http://`, so a token can travel in cleartext.
  The configuration validates the scheme (`https` only for a non-local build).
- **SAFE-004 (M-59).** A value interpolated into a URL path is encoded with the
  encoder of its context, for example `encodeURIComponent`, before the request is
  built. A raw identifier in a path is a `FAIL`; the encoded form passes.
- **SAFE-005 (M-56).** A cache or storage key with no identity partition (user,
  account, tenant, or owner) is a candidate: the reviewer classifies whether the
  data is identity-owned or public. Payment data written to plain storage is a
  `FAIL`. A sign-out definition that clears no cache partition within 25 lines is a
  `FAIL`, so a sign-out that removes only the token is caught.
- **SAFE-006 (M-60).** A `401` answer that does not end the session within 15 lines
  (sign-out, session end, or cache-partition clear) is a `FAIL`: a rejected
  credential ends the session, and cached data is never served to it.
- **SAFE-007 (M-61).** A build manifest or app configuration that sets
  `allowBackup` to true lets device backup extract the app data. An app that holds
  personal or payment data does not allow backup of personal data, so the setting
  is a `FAIL`, and a manifest that does not exist is `NOT VERIFIED`, never clean.
- **AUTH-001 (M-52).** An identity read from a client-controlled header
  (`x-user-id`) or query (`userId` in a stream URL) is not authentication.

**Scope limit.** The detectors read TypeScript, JavaScript, and POSIX shell, plus the
manifests named above. A semantic ownership check (an object written without a
verified owner) and a device-level run are reviewed under Dimension 6 and are never
reported as passing by these detectors.

```sh
# $1 = repository root; $2... = changed or tracked files, relative to it.
# SAFE-004 (M-59): a template literal that builds a URL path interpolates an
# identifier with no encoder around it.
id_encoding_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    (cd "$root" && perl -ne '
      next unless /`[^`]*\/[^`]*\$\{(?!\s*encodeURIComponent\()\s*[\w.]*(?:Id|ID|id|Slug|slug)\s*\}/;
      print "FAIL SAFE-004: $ARGV:$.: identifier interpolated into a URL path with no encoder (M-59)\n";
    ' "$f")
  done
}

# SAFE-002 scheme form (M-55): a file that validates a URL setting with no
# scheme refinement accepts cleartext http://.
url_scheme_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    grep -qE '\.url\(' "$root/$f" 2>/dev/null || continue
    grep -qE 'refine|superRefine|startsWith|protocol' "$root/$f" && continue
    awk -v file="$f" '/\.url\(/ {
      printf "FAIL SAFE-002: %s:%d: a URL setting accepts any scheme, so cleartext http:// passes (M-55)\n", file, NR
    }' "$root/$f"
  done
}

# SAFE-005 (M-56): a cache or storage key with no identity partition, and
# payment data written to plain storage.
cache_partition_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    (cd "$root" && perl -ne '
      next unless /(?:cacheKey\s*:|(?:readCache|writeCache|clearCache|setItem|getItem|removeItem|multiSet)\s*\()\s*[`"\x27]([^`"\x27]*)/;
      my $key = $1;
      if ($key !~ /user|account|tenant|identity|owner|session|subject/i && !/\$\{[^}]*(?:user|account|tenant|identity|owner|session|subject)/i) {
        print "FAIL SAFE-005: $ARGV:$.: cache key \"$key\" has no identity partition (M-56)\n";
      }
      if ($key =~ /payment|card|iban|cvv/i) {
        print "FAIL SAFE-005: $ARGV:$.: payment data written to plain storage under \"$key\" (M-56)\n";
      }
    ' "$f")
  done
}

# SAFE-005 (M-56), sign-out form: a sign-out definition whose body, within 25
# lines, clears no cache partition of the identity.
signout_clear_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    awk -v file="$f" '
      { lines[NR] = $0 }
      /((const|let|var)[[:space:]]+|function[[:space:]]+)(signOut|logOut|logout)[[:space:]]*[=(]/ { defs[++n] = NR }
      END {
        for (i = 1; i <= n; i++) {
          d = defs[i]; ok = 0
          for (k = d; k <= d + 25 && k <= NR; k++)
            if (lines[k] ~ /clear[A-Za-z]*(Cache|Partition|Identity)|clearAll|multiRemove/) ok = 1
          if (!ok) printf "FAIL SAFE-005: %s:%d: sign-out clears no cache partition of the identity (M-56)\n", file, d
        }
      }' "$root/$f"
  done
}

# SAFE-006 (M-60): a 401 answer that a client does not handle by ending the
# session. Test files are excluded because a test may assert a 401 on purpose,
# and server route handlers are excluded because they emit a 401 to their client
# rather than handling one; the client side is the one reviewed here.
rejected_credential_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*|*/route.ts|*/route.js|server.ts|*/server.ts|*/server/*) continue ;; esac
    awk -v file="$f" '
      { lines[NR] = $0 }
      /(===?|!==?|:)[[:space:]]*401([^0-9]|$)/ { hits[++n] = NR }
      END {
        for (i = 1; i <= n; i++) {
          h = hits[i]; ok = 0
          for (k = h; k <= h + 15 && k <= NR; k++)
            if (lines[k] ~ /signOut|logOut|logout|endSession|clear[A-Za-z]*(Cache|Partition|Identity)/) ok = 1
          if (!ok) printf "FAIL SAFE-006: %s:%d: a 401 answer does not end the session (M-60)\n", file, h
        }
      }' "$root/$f"
  done
}

# AUTH-001 (M-52): an identity read from a client-controlled header or query.
# Test files are excluded: a test sets the header on purpose to exercise the route.
identity_source_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*) continue ;; esac
    (cd "$root" && perl -ne '
      print "FAIL AUTH-001: $ARGV:$.: identity read from or sent in a client-controlled header, query, or connection parameter, not from a verified credential (M-52)\n"
        if /headers(?:\.get\(|\[)\s*["\x27](?:x-user-id|x-user|x-account-id|x-tenant-id)|searchParams\.get\(\s*["\x27](?:userId|user_id|uid|accountId)|(?:query|params)\.(?:userId|user_id)\b|["\x27](?:x-user-id|x-user|x-account-id|x-tenant-id)["\x27]\s*[:,\]}]/i;
    ' "$f")
  done
}

# SAFE-007 (M-61): a build manifest or app configuration that allows backup, or
# that does not set it. On Android the default allows backup, so an unset value
# is a finding too, and the built manifest confirms it.
backup_check() {
  root="$1"; shift
  for f in "$@"; do
    grep -nE 'allowBackup"?[[:space:]]*[:=][[:space:]]*"?true' "$root/$f" 2>/dev/null |
      sed "s#^#FAIL SAFE-007: $f:#"
    grep -qE 'allowBackup' "$root/$f" 2>/dev/null && continue
    grep -qE '<application|android"?[[:space:]]*:' "$root/$f" 2>/dev/null &&
      echo "FAIL SAFE-007: $f:1: allowBackup is not set, and the Android default allows backup (M-61)"
  done
}

# DEP-001 (M-57): a manifest change that moves a dependency to an older major.
# $1 = repository root; $2 = base revision; $3... = changed package.json files.
dependency_downgrade_check() {
  root="$1"; base="$2"; shift 2
  for f in "$@"; do
    case "$f" in */package.json|package.json) ;; *) continue ;; esac
    old=$(mktemp) || return 1
    if git -C "$root" show "$base:$f" > "$old" 2>/dev/null; then
      (cd "$root" && perl -MJSON::PP -e '
        my ($oldf, $newf, $name) = @ARGV;
        my $read = sub { open my $h, "<", $_[0] or die "$_[0]: $!"; local $/; my $t = <$h>; close $h; JSON::PP->new->decode($t) };
        my ($o, $n) = ($read->($oldf), $read->($newf));
        for my $sec (qw(dependencies devDependencies optionalDependencies)) {
          next unless ref $o->{$sec} eq "HASH" && ref $n->{$sec} eq "HASH";
          for my $dep (sort keys %{ $o->{$sec} }) {
            next unless exists $n->{$sec}{$dep};
            my ($om) = ($o->{$sec}{$dep} =~ /(\d+)/);
            my ($nm) = ($n->{$sec}{$dep} =~ /(\d+)/);
            next unless defined $om && defined $nm && $nm < $om;
            my $ln = 0;
            open my $fh, "<", $newf or die "$newf: $!";
            while (my $l = <$fh>) { $ln++; last if $l =~ /"\Q$dep\E"\s*:/ }
            close $fh;
            print "FAIL DEP-001: $name:$ln: $dep moves from $o->{$sec}{$dep} to $n->{$sec}{$dep} in $sec, a downgrade (M-57)\n";
          }
        }
      ' "$old" "$f" "$f")
    fi
    rm -f "$old"
  done
}
```

### SAFE-008, SAFE-009 — Permissive CORS, Development Tools, and Owner-Only Writes

These detections enforce the rules of M-62 and M-63, which the third external
audit found and which the catalog named only as signals:

- **SAFE-008 (M-62), wildcard with credentials.** A wildcard origin
  (`origin: "*"`, `Access-Control-Allow-Origin: *`) within 15 lines of a
  credentials enabling (`credentials: true`, `Allow-Credentials: true`) is a
  `FAIL`: a wildcard origin is never combined with credentials, and the allowed
  origins are an explicit list (M-62). A wildcard with no credentials is not
  reported here.
- **SAFE-008 (M-62), development tool.** A development tool (GraphiQL, a
  playground, an introspection or debug switch) enabled by a literal `true` is a
  `FAIL`. A tool enabled by an expression, such as an environment check, is not
  reported here: the reviewer reads the check and confirms that it is false in
  production.
- **SAFE-009 (M-63), owner-only mode.** A write (`writeFile`, `writeFileSync`,
  `createWriteStream`, `copyFile`) in a file whose path names an upload, receipt,
  attachment, avatar, document, image, or photo is a `FAIL` when no `mode` of
  `0600` or `0400` appears within the call's six lines. A file that holds personal
  or sensitive data is written with an owner-only file mode (M-63), and the default
  file mode is not owner-only.
- **SAFE-009 (M-63), metadata.** Such an upload file that is written with no
  metadata handling in the same file (`exif`, `metadata`, or `sharp`) is
  `NOT VERIFIED` for metadata stripping, never clean.

**Scope limit.** These detectors read TypeScript and JavaScript only. A CORS or
development-tool setting in JSON, YAML, or an environment file is reviewed under
Dimension 6, and an owner-only mode in another language is reviewed the same way.

```sh
# $1 = repository root; $2... = changed or tracked TypeScript or JavaScript files.
# SAFE-008 (M-62): a wildcard origin combined with credentials, and a development
# tool enabled by a literal true. The pattern avoids quote characters so that the
# awk program stays inside its single-quoted block.
cors_debug_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*) continue ;; esac
    awk -v file="$f" '
      { lines[NR] = $0; low = tolower($0) }
      low ~ /origin[^a-z0-9]+\*/ { wild[++w] = NR }
      low ~ /credentials[^a-z0-9]+true/ { cred[++c] = NR }
      low ~ /(graphiql|playground|introspection)[a-z]*[^a-z0-9]+true([^a-z0-9_]|$)/ {
        printf "FAIL SAFE-008: %s:%d: development tool enabled by a literal true, not by a production-off check (M-62)\n", file, NR
      }
      END {
        for (i = 1; i <= w; i++) {
          hit = 0
          for (j = 1; j <= c && !hit; j++) {
            d = wild[i] - cred[j]; if (d < 0) d = -d
            if (d <= 15) hit = 1
          }
          if (hit) printf "FAIL SAFE-008: %s:%d: a wildcard origin is combined with credentials (M-62)\n", file, wild[i]
        }
      }' "$root/$f"
  done
}

# SAFE-009 (M-63): an upload, receipt, or document written with no owner-only
# mode in its call, and an upload path that handles no metadata.
upload_write_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*) continue ;; esac
    case "$f" in *upload*|*receipt*|*attachment*|*avatar*|*document*|*image*|*photo*) ;; *) continue ;; esac
    awk -v file="$f" '
      { lines[NR] = $0 }
      /(writeFile|writeFileSync|createWriteStream|copyFile)[[:space:]]*\(/ { w[++n] = NR }
      END {
        for (i = 1; i <= n; i++) {
          s = w[i]; ok = 0
          for (k = s; k <= NR && k <= s + 6; k++) if (lines[k] ~ /mode[[:space:]]*:|0o?[46]00/) ok = 1
          if (!ok) printf "FAIL SAFE-009: %s:%d: upload written with the default file mode, not owner-only (M-63)\n", file, s
        }
        if (n > 0) {
          meta = 0
          for (k = 1; k <= NR; k++) if (tolower(lines[k]) ~ /exif|metadata|sharp/) meta = 1
          if (!meta) printf "NOT VERIFIED SAFE-009: %s: no metadata stripping found for an upload, so it is not verified (M-63)\n", file
        }
      }' "$root/$f"
  done
}
```

### Whole-project sweep (health probe)

The health probe (`workflows/project-validation.md`, Phase 8a items 6 and 7)
must scan the **whole project**, not only the change set: a pre-existing
violation in unrelated code is a finding, and it is reported without blocking
the task. This sweep is one command that lists every tracked and untracked file,
so no file is skipped by hand, runs every delivery-safety detector, and prints
what it scanned and what it excluded (M-51). It runs from the project root. A
check that needs a base revision is reported as `NOT VERIFIED` for the sweep and
runs on the change set instead.

```sh
# Run from the project root. Falls back to find when the project is not a git repository.
project_safety_sweep() {
  files=$( (git ls-files --cached --others --exclude-standard 2>/dev/null) || (find . -type f | sed 's#^\./##') )
  files=$(printf '%s\n' "$files" | sed '/^$/d' | grep -vE '(^|/)(node_modules|dist|build|\.next|coverage|\.git)/')
  src=$(printf '%s\n' "$files" | grep -E '\.(ts|tsx|js|jsx|mjs|cjs)$' | grep -vE '\.d\.ts$')
  shells=$(printf '%s\n' "$files" | grep -E '\.(sh|bash|zsh)$')
  manifests=$(printf '%s\n' "$files" | grep -E '(^|/)(AndroidManifest\.xml|app\.json|app\.config\.(js|ts|json))$')
  nonbarrel=$(printf '%s\n' "$src" | grep -vE '(^|/)index\.(ts|tsx|js|jsx|mjs|cjs)$')
  mds=$(printf '%s\n' "$files" | grep -E '\.md$')
  decisions=$(printf '%s\n' "$files" | grep -E '(^|/)docs/decisions/.*\.md$')
  catalogs=$(printf '%s\n' "$files" | grep -E '\.md$' | grep -iE '(^|/)catalog[^/]*\.md$')
  contracts=$(printf '%s\n' "$files" | grep -E '\.md$' | grep -iE 'contract|contrato|api|error|docs/')
  workflows=$(printf '%s\n' "$files" | grep -E '\.(ya?ml)$')
  migrations=$(printf '%s\n' "$files" | grep -E '(^|/)(migrations?|db/migrate)/')
  tests=$(printf '%s\n' "$files" | grep -E '(\.test\.|\.spec\.|(^|/)(tests?|__tests__)/)')
  echo "SWEEP scanned: $(printf '%s\n' "$src" | sed '/^$/d' | wc -l | tr -d ' ') source files, $(printf '%s\n' "$shells" | sed '/^$/d' | wc -l | tr -d ' ') shell scripts, $(printf '%s\n' "$manifests" | sed '/^$/d' | wc -l | tr -d ' ') build manifests"
  echo "SWEEP excluded: node_modules, dist, build, .next, coverage, .git, and declaration files (*.d.ts)"
  safe_timeout_check . $src
  safe_cleartext_check . $src $shells $workflows
  safe_bound_check . $shells $workflows
  safe_endpoint_check . $src $shells
  url_scheme_check . $src
  id_encoding_check . $src
  cache_partition_check . $src
  signout_clear_check . $src
  rejected_credential_check . $src
  identity_source_check . $src
  cors_debug_check . $src
  upload_write_check . $src
  backup_check . $manifests
  dead_export_check . $nonbarrel
  i18n_default_locale_check . $src
  i18n_direction_check . $src
  i18n_rtl_test_check . $src
  i18n_copy_check . $src
  a11y_live_region_check . $src
  a11y_android_path_check . $src
  platform_side_effect_check . $src
  for f in $src; do dip_concrete_check . "$f"; done
  artifact_path_check . $mds
  evidence_claim_check . $mds
  precondition_check . $mds
  ci_presence_check .
  node_drift_check .
  dep_range_check .
  decision_record_check . $decisions
  decision_citation_check . $decisions
  claim_check . $mds $src
  comment_claim_check . $src
  error_model_check . $src
  error_outcome_check . $src -- $tests -- $contracts
  entry_point_check . $src -- $tests
  validation_order_check . $src
  read_transaction_check . $src
  db_runtime_ddl_check . $src
  db_deploy_order_check . $migrations -- $tests
  db_visibility_check . $migrations -- $tests
  cache_decision_check . $src
  db_schema_state_check . $src
  index_coverage_check . $src $migrations
  timer_hold_check . $tests
  bound_test_check . $src -- $tests
  destructive_path_check . $src
  parse_dereference_check . $src
  srp_unit_check . $src
  srp_workflow_check . $workflows
  for d in $decisions; do srp_claim_check . "$d" $src $workflows; done
  dup_helper_check . -- $src
  dup_block_check . -- $shells $workflows
  guard_test_check . $shells $workflows -- $tests
  approval_citation_check . $mds
  approval_binding_check . $workflows
  artifact_ref_check . $workflows
  import_effect_check . $src
  exec_claim_check . $mds
  [ -n "$catalogs" ] && echo "NOT VERIFIED TRACE-001: a catalog is tracked; run trace_check . <catalog> <prefix> $mds with the catalog's requirement prefix" ||
    echo "NOT VERIFIED TRACE-001: no catalog is tracked, so no document-catalog link was checked"
  [ -n "$manifests" ] || echo "NOT VERIFIED M-61: no build manifest or app configuration is tracked, so backup is not verified"
  echo "NOT VERIFIED M-57 (downgrade): needs a base revision; run dependency_downgrade_check on the change set"
  echo "NOT VERIFIED COMMIT-001: needs a proposed commit group; run commit_group_closure_check on each group (workflows/commit.md Phase 11)"
  echo "NOT VERIFIED BEHAVIOR-001 (M-94): needs the base revision and the approved manifest; run test_title_removal_check in the commit gate"
  echo "NOT VERIFIED PREFLIGHT-001 (M-95): is a task-level check; run preflight_state_check before alignment"
}
```

### I18N-001 — Formatters take the locale; no default locale literal

**Scope limit (I18N-001 to I18N-004).** These detectors read TypeScript and
JavaScript, and the React Native layout they name. The native layouts (Android XML,
iOS storyboards and SwiftUI), Flutter, Angular and CSS direction, and the platform
catalogs are not read by these detectors: their direction and locale rules are
reviewed under Dimension 11 of `references/audit-pass-criteria.md`, and the verdict
for them is `NOT VERIFIED` for the detectors, never clean.

M-66. A formatter whose locale parameter has a default literal hides the
locale of its caller, and a locale-aware call with no locale argument uses the
runtime default, which is not the user's locale. A candidate is reported with
`file:line`; the reviewer confirms it. Test files are excluded.

```sh
# $1 = repository root; $2... = changed or tracked source files, relative to it.
i18n_default_locale_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*|*/testing/*) continue ;; esac
    grep -nE "[(,][[:space:]]*locale[A-Za-z]*[[:space:]]*(:[^=,)]*)?=[[:space:]]*[\"'][a-z]{2}(-[A-Za-z]{2,4})?[\"']" "$root/$f" 2>/dev/null |
      sed "s#^#FAIL I18N-001: $f:#; s#\$#  (formatter parameter with a default locale literal, M-66)#"
    grep -nE "toLocale(Date|Time)?String\(\)|Intl\.(NumberFormat|DateTimeFormat|PluralRules|RelativeTimeFormat)\(\)" "$root/$f" 2>/dev/null |
      sed "s#^#FAIL I18N-001: $f:#; s#\$#  (locale-aware call with no locale argument, M-66)#"
  done
}
```

### I18N-002 — No literal direction in a project that supports RTL

M-66: every direction-dependent unit resolves its layout direction from the active
direction. The project supports RTL when the layout configuration says so. Under
that condition a literal `flexDirection` row, a literal `left` or `right`
offset, or a literal margin or padding side is a candidate: it relies on
automatic mirroring, which React Native disables with `allowRTL(false)`. The
reviewer confirms each candidate and reads the resolver the unit uses.

```sh
# $1 = repository root; $2... = changed or tracked source files, relative to it.
rtl_project() {
  (cd "$1" && grep -rlE "allowRTL|forceRTL|I18nManager|supportsRTL|dir=\"rtl\"" \
    --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' \
    --include='*.xml' --include='*.html' --include='*.json' \
    --exclude-dir=node_modules --exclude-dir=dist --exclude-dir=build . 2>/dev/null | head -n 1)
}

i18n_direction_check() {
  root="$1"; shift
  [ -n "$(rtl_project "$root")" ] || return 0
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*|*/testing/*) continue ;; esac
    grep -nE "flexDirection:[[:space:]]*[\"']row(-reverse)?[\"']|(margin|padding)(Left|Right)[[:space:]]*:|[[:space:]](left|right):[[:space:]]*[-0-9\"']" "$root/$f" 2>/dev/null |
      sed "s#^#FAIL I18N-002: $f:#; s#\$#  (literal direction in a project that supports RTL, M-66)#"
  done
}
```

### I18N-003 — A UI unit has an RTL assertion in its own test

M-66 and the per-unit coverage rule of `skills/internationalization/SKILL.md`.
In a project that supports RTL, every changed UI unit must have a test next to it
that names an RTL locale (`'ar'`, `rtl`, or `RTL`). A unit with a literal direction
is `FAIL`: it is direction-dependent by its own code. Any other changed UI unit is
`NOT VERIFIED`, because a screen can render a direction-dependent child without a
literal of its own; the reviewer either adds the assertion or records why the unit
renders no direction. A unit with no test is reported too, so the gap is never silent.
The check reads the presence of the locale in the test; the reviewer confirms that
the assertion renders the unit.

```sh
# $1 = repository root; $2... = changed or tracked source files, relative to it.
i18n_rtl_test_check() {
  root="$1"; shift
  [ -n "$(rtl_project "$root")" ] || return 0
  for f in "$@"; do
    case "$f" in *.tsx|*.jsx) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*|*/testing/*) continue ;; esac
    if grep -qE "flexDirection:[[:space:]]*[\"']row|(margin|padding)(Left|Right)[[:space:]]*:|[[:space:]](left|right):[[:space:]]*[-0-9\"']" "$root/$f" 2>/dev/null; then
      verdict="FAIL"
      reason="a literal direction in this unit, so it is direction-dependent"
    else
      verdict="NOT VERIFIED"
      reason="a changed UI unit that may render a direction-dependent child; the reviewer confirms it renders no direction, or adds the assertion"
    fi
    dir="$(dirname "$root/$f")"
    name="$(basename "${f%.*}")"
    tests="$(find "$dir" -maxdepth 1 \( -name "$name.test.*" -o -name "$name.spec.*" \) 2>/dev/null)"
    if [ -z "$tests" ]; then
      echo "$verdict I18N-003: $f:1: no test, so no RTL assertion exists: $reason (M-66)"
      continue
    fi
    printf '%s\n' "$tests" | while IFS= read -r t; do
      grep -qE "'ar'|\"ar\"|rtl|RTL" "$t" 2>/dev/null ||
        echo "$verdict I18N-003: $f:1: its test ${t#"$root"/} names no RTL locale: $reason (M-66)"
    done
  done
}
```

### I18N-004 — Hardcoded sentences in user-facing copy (candidates)

Notification, push, email, and banner copy reaches the user as text, so an
English sentence built in a service is a hardcoded string (M-66). The check
reports candidate sentences in those files for review: two capitalized or
lowercase words in quotes, not inside a translation call. A reviewer confirms
each candidate before it is reported as a defect or as a finding.

```sh
# $1 = repository root; $2... = changed or tracked source files, relative to it.
i18n_copy_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*|*/testing/*) continue ;; esac
    case "$f" in *notification*|*push*|*email*|*mail*|*sms*|*banner*) ;; *) continue ;; esac
    grep -nE "[\"'\`][A-Z][a-z]+ [a-z]+" "$root/$f" 2>/dev/null |
      grep -vE "t\(|i18n|translate" |
      sed "s#^#FAIL I18N-004: $f:#; s#\$#  (candidate hardcoded sentence in user-facing copy, reviewer confirms, M-66)#"
  done
}
```

### CONSUME-001 — A changed unit does not consume a pre-existing i18n defect

M-67. The change set is the list of files it creates or modifies. A changed unit
that imports a relative module outside that list inherits the defect of that
module: the consumed unit is part of the change set, so the consumer is reported
with both lines. The check resolves each relative import of a changed source
file to its target and reports the defects of the target that the detectors
above define: a default locale literal, a literal row direction in a project that
supports RTL, and a hardcoded sentence in notification copy. A target that is in
the change set is not reported here, because its own detector reports it. An aliased
import (`from "@/..."`) is not resolved by this check: the check reports it as
`NOT VERIFIED`, never as clean.

```sh
# $1 = repository root; $2... = changed source files, relative to it. It uses
# rtl_project from the I18N-002 block.
consumed_defect_check() {
  root="$1"; shift
  rootabs="$(cd "$root" && pwd)"
  changed="$(printf '%s\n' "$@")"
  rtl="$(rtl_project "$root")"
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*|*/testing/*) continue ;; esac
    grep -nE "from [\"']@/" "$rootabs/$f" 2>/dev/null |
      sed "s#^#NOT VERIFIED CONSUME-001: $f:#; s#\$#  (an aliased import is not resolved by this check: review its target, M-67)#"
    dir="$(dirname "$rootabs/$f")"
    grep -nE "from [\"']\.[^\"']*[\"']" "$rootabs/$f" 2>/dev/null |
      while IFS= read -r line; do
        lineno="${line%%:*}"
        spec="$(printf '%s' "$line" | sed -E "s/^[^:]*:.*from [\"']([^\"']*)[\"'].*$/\1/")"
        d="$(cd "$dir" && cd "$(dirname "$spec")" 2>/dev/null && pwd)" || continue
        b="$d/$(basename "$spec")"
        target=""
        for c in "$b.tsx" "$b.ts" "$b.jsx" "$b.js" "$b/index.tsx" "$b/index.ts"; do
          [ -f "$c" ] && { target="$c"; break; }
        done
        [ -n "$target" ] || continue
        rel="${target#"$rootabs"/}"
        printf '%s\n' "$changed" | grep -qxF "$rel" && continue
        defect=""
        hit="$(grep -nE "[(,][[:space:]]*locale[A-Za-z]*[[:space:]]*(:[^=,)]*)?=[[:space:]]*[\"'][a-z]{2}" "$target" 2>/dev/null | head -n 1)"
        [ -n "$hit" ] && defect="a default locale literal at $rel:${hit%%:*}"
        if [ -z "$defect" ] && [ -n "$rtl" ]; then
          hit="$(grep -nE "flexDirection:[[:space:]]*[\"']row(-reverse)?[\"']" "$target" 2>/dev/null | head -n 1)"
          [ -n "$hit" ] && defect="a literal row direction at $rel:${hit%%:*}"
        fi
        if [ -z "$defect" ]; then
          case "$rel" in *notification*|*push*|*banner*)
            hit="$(grep -nE "[\"'\`][A-Z][a-z]+ [a-z]+" "$target" 2>/dev/null | grep -vE "t\(|i18n|translate" | head -n 1)"
            [ -n "$hit" ] && defect="a hardcoded sentence in user-facing copy at $rel:${hit%%:*}" ;;
          esac
        fi
        [ -n "$defect" ] && printf 'FAIL CONSUME-001: %s:%s: imports %s, which carries %s (M-67)\n' "$f" "$lineno" "$spec" "$defect"
      done
  done
}
```

### DEC-001 — A decision record names its patterns and the five SOLID rows

M-50 and M-45: its record is in the change set and names its patterns. M-78: an SRP row names the unit and each responsibility it keeps with `file:line`, and a selected pattern names at least two consumers with `file:line`. A record in
the project's records convention, or in `docs/decisions/`, that selects design
patterns must contain the pattern table
(`| Pattern |`) and exactly one SOLID row per principle, each with a status of
`APPLIES`, `PRESERVED`, or `NOT APPLICABLE`. A count alone fails. The form is
the one in `references/design-patterns.md` ("Presentation Record").

```sh
# $1 = repository root; $2... = decision records (markdown), relative to it.
decision_record_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.md) ;; *) continue ;; esac
    grep -qiE "pattern|patr" "$root/$f" 2>/dev/null || continue
    grep -qE "^\| *Pattern *\|" "$root/$f" 2>/dev/null ||
      echo "FAIL DEC-001: $f:1: no pattern table names the selected patterns (M-50, M-45)"
    for p in SRP OCP LSP ISP DIP; do
      row="$(grep -E "^\| *$p *\|" "$root/$f" 2>/dev/null | head -n 1)"
      if [ -z "$row" ]; then
        echo "FAIL DEC-001: $f:1: no SOLID row for $p (M-50, M-45)"
      elif ! printf '%s\n' "$row" | grep -qE "APPLIES|PRESERVED|NOT APPLICABLE"; then
        echo "FAIL DEC-001: $f: the SOLID row for $p has no status (M-50, M-45)"
      fi
    done
    # M-78: the SRP row cites the unit and its responsibilities with file:line.
    srp="$(grep -E "^\| *SRP *\|" "$root/$f" 2>/dev/null | head -n 1)"
    if [ -n "$srp" ] && ! printf '%s\n' "$srp" | grep -qE '[A-Za-z0-9_./-]+\.[A-Za-z0-9]+:[0-9]+'; then
      echo "FAIL DEC-001: $f: the SRP row cites no file:line for its unit (M-78)"
    fi
    # M-78: every pattern row names at least two consumers with file:line.
    awk '/^\| *Pattern *\|/{on=1; next} on && /^\|[-| :]*\|$/{next} on && /^\|/{print NR": "$0; next} {on=0}' "$root/$f" 2>/dev/null |
    while IFS= read -r prow; do
      refs="$(printf '%s\n' "$prow" | grep -oE '[A-Za-z0-9_./-]+\.[A-Za-z0-9]+:[0-9]+' | wc -l | tr -d ' ')"
      [ "$refs" -ge 2 ] ||
        echo "FAIL DEC-001: $f: a pattern row names $refs consumer(s) with file:line, two are required (M-78): ${prow%%:*}"
    done
    # M-96: a DIP row that claims APPLIES or PRESERVED is contradicted by any DIP-001 finding
    # in the source tree. The record states the composition root; the concrete imports decide.
    dip_row="$(grep -E "^\| *DIP *\|" "$root/$f" 2>/dev/null | head -n 1)"
    # M-107: a DIP row that claims APPLIES or PRESERVED cites the composition root with file:line.
    if printf '%s\n' "$dip_row" | grep -qE "APPLIES|PRESERVED" &&
       ! printf '%s\n' "$dip_row" | grep -qE '[A-Za-z0-9_./-]+\.[A-Za-z0-9]+:[0-9]+'; then
      echo "FAIL DEC-001: $f: the DIP row claims APPLIES or PRESERVED and cites no file:line for the composition root (M-107)"
    fi
    if printf '%s\n' "$dip_row" | grep -qE "APPLIES|PRESERVED"; then
      dip_hits="$(cd "$root" && find . -type f \( -name '*.ts' -o -name '*.tsx' -o -name '*.js' -o -name '*.jsx' -o -name '*.mjs' -o -name '*.cjs' \) -not -path '*/node_modules/*' -not -path '*/dist/*' -not -path '*/build/*' | sed 's#^\./##' | while IFS= read -r src; do dip_concrete_check . "$src"; done)"
      if [ -n "$dip_hits" ]; then
        count="$(printf '%s\n' "$dip_hits" | wc -l | tr -d ' ')"
        first="$(printf '%s\n' "$dip_hits" | head -n 1)"
        echo "FAIL DEC-001: $f:1: the DIP row claims a status that DIP-001 contradicts: $count concrete import(s), first: $first (M-96)"
      fi
    fi
  done
}
```

M-78 is checked on the record as written. A pattern row that names a single consumer is a `FAIL`, and the reviewer also confirms that the second consumer exists in the change set or the sweep, not only in the text.

### CLAIM-001 — Universal, exclusive, and artifact claims are enumerated first

M-68: a claim that is verified before it is kept. A README, document, or comment that says `only`, `every`, `each`, `all`,
or the equivalent (`solo`, `único`, `cada`, `todos`), or that cites a
screenshot, capture, log, or image, makes a claim that must be checked against
the repository before it is kept. The check cannot decide a claim, so every
match is reported as `NOT VERIFIED` with its line: the verifier enumerates every element of the
scope with a search, or confirms that the artifact exists, and cites the result.

A correction to a claim is searched across the change set and the sweep: a surviving
occurrence is a finding, even when the test or the contract was already corrected (M-75).

M-65 extends the same rule to two claims the check cannot decide from text: a
**performance claim** (`faster`, `más rápido`, `speeds up`) needs a before-and-after
measurement of the named query, and a **technical property** (`covering index`,
`index-only`, `constant time`) needs the query plan or the code that decides it. Both
are `NOT VERIFIED` with their line until the evidence is cited. The sweep reads the
source comments as well as the documents, so a claim in a `.ts` comment is not
skipped (`project_safety_sweep` passes `$src` to `claim_check`).

```sh
# $1 = repository root; $2... = documents or source files, relative to it.
# Prints "LINE:TEXT" for the prose the claim rule reads: the comments of a source file,
# and the whole text of a document or script. Code is not prose: a call such as
# `.all(` is not a universal claim (M-68 scopes CLAIM-001 to documents and comments).
claim_text() {
  case "$1" in
    *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs)
      awk '{ l = $0; if (match(l, /\/\/.*$/)) print NR ":" substr(l, RSTART); else if (l ~ /^[ \t]*(\/\*|\*)/) print NR ":" l }' "$1" ;;
    *) awk '{ print NR ":" $0 }' "$1" ;;
  esac
}

claim_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.md|*.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs|*.sh) ;; *) continue ;; esac
    claim_text "$root/$f" | grep -Ei "(^|[^A-Za-z])(only|every|each|all|always|never|solo|todos|todas|cada|único|unico|siempre|nunca)([^A-Za-z]|$)" |
      sed "s#^#NOT VERIFIED CLAIM-001: $f:#; s#\$#  (universal or exclusive claim: enumerate its scope before keeping it, M-68)#"
    claim_text "$root/$f" | grep -Ei "capturas?|screenshots?|\.png|\.jpe?g|\.gif|evidence|evidencia" |
      sed "s#^#NOT VERIFIED CLAIM-001: $f:#; s#\$#  (claim that cites an artifact: confirm the artifact is in the repository, M-68)#"
    claim_text "$root/$f" | grep -Ei "order-safe|order safe|either order|any deployment order|both directions|ambas direcciones|cualquier orden|en ambos sentidos|keeps working|still (works|serves|reads)|(previous|old|older|prior|legacy) (release|version|instance|build|schema)[^.]*(keeps|continues|still)[[:space:]]+(working|serving|reading|running)|(versi[oó]n|release) anterior[^.]*sigue[[:space:]]+(funcionando|operando|sirviendo|leyendo)|sigue funcionando" |
      sed "s#^#NOT VERIFIED CLAIM-001: $f:#; s#\$#  (compatibility claim: keep it only with a test that executes each deployment order, M-69)#"
    claim_text "$root/$f" | grep -Ei "(^|[^A-Za-z])(faster|más rápid[oa]s?|speed(s|ed)? up|accelerat(es|ed|e))([^A-Za-z]|$)" |
      sed "s#^#NOT VERIFIED CLAIM-001: $f:#; s#\$#  (performance claim: cite the before and after measurement of the named query, M-65)#"
    claim_text "$root/$f" | grep -Ei "(^|[^A-Za-z])(expensive|costly|cheaper|slower|cheap|lo caro|caro|costos[oa]s?|barat[oa]s?|más lent[oa]s?|lent[oa]s?)([^A-Za-z]|$)" |
      sed "s#^#NOT VERIFIED CLAIM-001: $f:#; s#\$#  (cost or speed claim: cite the measurement of the named operation, M-65)#"
    claim_text "$root/$f" | grep -Ei "covering index|index[- ]only|índice (cubriente|de cobertura)|constant[- ]time|tiempo constante|O\(1\)|O\(log n\)" |
      sed "s#^#NOT VERIFIED CLAIM-001: $f:#; s#\$#  (technical property claim: cite the query plan or the code that decides it, M-65)#"
  done
}
```

### ERR-001 — A module raises its failures through its typed error class

M-81: a failure raised from a module uses the module's typed error class, never a
generic `Error`, unless the project has no error model and that is stated. A
project has an error model when a non-test source file in **any folder** declares a
class that extends `Error`. Once it does, a bare `new Error(` or `throw Error(` in
any source file of the project is a `FAIL` with its line, because the caller can no
longer tell the project's failures apart from a programming error.

**Scope.** The error model is read from the whole project, not from the folder of the
file under review. The earlier folder-local reading let a module with no error file
in its own folder escape the rule even when the project declared a typed error class
elsewhere; that reading is retired (M-81). The detector reads the whole project for
the typed class, and reads the source files of the change set and the sweep for the
generic error. A `new Error(` in a comment is a candidate: the reviewer confirms each
match before reporting it.

```sh
# $1 = repository root; $2... = source files, relative to it.
error_model_check() {
  root="$1"; shift
  typed="$(grep -rlE 'class[[:space:]]+[A-Za-z0-9_]+[[:space:]]+extends[[:space:]]+Error' "$root" \
    --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' --include='*.mjs' --include='*.cjs' \
    --exclude-dir=node_modules --exclude-dir=dist --exclude-dir=build --exclude-dir=.git --exclude-dir=coverage 2>/dev/null |
    grep -vE '\.(test|spec)\.|/(tests?|__tests__|testing)/' | head -n 1)"
  [ -n "$typed" ] || return 0
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/test/*|*/tests/*|*__tests__*|*/testing/*) continue ;; esac
    grep -nE '(new|throw)[[:space:]]+Error[[:space:]]*\(' "$root/$f" 2>/dev/null |
      sed "s#^#FAIL ERR-001: $f:#; s#\$#  (generic Error in a project with a typed error class: raise the typed error, M-81)#"
  done
}
```

### ERR-002 and ERR-003 — Error codes are declared and tested in both directions

M-118: each error branch and each exit code is covered on its own. A code emitted from several sites is
checked per site (`NOT VERIFIED` until the test references reach the number of sites), and an exit code the
entry point sets is asserted by a test that invokes it (`ENTRY-001`).

M-77: every error code a boundary emits is declared in the contract that names it
(`ERR-003`), and every error code a contract declares has an executing test that
asserts it (`ERR-002`). The codes are the upper-snake identifiers that a source file
quotes as a literal, and the identifiers a contract document states on a line that
names an error, a failure, or an exit. `ERR-002` is the documented-but-untested
case: an implemented and documented code that no test names. `ERR-003` is the
emitted-but-undeclared case, where a literal in the source is absent from every
contract. Both are candidates: the reviewer confirms that a literal is an error code
and not a configuration key before reporting it.

```sh
# $1 = repository root; $2... = source files; then `--`; then test files; then `--`; then contract documents. All relative to root.
error_outcome_check() {
  root="$1"; shift
  srcs=""; tests=""; docs=""; mode=src
  for a in "$@"; do
    if [ "$a" = "--" ]; then
      if [ "$mode" = src ]; then mode=tests; else mode=docs; fi
      continue
    fi
    case "$mode" in
      src) srcs="$srcs$a
" ;;
      tests) tests="$tests$a
" ;;
      docs) docs="$docs$a
" ;;
    esac
  done
  code_re='[A-Z][A-Z0-9]*(_[A-Z0-9]+)+'
  # Codes named by any test file.
  tested="$(printf '%s' "$tests" | while IFS= read -r t; do
    [ -n "$t" ] && [ -f "$root/$t" ] && grep -ohE "$code_re" "$root/$t"
  done | sort -u)"
  # Documented codes: file:line:CODE, backticked, on a line that names an error, a failure, or an exit.
  documented="$(printf '%s' "$docs" | while IFS= read -r d; do
    [ -n "$d" ] && [ -f "$root/$d" ] || continue
    awk -v f="$d" '{ line = $0; if (tolower(line) ~ /error|fail|exit/) {
      while (match(line, /`[A-Z][A-Z0-9]*(_[A-Z0-9]+)+`/)) {
        print f ":" NR ":" substr(line, RSTART + 1, RLENGTH - 2); line = substr(line, RSTART + RLENGTH) } } }' "$root/$d"
  done)"
  # Emitted codes: file:line:CODE, quoted as a literal in a source file.
  emitted="$(printf '%s' "$srcs" | while IFS= read -r s; do
    [ -n "$s" ] && [ -f "$root/$s" ] || continue
    grep -noE "[\"'\`][A-Z][A-Z0-9]*(_[A-Z0-9]+)+[\"'\`: ]" "$root/$s" | tr -d "\"'\`" | sed -E 's/[: ]$//' | sed "s#^#$s:#"
  done)"
  dcodes="$(printf '%s\n' "$documented" | awk -F: 'NF { print $3 }' | sort -u)"
  printf '%s\n' "$emitted" | while IFS= read -r e; do
    [ -n "$e" ] || continue
    c="${e##*:}"
    printf '%s\n' "$dcodes" | grep -qx "$c" ||
      echo "FAIL ERR-003: ${e%:*}: error code $c is emitted and no contract declares it (candidate: confirm it is an error code, M-77)"
  done
  # M-118: a code emitted from several sites is covered per site. One test line that names the
  # code does not prove that each condition is tested, so the code stays NOT VERIFIED until the
  # test references reach the number of sites or the branch coverage of the emitting file is cited.
  printf '%s\n' "$emitted" | awk -F: 'NF { if (!($3 in first)) first[$3] = $1 ":" $2; sites[$3]++ }
    END { for (c in sites) if (sites[c] > 1) print c, sites[c], first[c] }' | while read -r c k where; do
    m="$(printf '%s' "$tests" | while IFS= read -r t; do
      [ -n "$t" ] && [ -f "$root/$t" ] && grep -c -- "$c" "$root/$t"
    done | awk '{ s += $1 } END { print s + 0 }')"
    [ "$m" -ge "$k" ] ||
      echo "NOT VERIFIED ERR-002: ${where%:*}:${where##*:}: error code $c is emitted from $k sites and named by $m test line(s); each condition needs its own test or the branch coverage of this file (M-118)"
  done
  printf '%s\n' "$documented" | while IFS= read -r d; do
    [ -n "$d" ] || continue
    c="${d##*:}"
    printf '%s\n' "$tested" | grep -qx "$c" ||
      echo "FAIL ERR-002: ${d%:*}: documented error code $c has no executing test (M-77)"
  done
}
```

### ENTRY-001 — A project entry point is tested through its real invocation

M-82: a project's entry point (a command-line program, a `bin` script, or a process
`main`) is tested through its real invocation, by a test that runs it, and the exit
status of each documented code is asserted. The detector
identifies an entry point by a `process.argv` read. When the entry point also sets
an exit status (`process.exit` or `process.exitCode`), a test that names it must
assert that status. A test that imports a helper from the entry point's folder does
not name the entry point, so it does not satisfy the rule.

**Scope limit.** The detector proves that a test names the entry point in a quoted
path or module specifier and that the test asserts a status. It does not prove that
each documented flag or argument error has its own case; the reviewer maps the
documented exit codes to the test cases.

```sh
# $1 = repository root; $2... = source files; then `--`; then test files. Relative to root.
entry_point_check() {
  root="$1"; shift
  srcs=""; tests=""; mode=src
  for a in "$@"; do
    if [ "$a" = "--" ]; then mode=tests; continue; fi
    if [ "$mode" = src ]; then srcs="$srcs$a
"; else tests="$tests$a
"; fi
  done
  printf '%s' "$srcs" | while IFS= read -r f; do
    [ -n "$f" ] && [ -f "$root/$f" ] || continue
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/test/*|*/tests/*|*__tests__*|*/testing/*) continue ;; esac
    grep -qE 'process\.argv' "$root/$f" || continue
    base="$(basename "$f")"; base="${base%.*}"
    invokers="$(printf '%s' "$tests" | while IFS= read -r t; do
      [ -n "$t" ] && [ -f "$root/$t" ] || continue
      grep -qE "['\"/]$base(\.[a-z]+)?['\"]" "$root/$t" && printf '%s\n' "$t"
    done)"
    if [ -z "$invokers" ]; then
      echo "FAIL ENTRY-001: $f:1: an entry point that no test invokes through its real invocation (M-82)"
      continue
    fi
    grep -qE 'process\.exit|exitCode *=' "$root/$f" || continue
    asserted="$(printf '%s\n' "$invokers" | while IFS= read -r t; do
      grep -lE 'exitCode|\.status|[Ee]xit' "$root/$t" 2>/dev/null
    done)"
    [ -n "$asserted" ] || echo "FAIL ENTRY-001: $f:1: the entry point sets an exit status and no test that invokes it asserts one (M-82)"
    # M-118: each exit code the entry point can set is asserted by a test that invokes it. The codes
    # are the literals it passes to process.exit or assigns to exitCode, and every constant the file
    # declares as EXIT_NAME = N (a returned constant reaches process.exit through a function). A test
    # asserts a code by its literal or by the constant's name, never by a test title.
    decl="$(grep -oE 'EXIT_[A-Z_]+ *= *[0-9]+' "$root/$f" | sed -E 's/ *= */=/')"
    codes="$( { grep -oE '(process\.exit\(|exitCode *= *)[0-9]+' "$root/$f" | grep -oE '[0-9]+$';
      printf '%s\n' "$decl" | grep -oE '[0-9]+$'; } | sort -u)"
    for n in $codes; do
      names="$(printf '%s\n' "$decl" | awk -F= -v n="$n" '$2 == n { print $1 }' | tr '\n' '|' | sed 's/|$//')"
      tok="(^|[^0-9])$n([^0-9]|\$)"
      [ -n "$names" ] && tok="$tok|$names"
      ok="$(printf '%s\n' "$invokers" | while IFS= read -r t; do
        grep -vE '^[[:space:]]*(test|it|describe)\(' "$root/$t" 2>/dev/null |
          grep -E "(status|exitCode|[Ee]xit|code)[^;]*($tok)|($tok)[^;]*(status|exitCode|[Ee]xit)"
      done)"
      [ -n "$ok" ] || echo "FAIL ENTRY-001: $f:1: exit code $n is set by the entry point and no test that invokes it asserts $n (M-118)"
    done
  done
}
```

### VAL-001 — An input is validated before a resource is opened

M-83: input is validated before any side effect. An entry point validates its
arguments before it opens a file, a connection, a database, or a network resource, so
a rejected invocation leaves no artifact. Within
an entry point file, the detector reads the first resource-opening call and the first
validator call. An opening that comes before the first validator is a `FAIL` at the
opening line. An entry point whose validator is not in the same file cannot be
ordered by this detector, so every opening in it is a `NOT VERIFIED` candidate until
the reviewer traces the validator's call.

```sh
# $1 = repository root; $2... = source files, relative to it.
validation_order_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/test/*|*/tests/*|*__tests__*|*/testing/*) continue ;; esac
    grep -qE 'process\.argv' "$root/$f" || continue
    open="$(grep -nE 'new (Sqlite|Database|Pool|Client)|createConnection\(|openDatabase\(|createWriteStream\(|open(Sync)?\(|mkdirSync\(|writeFileSync\(|create[A-Za-z]*Adapter\(' "$root/$f" | head -n 1)"
    [ -n "$open" ] || continue
    val="$(grep -nE '(parse|validate|assert|isValid)[A-Za-z]*\(' "$root/$f" | head -n 1)"
    openline="${open%%:*}"
    if [ -n "$val" ] && [ "$openline" -lt "${val%%:*}" ]; then
      echo "FAIL VAL-001: $f:$openline: a resource is opened before the input is validated at line ${val%%:*} (M-83)"
    else
      [ -n "$val" ] || echo "NOT VERIFIED VAL-001: $f:$openline: a resource is opened and no validator is in this file; the reviewer traces the validator's call (M-83)"
    fi
  done
}
```

### DB-003 — A paginated read in a loop runs in one read transaction

M-84: a paginated or cursor-based read that issues several statements, which is a page
loop, runs in one read transaction or snapshot. Without one, a row that changes between
pages is counted twice or skipped. The detector identifies a file that holds a page
query (`LIMIT`, `OFFSET`, a cursor, or a keyset) inside a loop and holds no transaction
or snapshot marker. It reports the page query's line. It is a candidate: a single
statement that is not a loop is not a multi-statement read, and the reviewer confirms
the loop before reporting it.

```sh
# $1 = repository root; $2... = source files, relative to it.
read_transaction_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs|*.py|*.java|*.kt|*.go|*.rb|*.cs|*.php) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/test/*|*/tests/*|*__tests__*|*/testing/*) continue ;; esac
    grep -qiE 'limit|offset|cursor|keyset|after_?id' "$root/$f" || continue
    grep -qE 'while[[:space:]]*\(|for[[:space:]]*\(|\bloop\b' "$root/$f" || continue
    grep -qiE 'BEGIN( DEFERRED| IMMEDIATE| EXCLUSIVE)?|transaction|snapshot|REPEATABLE READ|START TRANSACTION' "$root/$f" && continue
    line="$(grep -niE 'limit|offset|cursor|keyset' "$root/$f" | head -n 1)"
    echo "FAIL DB-003: $f:${line%%:*}: a paginated read in a loop with no read transaction or snapshot; a row changed between pages can be counted twice or skipped (M-84)"
  done
}
```

### DB-001 — Application code issues no schema DDL

M-70: application code never changes the schema. A schema statement belongs to a
migration and runs only from the migration tooling. A `CREATE`, `ALTER`, or `DROP` of a table or index in an application
source file, outside a migration folder and outside a test, is a `FAIL` with its
line, because it runs at startup or in a request path and bypasses the tooling.
The detector matches the SQL keywords, so a prose mention in a comment is a
candidate: the reviewer confirms each match before reporting it.

**Scope limit.** The detector reads source files by extension and excludes the
migration folders (`migrations/`, `migrate/`) and the test files. A schema
statement built from a language-level query builder with no SQL keyword is not
matched; that case is reviewed under Dimension 5 of `references/audit-pass-criteria.md`.

```sh
# $1 = repository root; $2... = source files, relative to it.
db_runtime_ddl_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs|*.py|*.go|*.java|*.kt|*.cs|*.rb|*.php|*.swift|*.dart) ;; *) continue ;; esac
    case "$f" in *migration*|*migrate*|*/test/*|*/tests/*|*__tests__*|*.test.*|*.spec.*) continue ;; esac
    grep -niE "(^|[^A-Za-z])(create|alter|drop)[[:space:]]+(unique[[:space:]]+)?(table|index)([^A-Za-z]|$)|(^|[^A-Za-z])alter[[:space:]]+table([^A-Za-z]|$)" "$root/$f" 2>/dev/null |
      sed "s#^#FAIL DB-001: $f:#; s#\$#  (schema DDL in application code: move it to a migration, M-70)#"
  done
}
```

### DB-002 — A deploy-order test accompanies a migration

M-69: a database change is compatible in either deployment order, so a test must
execute the code against a database created by the other schema, in each order.
The detector finds the migrations and the test files of the project. When a
migration exists and no test file names the legacy, previous, or unmigrated
schema, the migration is a `FAIL` with its file. When such a test is named, the
result is `NOT VERIFIED`: the presence of the name is a signal, and the test must
be executed and its result read before the order is claimed.

```sh
# $1 = repository root; $2... = migration files, then `--`, then test files, relative to it.
DEPLOY_ORDER_TEST_RE='legacy|previous (release|schema|version)|unmigrated|not yet migrated|pre-?migration|old schema|before (the )?migration|sin migrar|esquema anterior'
db_deploy_order_check() {
  root="$1"; shift
  migs=""
  while [ "$#" -gt 0 ] && [ "$1" != "--" ]; do migs="$migs $1"; shift; done
  [ "$#" -gt 0 ] && shift
  [ -n "$migs" ] || return 0
  found=""
  for t in "$@"; do
    grep -qiE "$DEPLOY_ORDER_TEST_RE" "$root/$t" 2>/dev/null && found="$found $t"
  done
  for m in $migs; do
    if [ -n "$found" ]; then
      first="$(printf '%s\n' $found | sed -n '1p')"
      echo "NOT VERIFIED DB-002: $m:1: a deploy-order test is named ($first); execute it in both orders before claiming order-safety (M-69)"
    else
      echo "FAIL DB-002: $m:1: no test executes this migration in either deployment order (M-69)"
    fi
  done
}
```

### DB-005 — A visibility migration asserts the previous release on a hidden row

M-100: "the previous release keeps working" is observable behavior. A migration that
adds a tombstone or a visibility column is a visibility change (`deleted_at`, `archived_at`, `revoked_at`,
`is_deleted`, `hidden_at`, `tombstone`) hides rows from the reads of every release
that does not filter on it. The deploy-order test must name that column and the
previous or legacy read path, so that the previous release's result on a row with
the flag set is asserted. The detector is textual: it proves the test names both,
and the test must still be executed before the claim is kept. The sequencing of the
hiding write (expand-contract) is a plan requirement reviewed under Dimension 5; this
detector does not decide it. It reuses `DEPLOY_ORDER_TEST_RE` from `DB-002`.

```sh
# $1 = repository root; $2... = migration files, then `--`, then test files, relative to it.
VISIBILITY_COL_RE='deleted_at|archived_at|revoked_at|is_deleted|hidden_at|tombstone'
db_visibility_check() {
  root="$1"; shift
  migs=""
  while [ "$#" -gt 0 ] && [ "$1" != "--" ]; do migs="$migs $1"; shift; done
  [ "$#" -gt 0 ] && shift
  for m in $migs; do
    grep -qiE "$VISIBILITY_COL_RE" "$root/$m" 2>/dev/null || continue
    col="$(grep -oiE "$VISIBILITY_COL_RE" "$root/$m" | sed -n '1p')"
    ok=""
    for t in "$@"; do
      if grep -qiE "$col" "$root/$t" 2>/dev/null && grep -qiE "$DEPLOY_ORDER_TEST_RE" "$root/$t" 2>/dev/null; then ok="$t"; fi
    done
    if [ -n "$ok" ]; then
      echo "NOT VERIFIED DB-005: $m:1: visibility column $col; $ok names the previous read path; execute it on a row with $col set before the claim is kept (M-100)"
    else
      echo "FAIL DB-005: $m:1: visibility column $col with no test that runs the previous release against a row with $col set (M-100)"
    fi
  done
}
```

### CACHE-001 — A cache never decides a charge, an authorization, or a revocation alone

M-101: caches never decide a charge, an authorization, or a revocation alone. A
process-local cache is read in a file that also decides such an action; invalidating
the process that wrote does not reach the others, so a TTL is not a bound. The
detector reports each such file as a candidate with its first cache read. The
result is `NOT VERIFIED` until the reviewer shows the source-of-truth read on the
decision path, and a cache used only for display is not a finding. Test files are
excluded.

```sh
CACHE_READ_RE='(cache|Cache)\.(get|has)\(|readCache\(|getCached|lruCache\.get|memo\.get\('
CACHE_DECISION_RE='(^|[^A-Za-z])(charge|authori[sz]e|checkout|pay|revoke|refund|debit)[A-Za-z]*'
cache_decision_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs|*.py|*.go|*.java|*.kt|*.cs|*.rb|*.php|*.swift|*.dart) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/test/*|*/tests/*|*__tests__*) continue ;; esac
    grep -qE "$CACHE_READ_RE" "$root/$f" 2>/dev/null || continue
    grep -qE "$CACHE_DECISION_RE" "$root/$f" 2>/dev/null || continue
    line="$(grep -nE "$CACHE_READ_RE" "$root/$f" | sed -n '1p' | cut -d: -f1)"
    echo "NOT VERIFIED CACHE-001: $f:$line: a cache is read in a file that decides a charge, authorization, or revocation; show the source-of-truth read on that path (M-101)"
  done
}
```

### DB-006 — Schema state is re-read per operation, not captured at process start

M-102: a value derived from the schema state is re-read for each operation or
revalidated after a migration. A capture in a field (`this.x = ...`) or in a
module-level constant freezes it at process start, so a migration that runs while the
process serves traffic is never seen: the store keeps refusing deletes until a restart.
The detector reports the capture with `file:line`. A probe inside a method body is a
per-operation read and is not reported. Test files and migrations are excluded.

```sh
SCHEMA_PROBE_RE='PRAGMA[[:space:]]+table_info|information_schema|hasColumn|columnExists|hasDeleted|tracksDeletions|hasTable|tableExists'
db_schema_state_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs|*.py|*.go|*.java|*.kt|*.cs|*.rb|*.php|*.swift|*.dart) ;; *) continue ;; esac
    case "$f" in *migration*|*migrate*|*.test.*|*.spec.*|*/test/*|*/tests/*|*__tests__*) continue ;; esac
    grep -nE "^[[:space:]]*this\.[A-Za-z_]+[[:space:]]*=.*($SCHEMA_PROBE_RE)" "$root/$f" 2>/dev/null |
      sed "s#^#FAIL DB-006: $f:#; s#\$#  (schema state captured at process start: re-read it per operation, M-102)#"
    grep -nE "^(const|let|var)[[:space:]].*=.*($SCHEMA_PROBE_RE)" "$root/$f" 2>/dev/null |
      sed "s#^#FAIL DB-006: $f:#; s#\$#  (schema state captured in a module constant: re-read it per operation, M-102)#"
  done
}
```

### DB-004 — A query predicate is served by an index on its column

M-85: a query predicate on an application table is served by an index whose leading
column is the predicate's column. A `WHERE col = ?` (or `$1`, `:name`, `@name`) on a
table that holds no index with `col` as its leading column is a full scan on every
miss, and the external audit found exactly that on a list query that filtered by the
owner's identifier. The detector reads the SQL statements of the source files and the
index definitions of every file it is given (migrations and the store files that
create an index at open, which also appear in `DB-001`).

**Scope limit.** The detector reads a `SELECT` with one `FROM` table and a `WHERE` or
`AND` equality on a column. A column covered by a `PRIMARY KEY` or `UNIQUE` constraint
in its `CREATE TABLE` is covered. A statement with a `JOIN` or a comma-separated `FROM`
cannot be resolved to one table, so its predicates are `NOT VERIFIED`, never passed.
The detector is a candidate generator for the Dimension 5 review, which confirms the
statement and the access pattern; it does not replace `EXPLAIN` for a performance
claim (M-65, `CLAIM-001`).

```sh
# $1 = repository root; $2... = source and migration files, relative to it.
index_coverage_check() {
  root="$1"; shift
  (cd "$root" && perl -e '
    my (%lead, %src);
    my @files = grep { !m{(^|/)(test|tests|spec|__tests__)/|\.(test|spec)\.} } @ARGV;
    for my $f (@files) {
      open my $h, "<", $f or next;
      local $/; my $s = <$h>; close $h;
      $src{$f} = $s;
      while ($s =~ /CREATE\s+(?:UNIQUE\s+)?INDEX\s+(?:IF\s+NOT\s+EXISTS\s+)?\S+\s+ON\s+[`"]?(\w+)[`"]?\s*\(\s*[`"]?(\w+)/gi) {
        $lead{lc $1}{lc $2} = 1;
      }
      while ($s =~ /CREATE\s+TABLE\s+(?:IF\s+NOT\s+EXISTS\s+)?[`"]?(\w+)[`"]?\s*\((.*?)\)\s*;/gis) {
        my ($t, $body) = (lc $1, $2);
        while ($body =~ /(?:^|[,\n(])\s*[`"]?(\w+)[`"]?[^,]*?\b(?:PRIMARY\s+KEY|UNIQUE)\b/gi) { $lead{$t}{lc $1} = 1 }
        while ($body =~ /PRIMARY\s+KEY\s*\(\s*[`"]?(\w+)/gi) { $lead{$t}{lc $1} = 1 }
      }
    }
    for my $f (@files) {
      my $s = $src{$f};
      next unless defined $s && $s =~ /\bSELECT\b/i;
      while ($s =~ /\bSELECT\b(.*?)(?=[`";]|\z)/gis) {
        my ($body, $base) = ($1, $-[1]);
        next unless $body =~ /\bFROM\s+[`"]?(\w+)[`"]?/i;
        my $t = lc $1;
        next unless $body =~ /\b(?:WHERE|AND)\b/i;
        my $ambiguous = ($body =~ /\bJOIN\b/i || $body =~ /\bFROM\s+[`"]?\w+[`"]?\s*,/i);
        while ($body =~ /\b(?:WHERE|AND)\s+(?:[`"]?\w+[`"]?\.)?[`"]?(\w+)[`"]?\s*=\s*(?:\?|\$\d+|:\w+|\@\w+)/gi) {
          my $col = lc $1;
          my $line = 1 + (substr($s, 0, $base + $-[0]) =~ tr/\n//);
          if ($ambiguous) {
            print "NOT VERIFIED DB-004: $f:$line: $col = ? is in a statement with a JOIN or several tables, so its table cannot be resolved (M-85)\n";
            next;
          }
          next if $lead{$t}{$col};
          print "FAIL DB-004: $f:$line: $col = ? on table $t has no index whose leading column is $col (M-85)\n";
        }
      }
    }
  ' "$@")
}
```

A `FAIL` names the predicate, its table, and its line. The compliant twin adds an index
whose first column is the predicate's column, through a migration; the same statement
then passes. Verified by `V2.83` on a fixture with the external audit's violation.

### AUTH-002 — A permission grant the approved requirement does not name is a human gate

M-86: a change that grants, widens, or adds a permission, role, scope, or access rule
that the approved requirement does not name is a human decision gate. The external
audit found a `viewer` role granted read access to payment methods, which exposed brand,
last four digits, and expiry to a role that had no payment access, and the requirement
did not ask for it. The detector lists each permission the change set adds in an
authorization file. The reviewer decides whether the requirement names it; an unnamed
grant is not merged until the user approves it explicitly (AGENTS.md, Human Decision
Gates).

**Scope.** Files whose path names an authorization concept (`auth`, `permission`,
`polic`, `role`, `scope`, `acl`, `rbac`) and the lines the change set adds. The detector
matches a permission verb inside a list or an object literal, such as `["read"]`. It
does not decide whether the grant is needed: that is the human gate.

```sh
# $1 = base revision; $2... = changed files, relative to the repository root.
authorization_grant_check() {
  base="$1"; shift
  for f in "$@"; do
    case "$f" in *auth*|*permission*|*polic*|*role*|*scope*|*acl*|*rbac*) ;; *) continue ;; esac
    git diff -U0 "$base" -- "$f" | awk -v file="$f" '
      /^@@/ { split($3, r, ","); n = substr(r[1], 2) + 0; next }
      /^\+\+\+/ { next }
      /^-/ { next }
      /^\+/ {
        if (substr($0, 2) ~ /\[[^]]*(read|write|create|update|delete|manage|admin|view|\*)/)
          print "FAIL AUTH-002: " file ":" n ": an added permission grant; the approved requirement must name it, or the user approves it (M-86)"
        n++; next
      }
      { n++ }'
  done
}
```

The compliant twin keeps the grant set of the base revision and adds no permission; the
check then prints nothing. The `FAIL` is the human gate: the report shows the grant, the
exposed data, and the requirement that does or does not name it, and the change waits for
the user's decision.

### SCOPE-001 — Every changed file is in the approved scope manifest

M-71: the change set is compared with the scope manifest the user approved. This
check runs on the change set, not on the project, because the approved scope is a
property of one task. A changed file the manifest does not name is a `FAIL`,
which catches the formatter rewrite of an unrelated file. The check reads file
names; a hunk that changes an unrelated line of an approved file is judged by
review under Dimension 14 of `references/audit-pass-criteria.md`.

Two defects of an earlier version are closed here (M-90, M-91). The change set
includes **untracked** files, so a new migration, script, or source file that the
manifest does not name is a `FAIL`; `git diff --name-only HEAD` alone misses it. And
a change set with **no recorded manifest** is a `FAIL`, never a silent pass: the
manifest is recorded at alignment before the first file write. An analysis-only
request records an empty manifest, so any changed file is a `FAIL` (M-91).

```sh
# $1 = repository root; $2 = scope manifest file, one approved path per line.
scope_file_check() {
  root="$1"; manifest="$2"
  changed="$( { git -C "$root" diff --name-only HEAD; git -C "$root" ls-files --others --exclude-standard; } 2>/dev/null | sed '/^$/d' | sort -u)"
  [ -n "$changed" ] || return 0
  if [ ! -f "$manifest" ]; then
    echo "FAIL SCOPE-001: $(printf '%s\n' "$changed" | head -1):1: the change set exists and no approved scope manifest was recorded at alignment (M-90)"
    return 0
  fi
  printf '%s\n' "$changed" | while IFS= read -r f; do
    grep -qxF -- "$f" "$manifest" || echo "FAIL SCOPE-001: $f:1: changed but not named in the approved scope manifest (M-71, M-90)"
  done
}
```

### SCOPE-002 — A changed value of existing behavior is named in the scope manifest

M-79: a changed value of existing behavior (a timeout, a limit, a retry budget, a default,
a pool size, or a threshold) is outside the approved scope unless
the approved requirement names it. `SCOPE-001` checks files; it does not see a changed
number inside an approved file, so a timeout that moves from 5000 to 500 passes it. This
detector reads the diff of the change set: a removed line whose numbers do not all
reappear in the added lines of the same file is a **candidate** changed value, reported
with its file and base-revision line. A candidate is a `FAIL` unless the manifest names
the file and the old value on one line (`<path> <old value> => <new value>`). It is a
candidate, not a verdict: a renamed identifier or a reformatted number also surfaces, and
the reviewer confirms each one. The reviewer never clears a candidate by reading the
diff alone; the manifest line is the evidence.

```sh
# $1 = repository root; $2 = scope manifest file; $3 = base revision (default HEAD).
changed_value_check() {
  root="$1"; manifest="$2"; base="${3:-HEAD}"
  git -C "$root" diff -U0 "$base" -- . 2>/dev/null | awk '
    /^\+\+\+ /{ f = substr($0, 7); sub(/^b\//, "", f); next }
    /^--- /{ next }
    /^@@/{ split($2, a, ","); oldln = substr(a[1], 2) + 0; next }
    /^\+/{ n = split(substr($0, 2), p, /[^0-9]+/); for (i = 1; i <= n; i++) if (p[i] != "") added[f SUBSEP p[i]] = 1; next }
    /^-/{
      n = split(substr($0, 2), p, /[^0-9]+/)
      for (i = 1; i <= n; i++) if (p[i] != "" && !((f SUBSEP p[i]) in added)) {
        print f ":" oldln ": " p[i]
        break
      }
      oldln++; next
    }' |
  while IFS= read -r cand; do
    file="${cand%%:*}"; rest="${cand#*:}"; line="${rest%%:*}"; num="${rest##*: }"
    if grep -qE "^$file .*(^|[^0-9])$num([^0-9]|$)" "$manifest" 2>/dev/null; then
      continue
    fi
    echo "FAIL SCOPE-002: $file:$line: candidate changed value $num is not named in the scope manifest (M-79)"
  done
}
```

### APPROVAL-001 — A claimed human approval quotes the user's message

M-92: an approval that a report, decision record, checkpoint, or proposal attributes to
the user is kept only when the verbatim message that gave it is quoted, with its
conversation turn. The check is lexical: it finds each approval claim and looks for a
quoted span of at least eight characters and a turn reference within two lines of the
claim. A claim without both is `NOT VERIFIED`, and the delivery gate that depends on it is
blocked. The reviewer judges whether the quoted message is the approval; the check only
requires that it is quoted. M-111: the whole-project sweep runs `APPROVAL-001` on every
Markdown document, not only on the decision records, so a report, note, or checkpoint that
claims an approval is checked wherever it is written.

```sh
# $1 = repository root; $2... = report, decision-record, checkpoint, or proposal files, relative to it.
approval_citation_check() {
  root="$1"; shift
  for f in "$@"; do
    [ -f "$root/$f" ] || { echo "NOT VERIFIED APPROVAL-001: $f: file not found"; continue; }
    awk -v f="$f" '
      { n++; L[n] = $0 }
      END {
        for (i = 1; i <= n; i++) {
          if (tolower(L[i]) !~ /aprob|approv|sign-?off|dueno|dueño|autoriz|authoriz|permis|consent|conformidad|acept|accept|confirm[oó]|confirmed by/) continue
          win = ""
          for (j = i - 2; j <= i + 2; j++) if (j >= 1 && j <= n) win = win " " L[j]
          q = (win ~ /"[^"]{8,}"/ || win ~ /`[^`]{8,}`/)
          t = (tolower(win) ~ /turn|turno|mensaje|message|dijo|said/)
          if (!(q && t))
            print "NOT VERIFIED APPROVAL-001: " f ":" i ": an approval is claimed with no quoted user message and turn; keep it only with the verbatim quote (M-92)"
        }
      }' "$root/$f"
  done
}
```

### TEARDOWN-001 — A required step recorded as NOT DONE blocks the commit proposal

M-93: a teardown, reset, cleanup, or rollback step that the checkpoint records as
`NOT DONE` is unresolved until the user resolves it or accepts it in writing. An open step
blocks the commit proposal until the user resolves it or accepts it in writing
(`workflows/commit.md` Phase 1). The check reads
the task's checkpoint in the user-level handoff directory, outside the repository, and
never the repository itself.

```sh
# $1... = checkpoint files of the task, absolute paths in the handoff directory.
teardown_gate_check() {
  for f in "$@"; do
    [ -f "$f" ] || { echo "NOT VERIFIED TEARDOWN-001: $f: checkpoint not found; the teardown state is unknown"; continue; }
    # M-89: a teardown, reset, cleanup, or rollback step that targets a path outside the target
    # project is a human gate. It passes only when the checkpoint records the authorization.
    grep -niE 'teardown|reset|cleanup|rollback' "$f" | grep -E '\.\./|~/|/Users/|/home/|/private/' |
      grep -viE 'authorized in the current conversation|accepted in writing|resolved by user' | while IFS=: read -r n rest; do
      echo "FAIL TEARDOWN-001: $f:$n: a teardown step targets a path outside the target project with no recorded authorization (M-89, M-93)"
    done
    grep -nE 'NOT[ _]DONE' "$f" | grep -viE 'accepted in writing|resolved by user' | while IFS=: read -r n rest; do
      echo "FAIL TEARDOWN-001: $f:$n: a required step is NOT DONE and unresolved; the proposal is blocked until the user resolves it or accepts it in writing (M-93)"
    done
  done
}
```

### INV-001 — An inventory is the enumeration of the target

M-72: an inventory in a report, gate, or analysis is produced by an enumeration of
the target in the same session. The detector compares two files: the **enumeration**
(one name per line, written by the command the report cites) and the **claim**
(one name per line, extracted from the report). A claimed name the enumeration does
not return is a `FAIL` with its line in the claim: it is a name from memory or from an
earlier draft. An enumerated item the claim neither lists nor excludes is a `FAIL`: the
inventory is incomplete. An exclusion is written in the claim as `# excluded <name>
<reason>`, and it is the only way an enumerated item may be left out. The count is
the number of enumerated items; the report states its definition next to it.

```sh
# $1 = enumeration file, one name per line; $2 = claim file, one name per line,
# plus `# excluded <name> <reason>` lines.
inventory_check() {
  enum="$1"; claim="$2"
  n=0
  while IFS= read -r c; do
    n=$((n + 1))
    case "$c" in ''|'#'*) continue ;; esac
    grep -qxF -- "$c" "$enum" || echo "FAIL INV-001: $claim:$n: $c is claimed and not returned by the enumeration (M-72)"
  done < "$claim"
  m=0
  while IFS= read -r e; do
    m=$((m + 1))
    [ -n "$e" ] || continue
    grep -qxF -- "$e" "$claim" && continue
    grep -qE "^# excluded $e( |\$)" "$claim" && continue
    echo "FAIL INV-001: $enum:$m: $e is enumerated by the target and neither listed nor excluded with a reason (M-72)"
  done < "$enum"
}
```

### STATE-001 — A read-only gate leaves the target state unchanged

M-73: a read-only gate records the state of the target repository before and after its
run. The snapshot holds HEAD, the branch, a checksum of `git status --porcelain`, and the
latest reflog entry. A difference between the snapshots is not attributed to anyone: the
detector prints the change and the last reflog entries with their dates, and the result is
`NOT VERIFIED` until the user attributes the change or it is reverted with authorization.
A snapshot that does not change is clean. The snapshot file is written **outside** the
target repository: a file the snapshot creates inside it is an untracked change that the
snapshot itself records, so the gate would report its own output as an anomaly.

```sh
# $1 = repository root. Prints the state snapshot to stdout.
repo_state_snapshot() {
  root="$1"
  echo "HEAD $(git -C "$root" rev-parse HEAD 2>/dev/null)"
  echo "BRANCH $(git -C "$root" symbolic-ref --short HEAD 2>/dev/null)"
  echo "STATUS $(git -C "$root" status --porcelain 2>/dev/null | cksum)"
  echo "REFLOG $(git -C "$root" reflog -n 1 --format='%H %gs' 2>/dev/null)"
}

# $1 = repository root; $2 = snapshot file written before the gate.
state_check() {
  root="$1"; before="$2"
  repo_state_snapshot "$root" > "$before.after" || return 1
  if cmp -s "$before" "$before.after"; then
    rm -f "$before.after"; return 0
  fi
  echo "NOT VERIFIED STATE-001: $root:1: the target state changed during a read-only gate; unattributed anomaly, the author is not named (M-73)"
  diff "$before" "$before.after" | sed 's/^/  /'
  git -C "$root" reflog -n 3 --format='  reflog %gd %gs (%ci)' 2>/dev/null
  rm -f "$before.after"
  return 1
}
```

### A11Y-001 — A live region announced on Android only

Audit Dimension 9 and `skills/accessibility/SKILL.md`. On React Native,
`accessibilityLiveRegion` is Android-only. A file that sets it, in a project that
never calls `AccessibilityInfo.announceForAccessibility`, has no iOS path, so the
platform verdict is `NOT VERIFIED`, never `PASS`.

```sh
# $1 = repository root; $2... = changed or tracked source files, relative to it.
a11y_live_region_check() {
  root="$1"; shift
  ios_path="$(cd "$root" && grep -rl "announceForAccessibility" \
    --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' \
    --exclude-dir=node_modules --exclude-dir=dist --exclude-dir=build . 2>/dev/null | head -n 1)"
  [ -n "$ios_path" ] && return 0
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*) continue ;; esac
    grep -n "accessibilityLiveRegion" "$root/$f" 2>/dev/null |
      sed "s#^#NOT VERIFIED A11Y-001: $f:#; s#\$#  (live region is Android-only and the project has no iOS announcement: Dimension 9, NOT VERIFIED for iOS)#"
  done
}
```

### DIP-002 — A platform side effect imported by a logic module

Audit Dimension 1 and AGENTS.md (the composition root). A hook, a service, a
use case, or a domain module that imports a platform capability with a side
effect from the platform module — `Linking`, `PermissionsAndroid`, `Alert`,
`AccessibilityInfo`, `Vibration`, `Share`, or `NativeModules` from `react-native`,
or the equivalent of an `expo-*` module — depends on the platform directly and
bypasses its port. The composition root (`app`, `main`, `index`) and component
files that only render primitives are exempt. A multi-line import cannot be read
on one line and is reported as `NOT VERIFIED`, never passed.

```sh
# $1 = repository root; $2... = changed or tracked source files, relative to it.
platform_side_effect_check() {
  root="$1"; shift
  side='Linking|PermissionsAndroid|Alert|AccessibilityInfo|Vibration|Share|NativeModules'
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*) continue ;; esac
    case "$f" in */components/*|*/screens/*|*/app/*|*/app.tsx|*/App.tsx|*/main.*|*/index.*) continue ;; esac
    case "$f" in */hooks/*|*/use-*|*/services/*|*/application/*|*/domain/*|*/usecases/*|*.service.*) ;; *) continue ;; esac
    grep -nE "^import .*\b($side)\b.*from ['\"](react-native|expo-[a-z-]+)['\"]" "$root/$f" 2>/dev/null |
      sed "s#^#FAIL DIP-002: $f:#; s#\$#  (platform side effect imported by a logic module; reach it through a port, Dimension 1)#"
    grep -nE "^import \{[^}]*$" "$root/$f" 2>/dev/null |
      sed "s#^#NOT VERIFIED DIP-002: $f:#; s#\$#  (multi-line import: the names cannot be read on one line)#"
  done
}
```

### DIP-001 — A concrete module is imported only by the composition root

M-96: AGENTS.md makes the composition root the only exception to the dependency rule.
A non-composition module (a service, a use case, an adapter, a hook, a domain module)
that imports a concrete module by a relative or aliased path depends on the
implementation and bypasses its port. `DIP-002` covers the platform side effects of
React Native and Expo; this check covers every other concrete module of the project.

A **concrete module** is recognized by its path or its file name: a folder named
`infrastructure/`, `adapters/`, `adapter/`, `persistence/`, `storage/`, `policies/`,
or `policy/`, or a file whose name ends in `.adapter`, `-adapter`, `-store`, `.store`,
`-policy`, or `.policy`. A project that names its concrete modules differently declares
its pattern in the decision record and the reviewer confirms each match. The
**composition root** is exempt: `main.*`, `index.*`, `server.*`, `app.*`, `cli.*`,
`bin.*`, `container.*`, `composition-root.*`, and any file under `composition/`, `bin/`,
`cli/`, `app/`, or `main/`. Tests and test support are exempt, and so is a type-only
import (`import type`), because a contract is not a behavior.

The check reads each static import and each re-export. A multi-line import is read
through its closing line (`} from '…'`), so a concrete specifier cannot hide behind a
line break. A type-only import that spans several lines is a candidate for the reviewer.

```sh
# $1 = repository root; $2 = source file, relative to it. Prints FAIL DIP-001 for a
# non-composition module that imports a concrete module.
dip_concrete_check() {
  root="$1"; f="$2"
  case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) return 0 ;; esac
  case "$f" in *.test.*|*.spec.*|*.d.ts|*/__tests__/*|*/tests/*|*/test/*|*/testing/*) return 0 ;; esac
  case "$f" in */bin/*|*/cli/*|*/composition/*|*/app/*|*/main/*) return 0 ;; esac
  case "$(basename "$f")" in main.*|index.*|server.*|app.*|cli.*|bin.*|container.*|composition-root.*|*.config.*) return 0 ;; esac
  [ -f "$root/$f" ] || return 0
  grep -nE "^((import|export) [^;]*|\}) from ['\"](\.{1,2}/|@/)[^'\"]*['\"]" "$root/$f" 2>/dev/null |
    grep -vE '^[0-9]+:import type ' |
    grep -E "(infrastructure|adapters?|persistence|storage|policies|policy)/|[-.](adapter|store|policy)(\.[a-z]+)?['\"]" |
    sed "s#^#FAIL DIP-001: $f:#; s#\$#  (concrete module imported by a non-composition file; receive it through a port and wire it in the composition root, M-96)#"
}
```

### ARTIFACT-001 — A cited artifact resolves inside the repository

M-97: a document that cites an artifact (a benchmark result, a query plan, a log, a
screenshot) is kept only when the artifact is in the repository. M-110: a cited path stays
inside the repository and exists. A path that climbs with `../`, starts at `~/`, or is
absolute (`/Users/`, `/home/`, `/private/`, `/var/`) is a `FAIL` wherever it appears, and a
cited directory that is not in the repository is `NOT VERIFIED`, because a sibling layout
named by a convention is a claim about the layout, not a citation. The check reads each
Markdown file and finds the paths it cites: a path with a directory and an artifact
extension, or a file name in backticks with an artifact extension. A cited path that
points outside the repository (an absolute path, or one that climbs with `..`) is a
`FAIL`. A cited path that is not in the repository is `NOT VERIFIED`, because the claim
that cites it cannot be checked, and an unverified claim blocks delivery (M-68). A line
that holds a URL is not read, so a web address is never mistaken for a path.

```sh
# $1 = repository root; $2... = documents, relative to it.
artifact_path_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.md) ;; *) continue ;; esac
    [ -f "$root/$f" ] || continue
    awk '
      /https?:\/\// { next }
      {
        s = $0
        while (match(s, /`[A-Za-z0-9_.-]+\.(json|txt|csv|png|jpe?g|pdf|log|svg|html|xml|ya?ml)`|[A-Za-z0-9_.\/-]*\/[A-Za-z0-9_.\/-]+\.(json|txt|csv|png|jpe?g|pdf|log|svg|html|xml|ya?ml)/)) {
          tok = substr(s, RSTART, RLENGTH); gsub(/`/, "", tok)
          print NR "\tART\t" tok
          s = substr(s, RSTART + RLENGTH)
        }
        s = $0
        while (match(s, /\.\.\/[A-Za-z0-9_.\/@-]*|~\/[A-Za-z0-9_.\/@-]*|\/(Users|home|private|var|opt|etc|tmp|Volumes)\/[A-Za-z0-9_.\/@-]*/)) {
          tok = substr(s, RSTART, RLENGTH)
          if (tok !~ /\.(json|txt|csv|png|jpe?g|pdf|log|svg|html|xml|ya?ml)$/) print NR "\tESC\t" tok
          s = substr(s, RSTART + RLENGTH)
        }
        s = $0
        while (match(s, /`[A-Za-z0-9_.@-]+(\/[A-Za-z0-9_.@-]+)+\/?`/)) {
          tok = substr(s, RSTART, RLENGTH); gsub(/`/, "", tok)
          if (tok !~ /\.\./ && (tok ~ /\/$/ || tok !~ /\/[^\/]*\./)) print NR "\tDIR\t" tok
          s = substr(s, RSTART + RLENGTH)
        }
      }' "$root/$f" |
    while IFS="$(printf '\t')" read -r n k p; do
      case "$k" in
        ART)
          case "$p" in
            /*|*..*) echo "FAIL ARTIFACT-001: $f:$n: $p points outside the repository (M-97)" ;;
            *) [ -e "$root/$p" ] || echo "NOT VERIFIED ARTIFACT-001: $f:$n: $p is not in the repository, so the claim that cites it is not verified (M-97, M-68)" ;;
          esac ;;
        ESC) echo "FAIL ARTIFACT-001: $f:$n: $p points outside the repository: a climbing, home, or absolute path is never a repository citation (M-110)" ;;
        DIR) [ -e "$root/$p" ] || echo "NOT VERIFIED ARTIFACT-001: $f:$n: directory $p is not in the repository, so the claim that cites its location is not verified (M-110)" ;;
      esac
    done
  done
}
```

### PRECOND-001 — A declared initial state is verified before the scenario runs

M-108: a guide or scenario that declares an initial state (`Initial state:`, `Estado inicial:`,
`Precondition`) is verified against the target before the scenario runs. A precondition
that names a path is checked for existence; a precondition about CI is checked against the
tracked CI configuration. A precondition that the target does not satisfy is a `FAIL`:
the scenario then exercises a different, weaker case than the guide describes, and its
result is reported with that gap. A precondition with no mechanical check is `NOT VERIFIED`,
never assumed. The check reads the documents of the sweep and the target, never the guide's
own claim that the state holds.

```sh
# $1 = repository root. Prints the tracked CI configuration files, relative to the root.
ci_files_check() {
  root="$1"
  files=$( (git -C "$root" ls-files 2>/dev/null) || (cd "$root" && find . -type f | sed 's#^\./##') )
  printf '%s\n' "$files" | grep -E '(^|/)(\.github/workflows/.*\.ya?ml|\.gitlab-ci\.ya?ml|azure-pipelines\.ya?ml|Jenkinsfile|\.circleci/config\.ya?ml|bitbucket-pipelines\.yml|\.drone\.yml|\.travis\.yml|cloudbuild\.ya?ml)$' || true
}

# $1 = repository root; $2... = documents, relative to it.
precondition_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.md) ;; *) continue ;; esac
    [ -f "$root/$f" ] || continue
    awk '
      { L[NR] = $0 }
      END {
        for (i = 1; i <= NR; i++) {
          if (tolower(L[i]) !~ /initial state|estado inicial|precondition|precondición|precondiciones/) continue
          if (L[i] !~ /:[[:space:]]*\**$/) print i "\t" L[i]
          for (j = i + 1; j <= NR && L[j] ~ /^[[:space:]]*([-*]|[0-9]+\.)[[:space:]]/; j++) print j "\t" L[j]
        }
      }' "$root/$f" |
    while IFS="$(printf '\t')" read -r n text; do
      low=$(printf '%s' "$text" | tr 'A-Z' 'a-z')
      paths=$(printf '%s\n' "$text" | grep -oE '`[A-Za-z0-9_./@-]+`' | tr -d '`' | grep -E '[/.]' || true)
      checked=0
      if [ -n "$paths" ]; then
        checked=1
        printf '%s\n' "$paths" | while read -r p; do
          [ -e "$root/$p" ] || echo "FAIL PRECOND-001: $f:$n: precondition not met: $p is missing, so the scenario does not run on this target (M-108)"
        done
      fi
      case "$low" in
        *" ci "*|*" ci."*|*" ci,"*|*"ci file"*|*continuous\ integration*|*pipeline*)
          checked=1
          [ -n "$(ci_files_check "$root")" ] || echo "FAIL PRECOND-001: $f:$n: precondition not met: no CI configuration is tracked, so a scenario that starts from a CI file does not run here (M-108)" ;;
      esac
      [ "$checked" -eq 1 ] || echo "NOT VERIFIED PRECOND-001: $f:$n: the declared initial state has no mechanical check; verify it against the target before the scenario runs, and report the scenario NOT VERIFIED (M-108)"
    done
  done
}
```

### EVIDENCE-001 — A temporal or count claim cites its evidence

M-109: a report or note that claims a time window (`run window`, `inside the run`, `mtime`,
`ventana`) cites a timestamp or the command output that shows it, within two lines. A claim
of a time window with neither is `NOT VERIFIED`. A
report that counts a set (`17 files`, `18 lines`, `N tests`) cites the enumerated file or
the command that produces the count, and the count equals the non-empty lines of that file
when the count names the file's entries. A mismatch is never decided by the check: it is
`NOT VERIFIED` until the author states the definition that produces the count, because a
summary line in the file may account for the difference. Each finding names `file:line`.

```sh
# $1 = repository root; $2... = documents, relative to it.
evidence_claim_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.md) ;; *) continue ;; esac
    [ -f "$root/$f" ] || continue
    awk '
      { n++; L[n] = $0 }
      END {
        for (i = 1; i <= n; i++) {
          lo = tolower(L[i]); win = ""
          for (j = i - 2; j <= i + 2; j++) if (j >= 1 && j <= n) win = win " " L[j]
          if (lo ~ /run window|inside the run|within the run|time window|ventana|mtime/) {
            ev = (win ~ /[0-9][0-9]?:[0-9][0-9]/ || win ~ /[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]/ || win ~ /`[^`]*(stat|ls -|date|git |find )[^`]*`/)
            if (!ev) print "TEMP\t" i
          }
          rest = lo
          while (match(rest, /[0-9]+ (files|archivos|ficheros|rutas|paths|lines|líneas|entries|items|commits|tests|errors|errores|módulos|modulos|reglas|rules|casos|cases|hallazgos|findings|dimensiones|filas|funciones|elementos)/)) {
            cnt = substr(rest, RSTART, RLENGTH); nxt = RSTART + RLENGTH; sub(/ .*/, "", cnt)
            p = "-"
            if (match(L[i], /`[^`]+`/)) tok = substr(L[i], RSTART + 1, RLENGTH - 2)
            else if (match(win, /`[^`]+`/)) tok = substr(win, RSTART + 1, RLENGTH - 2)
            else tok = ""
            if (tok ~ / /) p = "CMD"
            else if (tok ~ /^[A-Za-z0-9_.\/@-]+$/) p = tok
            print "CNT\t" i "\t" cnt "\t" p
            rest = substr(rest, nxt)
          }
          if (lo ~ /(cubri|cubre|covered|covers|covera|checked by|formatted by|linted by)/ && lo ~ /prettier|eslint|ruff|checkstyle|sqlfluff|formatter|linter|formateador/) {
            ev = (win ~ /`[^`]*(--file-info|--list-different|--show-files|--debug-check|--print-config|--list-files)[^`]*`/)
            if (!ev) print "TOOL\t" i
          }
        }
      }' "$root/$f" |
    while IFS="$(printf '\t')" read -r tag n cnt p; do
      case "$tag" in
        TEMP) echo "NOT VERIFIED EVIDENCE-001: $f:$n: a temporal claim (run window, mtime) cites no timestamp or command output; cite the stat output and the window bounds (M-109)" ;;
        TOOL) echo "NOT VERIFIED EVIDENCE-001: $f:$n: a tool is said to cover files, but no output of its own file resolution is cited (for example prettier --file-info or ruff check --show-files); a file the tool ignores is not covered, so a count of covered files is not evidence of coverage (M-109)" ;;
        CNT)
          if [ "$p" = "CMD" ]; then
            :
          elif [ -z "$p" ] || [ "$p" = "-" ] || [ ! -f "$root/$p" ]; then
            echo "NOT VERIFIED EVIDENCE-001: $f:$n: the count $cnt cites no enumerated file in the repository; cite the list or the command that produces it (M-109)"
          else
            m=$(grep -c . "$root/$p")
            [ "$m" -eq "$cnt" ] || echo "NOT VERIFIED EVIDENCE-001: $f:$n: the count $cnt does not match the $m non-empty lines of $p; state the definition that produces the count (M-109)"
          fi ;;
      esac
    done
  done
}
```

### DELIVERY-001 — An approval is bound to the artifact digest and to a distinct approver

M-114. An approval that a delivery workflow records is bound to the artifact it authorizes: it names the digest of the artifact, the gate compares that digest with the promoted artifact, and the approver is not the actor that
dispatched the run. The detector reads the workflow YAML. A workflow with a named approval and no digest
comparison is a `FAIL`; an approver read from `github.actor` or `github.triggering_actor` is a `FAIL`. The
reviewers of a platform environment are not in the repository, so their presence is `NOT VERIFIED`, never
passed. Limit: the detector reads the statement and the comparison, not the platform settings.

```sh
# M-114: an approval is bound to the artifact digest and to an approver other than the dispatcher.
# $1 = repository root; $2... = workflow YAML files, relative to it.
approval_binding_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.yml|*.yaml) ;; *) continue ;; esac
    [ -f "$root/$f" ] || continue
    awk -v file="$f" '
      { L[NR] = $0 }
      END {
        for (i = 1; i <= NR; i++) {
          if (tolower(L[i]) ~ /approved_by|approver|approval_by|aprobad/) {
            ctx = ""; for (j = i; j <= i + 2 && j <= NR; j++) ctx = ctx " " L[j]
            if (tolower(ctx) ~ /github\.(actor|triggering_actor)/)
              printf "FAIL DELIVERY-001: %s:%d: the approver is the actor that dispatched the run (github.actor); the approver must be a different person, not the dispatcher (M-114)\n", file, i
          }
        }
        for (i = 1; i <= NR; i++) {
          lo = tolower(L[i])
          if (lo ~ /approv|aprob/) { if (!app) app = i }
          if (lo ~ /sha256|digest|checksum/) dig = 1
          if (lo ~ /sha256sum -c|digest.*(==|!=)|\[.*(sha|digest).*(=|!=)|cmp /) cmp = 1
          if (lo ~ /^[ \t]*environment:/ && !env) env = i
        }
        if (app && !(dig && cmp))
          printf "FAIL DELIVERY-001: %s:%d: an approval is named and the digest of the artifact is not compared with it; the approval is not bound to the artifact it authorizes (M-114)\n", file, app
        if (app && env)
          printf "NOT VERIFIED DELIVERY-001: %s:%d: the reviewers of the environment are configured on the platform and are not read from the repository; the separation of the approver is not verified here (M-114)\n", file, env
      }' "$root/$f"
  done
}
```

### DELIVERY-002 — Gates and deploy consume the built artifact, not a re-checkout of a dispatched ref

M-116. A delivery workflow that checks out a dispatched ref (`ref:` read from `inputs.` or
`github.event.inputs`) and then deploys or publishes in the same file runs its gates and its deploy on a
source tree that is not the artifact that was verified. The gates and the deploy consume the built artifact by
its digest. The detector reports the checkout with `file:line`.

```sh
# M-116: gates and deploy consume the built artifact by digest, never a re-checkout of a dispatched ref.
# $1 = repository root; $2... = workflow YAML files, relative to it.
artifact_ref_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.yml|*.yaml) ;; *) continue ;; esac
    [ -f "$root/$f" ] || continue
    awk -v file="$f" '
      { lo = tolower($0)
        if (lo ~ /ref:.*(inputs\.|github\.event\.inputs)/ && !rf) rf = NR
        if (lo ~ /deploy|publish|upload|kubectl|gh release|helm (upgrade|install)/ && !dp) dp = NR
      }
      END {
        if (rf && dp)
          printf "FAIL DELIVERY-002: %s:%d: a dispatched ref is checked out and a deploy follows in the same file; the gates and the deploy must consume the built artifact by its digest (M-116)\n", file, rf
      }' "$root/$f"
  done
}
```

### IMPORT-001 — A module that a test can import runs no effect at import

M-117. A source module performs no effect when it is imported: a column-zero call such as `main().catch(...)` or
a column-zero `process.` assignment runs at import, so a test that imports the module runs the program. The
effect moves into a function that the entry point calls. A module with a `require.main === module` or
`import.meta.main` guard is `NOT VERIFIED` until the reviewer confirms the guard covers the call. Limit: a
module-scope assignment whose right side is a call (`const x = f()`) is not read; the reviewer checks it.

```sh
# M-117: a module that a test can import runs no effect at import. A column-zero call or a
# process assignment is an effect; a declaration, an import, or an export is not.
# $1 = repository root; $2... = source files, relative to it.
import_effect_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*.d.ts|*/test/*|*/tests/*|*__tests__*|*/testing/*) continue ;; esac
    [ -f "$root/$f" ] || continue
    awk -v file="$f" '
      /^(export|import|const|let|var|function|async|class|type|interface|enum|return|if|for|while|switch|try|describe|it|test|beforeAll|beforeEach|afterAll|afterEach)[ (]/ { next }
      /^[ \t]*$/ || /^[ \t]*(\/\/|\/\*|\*)/ || /^[})\]]/ { next }
      /require\.main *=== *module|import\.meta\.main|import\.meta\.url *===/ { guard = NR; next }
      /^[A-Za-z_$][A-Za-z0-9_$.]*\(/ || /^process\.[A-Za-z.]+ *=[^=]/ { if (!first) first = NR; n++ }
      END {
        if (n && guard)
          printf "NOT VERIFIED IMPORT-001: %s:%d: a module-scope call sits beside a main guard at line %d; the reviewer confirms the guard covers it (M-117)\n", file, first, guard
        else if (n)
          printf "FAIL IMPORT-001: %s:%d: a module-scope call runs an effect when the module is imported; move it into a function that the entry point calls (M-117)\n", file, first
      }' "$root/$f"
  done
}
```

### EXEC-001 — A claim that an agent or a prompt ran cites the client invocation

M-115. A report that says an agent or a prompt was executed cites the client invocation that ran it: a session,
a client command, or its transcript. A direct call to the program (`node`, `npx`, `bun`) proves the program, not
the agent behavior, so a claim that cites only a direct call, or names the direct call as the way it ran, is
`NOT VERIFIED` (M-115). The detector reads the report text within two lines of the claim.

```sh
# M-115: a claim that an agent or a prompt ran cites the client invocation, not a direct CLI call.
# $1 = repository root; $2... = report documents, relative to it.
exec_claim_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.md) ;; *) continue ;; esac
    [ -f "$root/$f" ] || continue
    awk -v file="$f" '
      { L[NR] = $0 }
      END {
        for (i = 1; i <= NR; i++) {
          lo = tolower(L[i])
          # M-115: the claim words include the behavior of an agent (conducta, comportamiento, behavior, "se respalda"),
          # not only "ran"; a report that backs the behavior of an agent with direct commands is the same claim (F-4).
          if (lo ~ /(prompt|agent|agente)[^.;]{0,40}(ejecut[^ ]*|executed|ran|conducta|comportamiento|behavio[u]?r|respald[^ ]*|backed by|supported by)( |$)|(ejecut[^ ]*|executed|ran)[^.;]{0,20}(prompt|agent|agente)|(conducta|comportamiento|behavio[u]?r)[^.;]{0,40}(prompt|agent|agente)|ran through the agent/) {
            ctx = ""; for (j = i - 2; j <= i + 2; j++) if (j >= 1 && j <= NR) ctx = ctx " " tolower(L[j])
            agent = (ctx ~ /opencode run|ses_|session|sesi.n|transcript|transcripci.n|client session|slash command|\/[a-z][a-z-]+ /)
            direct = (ctx ~ /directamente|directly|direct cli|invocaci.n directa|direct invocation|comandos directos|direct commands|`node|`npx|`bun|`tsx/)
            if (!agent || direct)
              printf "NOT VERIFIED EXEC-001: %s:%d: an agent or prompt execution is claimed with no client invocation (a session, a client command, or its transcript); a direct CLI call proves the program, not the agent behavior (M-115)\n", file, i
          }
        }
      }' "$root/$f"
  done
}
```

### CI-001, NODE-001, DEP-RANGE-001 — Baseline health findings

M-112: the health probe reports three baseline conditions as findings that never block an
unrelated task, and never as passing. `CI-001`: no CI configuration is tracked, so the
delivery gates do not run automatically. `NODE-001`: the Node major declared in `.nvmrc`,
`.node-version`, `package.json` `engines`, and CI `node-version` differs across the project.
`DEP-RANGE-001`: a runtime dependency declares a caret or tilde range. A missing declaration
of Node is `NOT VERIFIED`, never clean.

```sh
# $1 = repository root. CI-001: a tracked CI configuration is required to report the gates as automated.
ci_presence_check() {
  root="$1"; list=$(ci_files_check "$root")
  if [ -n "$list" ]; then echo "PASS CI-001: CI configuration tracked: $(printf '%s' "$list" | tr '\n' ' ')"
  else echo "FINDING CI-001: no CI configuration is tracked (.github/workflows, .gitlab-ci.yml, azure-pipelines.yml, Jenkinsfile, .circleci, bitbucket-pipelines.yml); the delivery gates are not run automatically (M-112)"; fi
}

# $1 = repository root. NODE-001: the declared Node major must agree across the project.
node_drift_check() {
  root="$1"
  rows=$( {
    [ -f "$root/.nvmrc" ] && printf '%s\t%s\n' ".nvmrc" "$(head -1 "$root/.nvmrc")"
    [ -f "$root/.node-version" ] && printf '%s\t%s\n' ".node-version" "$(head -1 "$root/.node-version")"
    [ -f "$root/package.json" ] && awk -F'"' '/"node"[[:space:]]*:/ { print "package.json\t" $4 }' "$root/package.json"
    for c in $(ci_files_check "$root"); do
      awk -v c="$c" '/node-version/ { v = $0; sub(/.*node-version:[[:space:]]*/, "", v); gsub(/[" ]/, "", v); print c "\t" v }' "$root/$c"
    done
  } )
  [ -n "$rows" ] || { echo "NOT VERIFIED NODE-001: no Node version is declared in .nvmrc, .node-version, package.json engines, or CI, so drift is not verified (M-112)"; return 0; }
  majors=$(printf '%s\n' "$rows" | awk -F'\t' '{ v = $2; sub(/^[^0-9]*/, "", v); sub(/[^0-9].*/, "", v); if (v != "") print v }' | sort -u)
  if [ "$(printf '%s\n' "$majors" | grep -c .)" -gt 1 ]; then
    printf '%s\n' "$rows" | awk -F'\t' '{ print "FINDING NODE-001: " $1 " declares Node " $2 "; the declared majors differ across the project (M-112)" }'
  else
    echo "PASS NODE-001: one Node major declared across the project: $(printf '%s' "$majors" | tr '\n' ' ')"
    # M-80 / task-to-code step 11: the gates run on the pinned major. A runtime that differs from the
    # pin means the result holds only for the active major, so it is NOT VERIFIED for the pinned one.
    pinned=$(printf '%s\n' "$majors" | head -1)
    if ! command -v node >/dev/null 2>&1; then
      echo "NOT VERIFIED NODE-001: no node runtime is available, so the active major is not compared with the pinned Node $pinned (M-80)"
    else
      active=$(node -p 'process.versions.node.split(".")[0]' 2>/dev/null)
      if [ -n "$active" ] && [ "$active" != "$pinned" ]; then
        echo "NOT VERIFIED NODE-001: the gates ran on Node $active and the project pins Node $pinned; the result holds only for Node $active, so it is NOT VERIFIED for the pinned major (M-80)"
      fi
    fi
  fi
}

# $1 = repository root. DEP-RANGE-001: runtime dependencies pinned to an exact version or a recorded range.
dep_range_check() {
  root="$1"
  files=$( (git -C "$root" ls-files 2>/dev/null) || (cd "$root" && find . -type f | sed 's#^\./##') )
  printf '%s\n' "$files" | grep -E '(^|/)package\.json$' | grep -v node_modules | while read -r p; do
    awk -v f="$p" '
      /"dependencies"[[:space:]]*:[[:space:]]*\{/ { inb = 1; next }
      inb && /^[[:space:]]*\}/ { inb = 0 }
      inb && /"[^"]+"[[:space:]]*:[[:space:]]*"[~^]/ { print "FINDING DEP-RANGE-001: " f ":" NR ": a caret or tilde range in runtime dependencies; pin the version or record why the range is accepted (M-112)"; hit++ }
      END { if (!hit) print "PASS DEP-RANGE-001: " f " has no caret or tilde range in runtime dependencies" }
    ' "$root/$p"
  done
}
```

### BEHAVIOR-001 — A removed or renamed test case is an approved behavior change

M-94: a test case that existed in the base revision and is absent from the working copy
was removed or renamed, and that is a change of behavior, not of form. The check
compares the titles of `describe`, `test`, and `it` in each test file the change set
modifies against the base revision. A title that disappeared is a `FAIL` unless the
approved scope manifest names it on a line of its own, in the form
`behavior: <exact title>`. The manifest is the one recorded at alignment (M-90), so the
removal is a decision the user approved, not a by-product of a rewrite.

**Scope limit.** The check reads titles. A test whose title is unchanged and whose
expectation changed is not seen here; that is judged under Dimension 7 by reading the
assertion, and it is reported as a finding until the reader confirms it.

```sh
# $1 = repository root; $2 = base revision; $3 = approved scope manifest; $4... = changed files, relative to root.
test_titles() {
  grep -oE "(^|[^A-Za-z0-9_.])(test|it|describe)\(['\"\`][^'\"\`]+" | sed -E "s/.*\((['\"\`])//" | sort -u
}

test_title_removal_check() {
  root="$1"; base="$2"; manifest="$3"; shift 3
  work="$(mktemp -d)" || return 1
  for f in "$@"; do
    case "$f" in *.test.*|*.spec.*) ;; *) continue ;; esac
    git -C "$root" cat-file -e "$base:$f" 2>/dev/null || continue
    git -C "$root" show "$base:$f" | test_titles > "$work/before"
    if [ -f "$root/$f" ]; then test_titles < "$root/$f" > "$work/after"; else : > "$work/after"; fi
    comm -23 "$work/before" "$work/after" | while IFS= read -r title; do
      grep -qxF -- "behavior: $title" "$manifest" 2>/dev/null && continue
      echo "FAIL BEHAVIOR-001: $f:1: test case \"$title\" was removed or renamed and the approved manifest does not name it as a behavior change (M-94)"
    done
  done
  rm -rf "$work"
}
```

### PREFLIGHT-001 — The repository state is recorded before alignment

M-95: before alignment the state of the target repository is recorded: the branch, the
HEAD, and `git status --porcelain`. A detached HEAD stops alignment, because a change set
has no branch to belong to, and the branch is asked for (`AGENTS.md`, Task Branch
Policy). A working tree with uncommitted paths stops alignment unless the task checkpoint
records them in a `working tree:` line as preserved, so no unrecorded user work is
carried into a change set. The check reads state only and never modifies the repository.

```sh
# $1 = repository root; $2 = task checkpoint (optional), absolute path in the handoff directory.
preflight_state_check() {
  root="$1"; checkpoint="${2:-}"
  git -C "$root" rev-parse --git-dir >/dev/null 2>&1 ||
    { echo "NOT VERIFIED PREFLIGHT-001: $root is not a git repository, so the state was not recorded"; return 0; }
  ref="$(git -C "$root" symbolic-ref --short -q HEAD 2>/dev/null)"
  [ -n "$ref" ] || echo "FAIL PREFLIGHT-001: $root: HEAD is detached; alignment stops and the branch is asked for (M-95)"
  dirty="$(git -C "$root" status --porcelain 2>/dev/null)"
  if [ -n "$dirty" ]; then
    n="$(printf '%s\n' "$dirty" | wc -l | tr -d ' ')"
    if [ -n "$checkpoint" ] && [ -f "$checkpoint" ] && grep -q '^working tree:' "$checkpoint"; then
      :
    else
      echo "FAIL PREFLIGHT-001: $root: $n uncommitted path(s) and no recorded state; alignment stops until the user decides to preserve or include them (M-95)"
    fi
  fi
}
```

### A11Y-002 — A live announcement with no Android path

Audit Dimension 9 and `skills/accessibility/SKILL.md`. An announcement made
through `AccessibilityInfo.announceForAccessibility` reaches iOS only. A change
that announces a state change and sets no `accessibilityLiveRegion` anywhere in
the change set has no Android path for it, so the platform verdict is
`NOT VERIFIED` for Android, never `PASS`. This detector reads the change set as a
whole: a live region on another changed file is an Android path.

```sh
# $1 = repository root; $2... = changed or tracked source files, relative to it.
a11y_android_path_check() {
  root="$1"; shift
  live=""
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*) continue ;; esac
    grep -q "accessibilityLiveRegion" "$root/$f" 2>/dev/null && live=1
  done
  [ -n "$live" ] && return 0
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*) continue ;; esac
    n="$(grep -n "announceForAccessibility" "$root/$f" 2>/dev/null | head -n 1 | cut -d: -f1)"
    [ -n "$n" ] && echo "NOT VERIFIED A11Y-002: $f:$n:  (announced on iOS only: no accessibilityLiveRegion in the change set; enumerate each status branch and set the live region on Android, Dimension 9)"
  done
}
```

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

### COMMIT-001 — Every proposed commit group is self-contained

M-05 and M-71: a commit group is proposed only when its files are reviewable and
buildable without the uncommitted files of another group. Each relative import of
a changed source file must resolve to a file in the same group or to a file tracked
at the base revision. Each file that a changed decision record cites by name must be
in the same group or at the base. A reference that resolves only to an uncommitted
file of another group is a `FAIL`: that group cannot be committed or reviewed alone.
The check runs once per proposed group from `workflows/commit.md` (Phase 11). It reads
the working tree and the base revision and never stages anything.

```sh
# $1 = repository root; $2 = base revision (HEAD); $3... = the files of ONE proposed commit group, relative to root.
commit_group_closure_check() {
  root="$1"; base="$2"; shift 2
  git -C "$root" rev-parse --verify -q "$base^{commit}" >/dev/null 2>&1 ||
    { echo "NOT VERIFIED COMMIT-001: base revision $base does not resolve, so the group closure is not checked"; return 0; }
  group="$(printf '%s\n' "$@")"
  basefiles="$(git -C "$root" ls-tree -r --name-only "$base")"
  for f in "$@"; do
    [ -f "$root/$f" ] || continue
    case "$f" in
      *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs)
        dir="$(dirname "$f")"
        grep -nE "(from|import)[[:space:]]*\(?[[:space:]]*['\"]\.\.?/" "$root/$f" 2>/dev/null |
        while IFS= read -r hit; do
          ln="${hit%%:*}"
          spec="$(printf '%s\n' "$hit" | sed -E "s/.*[\"'](\.[^\"']*)[\"'].*/\1/")"
          target="$(printf '%s/%s\n' "$dir" "$spec" | awk -F/ '{ n = 0; for (i = 1; i <= NF; i++) { if ($i == "" || $i == ".") continue; if ($i == "..") { if (n > 0) n--; continue } p[++n] = $i } s = ""; for (i = 1; i <= n; i++) s = s (i > 1 ? "/" : "") p[i]; print s }')"
          resolved=0
          for cand in "$target" "$target.ts" "$target.tsx" "$target.js" "$target.jsx" "$target/index.ts" "$target/index.tsx" "$target/index.js"; do
            if printf '%s\n' "$group" | grep -qxF -- "$cand" || printf '%s\n' "$basefiles" | grep -qxF -- "$cand"; then resolved=1; break; fi
          done
          [ "$resolved" = 1 ] || echo "FAIL COMMIT-001: $f:$ln: import $spec resolves neither in this commit group nor at $base (M-05, M-71)"
        done ;;
      *.md)
        grep -oE '[A-Za-z0-9_./-]+\.(ts|tsx|js|jsx|mjs|cjs|sql)' "$root/$f" 2>/dev/null | sort -u |
        while IFS= read -r tok; do
          name="$(basename "$tok")"
          printf '%s\n%s\n' "$group" "$basefiles" |
            awk -v b="$name" 'BEGIN { ok = 0 } { n = split($0, a, "/"); if (a[n] == b) ok = 1 } END { exit ok ? 0 : 1 }' ||
            echo "FAIL COMMIT-001: $f: cites $tok, which is neither in this commit group nor at $base (M-65)"
        done ;;
    esac
  done
}
```

### COMMIT-002 — Every commit a proposal carries is in the approved scope

M-87: a proposed commit or pull request carries only the commits of the approved scope.
A branch stacked on another task's commits carries them into `base..HEAD`; the external
audit found nine commits of other tasks in a branch that was meant to ship one change. A
commit in `base..HEAD` that touches a file the approved scope manifest does not name is a
`FAIL` that blocks the proposal until the user either rebases the branch onto the base or
accepts the commit in writing (`workflows/commit.md`, Phase 1).

```sh
# $1 = base revision; $2 = approved scope manifest (one approved path per line).
out_of_scope_commit_check() {
  base="$1"; manifest="$2"
  [ -f "$manifest" ] || { echo "NOT VERIFIED COMMIT-002: no approved scope manifest, so the commits of $base..HEAD were not attributed"; return 0; }
  git rev-list --reverse "$base"..HEAD | while IFS= read -r c; do
    git show --name-only --format= "$c" | sed '/^$/d' | while IFS= read -r p; do
      grep -qxF -- "$p" "$manifest" || {
        echo "FAIL COMMIT-002: $c $(git log -1 --format=%s "$c"): touches $p, which the approved manifest does not name (M-87)"
        break
      }
    done
  done
}
```

A branch whose commits touch only manifest paths prints nothing. A stacked branch prints
one `FAIL` per foreign commit, with the commit, its subject, and the first path outside the
manifest. The check runs in the commit proposal before Phase 13.

### BOUND-001 — A declared timeout is proven by a test that holds the dependency

M-54 and M-76: a declared bound (a timeout option, a busy or lock wait, a deadline)
is proven only by a test in which the dependency never resolves, under controlled
time, and whose terminal state is asserted. A test that makes the dependency return
a timeout result proves the dependency, not the bound. The detector covers declared
bounds, not only HTTP calls: a SQLite `busy_timeout`, a `BEGIN IMMEDIATE` wait, a
lock or statement timeout, and a broker or process deadline are declared bounds too.
For each changed source file that declares a bound, a test that imports
the module must hold the dependency so that it never settles, by a never-resolving
promise, a second connection that holds the lock, or an equivalent construct. The
detector identifies candidates; the reviewer confirms that the test asserts the
terminal state the user sees.

```sh
# $1 = repository root; $2... = changed or tracked source files; then `--`; then the test files, relative to root.
bound_test_check() {
  root="$1"; shift
  srcs=""; tests=""; mode=src
  for a in "$@"; do
    if [ "$a" = "--" ]; then mode=tests; continue; fi
    if [ "$mode" = src ]; then srcs="$srcs$a
"; else tests="$tests$a
"; fi
  done
  printf '%s' "$srcs" | while IFS= read -r f; do
    [ -n "$f" ] && [ -f "$root/$f" ] || continue
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*|*/testing/*) continue ;; esac
    grep -qE '(busy_timeout|timeoutMs|timeout[[:space:]]*[:=]|lock_timeout|statement_timeout|deadline[[:space:]]*[:=]|BEGIN IMMEDIATE|maxAttempts|max_attempts|retries[[:space:]]*[:=]|attempts[[:space:]]*[:=]|retryBudget|max_retries)' "$root/$f" || continue
    mod="$(basename "$f")"; mod="${mod%.*}"
    importers="$(printf '%s' "$tests" | while IFS= read -r t; do
      [ -n "$t" ] && [ -f "$root/$t" ] || continue
      grep -qE "^import .*from ['\"][^'\"]*$mod(\.[a-z]+)?['\"]" "$root/$t" && printf '%s\n' "$t"
    done)"
    if [ -z "$importers" ]; then
      echo "FAIL BOUND-001: $f:1: a bound is declared, and no test imports $mod to hold its dependency (M-54, M-76)"
      continue
    fi
    held="$(printf '%s\n' "$importers" | while IFS= read -r t; do
      grep -lE 'new Promise(<[^>]*>)?\(\( *\) *=>|never (resolves|settles)|BEGIN IMMEDIATE|holds? the (lock|connection)' "$root/$t" 2>/dev/null
    done)"
    [ -n "$held" ] || echo "FAIL BOUND-001: $f:1: no test that imports $mod holds the dependency so that it never resolves; the bound is not proven (M-54, M-76)"
  done
}
```

### COMMENT-001 — An ordering or return claim in a comment is checked against the code

M-65 and the comment-accuracy rule of Dimension 3: a comment that states the order
in which data is produced, the cursor or traversal that produces it, or the most or
least recent item is a claim about the code. The claim is checked against the
function it documents and against its return type. The detector lists the changed comments that carry an ordering or traversal claim, so that none is missed. Each
listed comment is `NOT VERIFIED` until the reviewer cites the code that decides it,
and a comment the code contradicts is a finding of the change, with both lines.

```sh
# $1 = repository root; $2... = changed or tracked TypeScript or JavaScript files, relative to root.
comment_claim_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    [ -f "$root/$f" ] || continue
    grep -nE '^[[:space:]]*(\*|//|/\*\*)' "$root/$f" 2>/dev/null |
    grep -iE 'order|ordered|sorted|sort|recent|newest|oldest|cursor|recorr|traverse|iterat|ascending|descending|orden|más reciente|mas reciente' |
    while IFS= read -r hit; do
      echo "NOT VERIFIED COMMENT-001: $f:${hit%%:*}: a comment states an ordering or traversal claim; the reviewer checks it against the code and the return type (M-65)"
    done
  done
}
```

### DEC-002 — A `file:line` in a decision record names the code it describes

M-78: a `file:line` that a record cites for a unit, a responsibility, or a consumer
must point to the code it describes. `DEC-001` checks that the citation exists in the
record; this check checks that it resolves in the repository. The cited file must
exist, the line must be inside it, and the identifier that the line names first in
backticks must appear within five lines of the cited line. A citation that points to
a different line of the same file is a `FAIL` with both the cited line and the line
where the identifier is found. A line that names no identifier is `NOT VERIFIED`,
because its citation cannot be checked.

**Scope.** Every line of a decision record or a contract document that carries a
`file:line` citation is checked, in a table row, a list item, or prose. A citation
outside a table, such as the one in the body of a decision note, is not exempt; the
external audit found a stale citation in exactly that place (R-136, R-145).

```sh
# $1 = repository root; $2... = decision records and contract documents, relative to root.
decision_citation_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.md) ;; *) continue ;; esac
    [ -f "$root/$f" ] || continue
    grep -nE '[A-Za-z0-9_./-]+\.[A-Za-z0-9]+:[0-9]+' "$root/$f" 2>/dev/null | while IFS= read -r hit; do
      rowno="${hit%%:*}"; row="${hit#*:}"
      sym="$(printf '%s\n' "$row" | grep -oE '`[A-Za-z_][A-Za-z0-9_]*`' | head -n 1 | tr -d '`')"
      printf '%s\n' "$row" | grep -oE '[A-Za-z0-9_./-]+\.[A-Za-z0-9]+:[0-9]+' | while IFS= read -r cite; do
        cited_file="${cite%:*}"; line="${cite##*:}"
        [ -f "$root/$cited_file" ] || { echo "FAIL DEC-002: $f:$rowno: cited file $cited_file does not exist (M-78)"; continue; }
        total="$(wc -l < "$root/$cited_file" | tr -d ' ')"
        if [ "$line" -lt 1 ] || [ "$line" -gt "$total" ]; then
          echo "FAIL DEC-002: $f:$rowno: $cite is beyond the end of the file ($total lines) (M-78)"; continue
        fi
        [ -n "$sym" ] || { echo "NOT VERIFIED DEC-002: $f:$rowno: $cite is in a row that names no identifier in backticks, so the line cannot be checked (M-78)"; continue; }
        start=$(( line > 5 ? line - 5 : 1 )); end=$(( line + 5 ))
        sed -n "${start},${end}p" "$root/$cited_file" | grep -qwF -- "$sym" ||
          echo "FAIL DEC-002: $f:$rowno: $cite does not show $sym within 5 lines of the cited line (M-78)"
      done
    done
  done
}
```

### DEL-001 — A destructive operation on a supplied path is confined to a run directory

M-103: code that deletes, overwrites, or replaces a supplied path operates only inside a
subdirectory the program creates for the run. It is the rule of `AGENTS.md` (Workspace Safety:
never destroy or replace user work). A recursive removal (`rmSync` with `recursive`, `rm -rf`, `shutil.rmtree`,
`os.RemoveAll`, `FileUtils.rm_rf`) whose target the program does not create itself
deletes whatever the user pointed at. The external audit found a `--scratch` option
that was used as the working directory and then removed whole, which also overwrote
the live data directory it overlapped. The detector reads each non-test source file.
A file that removes a path and reads a supplied value (`argv`, `parseArgs`,
`options.`, `opts.`, `flags.`, `args.`) and creates no per-run temporary directory
(`mkdtemp`, `mktemp`, `tempfile`, `MkdirTemp`, `tmpdir`) is a `FAIL` at the removal
line. A file that removes a path with no supplied value is `NOT VERIFIED`: the
reviewer confirms the program owns the path. The overlap guard (a supplied path that
equals or contains a data, backup, or manifest path is rejected before any side
effect) is not decided by text; it is verified by the behavioral fixture of the
owning project, so this detector never reports a guarded program as clean.

```sh
# $1 = repository root; $2... = source files, relative to it.
destructive_path_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs|*.py|*.go|*.rs|*.rb|*.php|*.java|*.kt|*.cs|*.sh) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*_test.*|*/__tests__/*|*/tests/*|*/test/*|*/test_*|*/testing/*) continue ;; esac
    awk -v file="$f" '
      /rmSync\(|rmdirSync\(|shutil\.rmtree|os\.RemoveAll|FileUtils\.rm_rf|remove_all\(|rm -rf|rm -r / { n++; at[n] = NR }
      /mkdtemp|mktemp|tempfile|MkdirTemp|tmpdir\(/ { tmp = 1 }
      /argv|parseArgs|options\.|opts\.|flags\.|args\./ { supplied = 1 }
      END {
        for (i = 1; i <= n; i++) {
          if (tmp) continue
          if (supplied)
            printf "FAIL DEL-001: %s:%d: a recursive removal of a path the program was given, with no per-run directory it created (M-103)\n", file, at[i]
          else
            printf "NOT VERIFIED DEL-001: %s:%d: a recursive removal; the reviewer confirms the program owns the path (M-103)\n", file, at[i]
        }
      }' "$root/$f"
  done
}
```

### PARSE-001 — A parsed external value is typed before it is dereferenced

M-104: a value a parser returns from external input is typed before it is dereferenced. The
external audit found `JSON.parse("null")` returning `null`, which the
code dereferenced outside the handler that mapped the error. A line that assigns the
result of `JSON.parse(` to a name is checked: within fifteen lines the name must meet
a type or null guard (`typeof X`, `X === null`, `Array.isArray(X)`, an `is...(X)`
validator, or the optional-access form `X?.`) before it is dereferenced (`X.`, `X[`).
A dereference that comes first is a `FAIL` at its line. A parse that goes straight
into a schema validator (`safeParse`, `.parse(` of a schema) is not a dereference
and is not reported. The detector is a candidate: the reviewer confirms that the
guard rejects with the input-error status, because the status mapping is not text.

```sh
# $1 = repository root; $2... = source files, relative to it.
parse_dereference_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*) continue ;; esac
    awk -v file="$f" '
      { L[NR] = $0 }
      END {
        for (i = 1; i <= NR; i++) {
          if (L[i] !~ /JSON\.parse\(/ || L[i] ~ /safeParse|\.parse\(.*\)\)|validate/) continue
          if (match(L[i], /(const|let|var)[ \t]+[A-Za-z_$][A-Za-z0-9_$]*[ \t]*(:[^=]*)?=[ \t]*JSON\.parse\(/) == 0) continue
          v = L[i]
          sub(/^[ \t]*(const|let|var)[ \t]+/, "", v); sub(/[^A-Za-z0-9_$].*$/, "", v)
          for (k = i + 1; k <= NR && k <= i + 15; k++) {
            if (L[k] ~ ("typeof " v "[^A-Za-z0-9_$]|" v " *(===|!==|==|!=) *null|null *(===|!==|==|!=) *" v "[^A-Za-z0-9_$]|Array\\.isArray\\(" v "\\)|is[A-Za-z]*\\(" v "\\)|" v "\\?\\.")) break
            if (L[k] ~ ("(^|[^A-Za-z0-9_$.])" v "[.\\[]")) {
              printf "FAIL PARSE-001: %s:%d: %s is dereferenced before the parsed value is typed (M-104)\n", file, k, v
              break
            }
          }
        }
      }' "$root/$f"
  done
}
```

### SRP-001 — A unit that combines several responsibilities is listed for its record

M-107 and M-78. A unit with more than one reason to change violates SRP even when
its record says `APPLIES`. The detector counts, for each non-test source file, the
kinds of responsibility the file contains: argument parsing, exit status, output,
file lifecycle, manifest or contract validation, and orchestration (an `await` or a
`.then` chain). A file with three or more kinds is listed with its kinds and the
line of the first one. The threshold is a candidate, not a verdict: the reviewer
confirms each listed unit, and a unit the reviewer accepts keeps its reason in the
decision record. `srp_record_check` then requires the decision record to name every
listed file, so an SRP row that covers only the folder that holds the unit is a
`FAIL`.

```sh
# $1 = repository root; $2... = source files, relative to it.
srp_unit_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*|*.d.ts) continue ;; esac
    awk -v file="$f" '
      function kind(name, re) { if (!(name in seen) && $0 ~ re) seen[name] = NR }
      {
        kind("argument parsing", "argv|parseArgs|takeValue|--[a-z][a-z-]*")
        kind("exit status", "process\\.exit|exitCode|EXIT_[A-Z]")
        kind("output", "console\\.(log|error|info|warn)|process\\.std(out|err)\\.write")
        kind("file lifecycle", "rmSync|mkdirSync|renameSync|copyFileSync|writeFileSync|unlinkSync|mkdtemp")
        kind("manifest or contract validation", "JSON\\.parse|\\.parse\\(|validate[A-Z][A-Za-z]*\\(|isRecord|safeParse")
        kind("orchestration", "await [A-Za-z_$.]+\\(|\\.then\\(")
      }
      END {
        n = 0; first = 0; list = ""
        for (k in seen) { n++; list = list k " (line " seen[k] "); "; if (first == 0 || seen[k] < first) first = seen[k] }
        if (n >= 3) printf "FAIL SRP-001: %s:%d: %d kinds of responsibility in one unit: %s(M-107)\n", file, first, n, list
      }' "$root/$f"
  done
}

# $1 = repository root; $2 = decision record, relative to it; $3... = files SRP-001 listed.
srp_record_check() {
  root="$1"; record="$2"; shift 2
  for f in "$@"; do
    grep -qF -- "$(basename "$f")" "$root/$record" 2>/dev/null ||
      echo "FAIL DEC-001: $record:1: SRP-001 lists $f with several responsibilities and the record names no row for it (M-107)"
  done
}

# $1 = repository root; $2... = changed workflow YAML files, relative to it.
# M-107 for a workflow file: a file that combines three or more kinds of responsibility
# is listed, as a source unit is. The kinds are read from its steps, not from its name.
srp_workflow_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.yml|*.yaml) ;; *) continue ;; esac
    awk -v file="$f" '
      function kind(name, re) { if (!(name in seen) && $0 ~ re) seen[name] = NR }
      {
        kind("contract validation", "validate|jq -e|node -e|python3? -c")
        kind("digest verification", "sha256|digest|checksum")
        kind("deploy or publish", "deploy|kubectl|gh release|publish|upload")
        kind("release record parsing", "release\\.json|JSON\\.parse|jq ")
        kind("orchestration", "uses: *\\./|uses: *[^ ]*/\\.github/workflows/")
      }
      END {
        n = 0; first = 0; list = ""
        for (k in seen) { n++; list = list k " (line " seen[k] "); "; if (first == 0 || seen[k] < first) first = seen[k] }
        if (n >= 3) printf "FAIL SRP-001: %s:%d: %d kinds of responsibility in one workflow: %s(M-107)\n", file, first, n, list
      }' "$root/$f"
  done
}

# $1 = repository root; $2 = decision record, relative to it; $3... = changed files.
# M-107 and DEC-001: a record row that says a listed unit has one responsibility ("only",
# "solo", "single") is contradicted by SRP-001 when the unit is listed. The row names the
# unit by its file, its stem, or the key of one of its workflow jobs. The row's claim
# is traced to the code, never accepted from the row itself.
srp_claim_check() {
  root="$1"; record="$2"; shift 2
  listed="$( { srp_unit_check "$root" "$@"; srp_workflow_check "$root" "$@"; } |
    sed -n 's/^FAIL SRP-001: \([^:]*\):.*/\1/p' | sort -u)"
  for f in $listed; do
    b="$(basename "$f")"; stem="${b%.*}"
    # A row names the unit by its file, its stem, or the key of one of its workflow jobs
    # ("deploy job ... deploys only" names the job deploy of promote.yml).
    jobs="$(awk '/^jobs:/ { j = 1; next } j && /^[^ #]/ { j = 0 }
      j && /^  [A-Za-z0-9_-]+:[ \t]*$/ { sub(/^  /, ""); sub(/:[ \t]*$/, ""); printf "%s ", $0 }' "$root/$f" 2>/dev/null)"
    for nm in $b $stem $jobs; do
      grep -nE "(^|[^A-Za-z0-9_-])$nm([^A-Za-z0-9_-]|\$)" "$root/$record" 2>/dev/null |
        grep -iE '(^|[^a-z])(only|solo|single|(u|ú)nica|(u|ú)nico|una sola|un solo|sola responsabilidad|one responsibility)([^a-z]|$)' |
        sed "s#^\([0-9]*\):#FAIL DEC-001: $record:\1: $f is claimed to have one responsibility and SRP-001 lists it with several kinds (M-107): #"
    done
  done | sort -u
}
```

### DUP-001 — A helper repeated in two units is promoted, not copied

M-105: a helper repeated in two or more units is not copied; the change promotes the helper
to the shared unit that owns it. The external audit found the same `takeValue` helper in three
files. The detector extracts each top-level declaration of at least three lines (a `const`,
`let`, or `function` whose body closes at depth zero), normalizes its whitespace, and
groups the declarations that have the same name and body in two or more files. A copy
in a changed file is a `FAIL` at its declaration line, because the change introduced
it or touched it; the copy must be replaced by an import of the shared unit that owns
the helper. A group with no changed copy is a `FINDING` (a pre-existing duplicate),
recorded and not blocking. Limit: a changed file whose copy already existed in the
base revision is reported as `FAIL` by this text check; the reviewer reads the base
revision (`git show`) and downgrades it to a finding, with the reason recorded.

```sh
# $1 = repository root; $2... = changed files, then -- and every source file of the tree.
dup_helper_check() {
  root="$1"; shift
  changed=""
  while [ "$#" -gt 0 ] && [ "$1" != "--" ]; do changed="$changed $1"; shift; done
  [ "$#" -gt 0 ] && shift
  work="$(mktemp -d)" || return 1
  for f in "$@"; do
    case "$f" in *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs) ;; *) continue ;; esac
    case "$f" in *.test.*|*.spec.*|*/__tests__/*|*/tests/*|*/test/*|*.d.ts) continue ;; esac
    awk -v file="$f" '
      {
        o = gsub(/[[{(]/, "&"); c = gsub(/[]})]/, "&")
        if (depth == 0 && $0 ~ /^(export )?(const|let|function) [A-Za-z_$][A-Za-z0-9_$]* *(=|\()/) {
          nm = $0; sub(/^export /, "", nm); sub(/^(const|let|function) /, "", nm); sub(/[^A-Za-z0-9_$].*$/, "", nm)
          cur = nm; start = NR; body = ""
        }
        if (cur != "") { norm = $0; gsub(/[ \t]+/, " ", norm); sub(/^ /, "", norm); body = body " " norm }
        depth += o - c
        if (cur != "" && depth <= 0) {
          if (NR - start + 1 >= 3) print cur "\t" file "\t" start "\t" body
          cur = ""; depth = 0
        }
      }' "$root/$f" >> "$work/blocks.tsv"
  done
  [ -f "$work/blocks.tsv" ] || { rm -rf "$work"; return 0; }
  awk -F '\t' -v changed="$changed" '
    BEGIN { n = split(changed, arr, " "); for (i = 1; i <= n; i++) isch[arr[i]] = 1 }
    {
      key = $1 SUBSEP $4
      if (!((key SUBSEP $2) in seen)) { seen[key SUBSEP $2] = 1; units[key]++ }
      row[NR] = $0; rk[NR] = key
    }
    END {
      for (r = 1; r <= NR; r++) {
        if (units[rk[r]] < 2) continue
        split(row[r], c, "\t")
        if (c[2] in isch)
          printf "FAIL DUP-001: %s:%s: %s is repeated in %d units; import the shared helper from the unit that owns it (M-105)\n", c[2], c[3], c[1], units[rk[r]]
        else
          printf "FINDING DUP-001: %s:%s: %s is repeated in %d units, none of them changed; recorded, not blocking (M-105)\n", c[2], c[3], c[1], units[rk[r]]
      }
    }' "$work/blocks.tsv"
  rm -rf "$work"
}
```

### DUP-001 (shell and workflow YAML) — A block repeated in two files is promoted, not copied

M-105 applies to shell and workflow YAML as it does to source. The text-block detector of
`dup_helper_check` reads TypeScript and JavaScript declarations only, so a loop copied into
two workflow files was never read. This detector normalizes each significant line (trimmed,
blank and comment lines dropped, tabs replaced), takes every window of six consecutive
significant lines of at least 120 characters, and groups the windows that appear in two or
more files. A copy in a changed file is a `FAIL` at its first line; a group with no changed
copy is a `FINDING`. Limit: it compares text, so a repeated block with renamed variables is
not matched; the reviewer compares the candidates the detector lists with the base revision.

```sh
# $1 = repository root; $2... = changed shell or workflow files, then -- and every such file of the tree.
dup_block_check() {
  root="$1"; shift
  changed=""
  while [ "$#" -gt 0 ] && [ "$1" != "--" ]; do changed="$changed $1"; shift; done
  [ "$#" -gt 0 ] && shift
  work="$(mktemp -d)" || return 1
  for f in "$@"; do
    case "$f" in *.sh|*.bash|*.zsh|*.yml|*.yaml) ;; *) continue ;; esac
    awk -v file="$f" '
      {
        line = $0; gsub(/\t/, " ", line); sub(/^ +/, "", line); sub(/ +$/, "", line)
      }
      line == "" || line ~ /^#/ { next }
      { n++; L[n] = line; N[n] = NR }
      END {
        for (i = 1; i + 5 <= n; i++) {
          key = ""
          for (k = 0; k < 6; k++) key = key "\034" L[i + k]
          if (length(key) >= 120) print N[i] "\t" file "\t" key
        }
      }' "$root/$f" >> "$work/blocks.tsv"
  done
  [ -f "$work/blocks.tsv" ] || { rm -rf "$work"; return 0; }
  awk -F '\t' -v changed="$changed" '
    BEGIN { n = split(changed, arr, " "); for (i = 1; i <= n; i++) isch[arr[i]] = 1 }
    {
      if (!(($3 SUBSEP $2) in seen)) { seen[$3 SUBSEP $2] = 1; units[$3]++ }
      row[NR] = $0
    }
    END {
      for (r = 1; r <= NR; r++) {
        split(row[r], c, "\t")
        if (units[c[3]] < 2) continue
        if (c[2] in isch)
          printf "FAIL DUP-001: %s:%s: a six-line block is repeated in %d files; promote it to the shared unit that owns it (M-105)\n", c[2], c[1], units[c[3]]
        else
          printf "FINDING DUP-001: %s:%s: a six-line block is repeated in %d files, none of them changed; recorded, not blocking (M-105)\n", c[2], c[1], units[c[3]]
      }
    }' "$work/blocks.tsv"
  rm -rf "$work"
}
```

### GUARD-001 — A guard in a shell or workflow artifact has a versioned test (M-113)

M-113: a guard in a non-TypeScript artifact (a shell script, a workflow step, a
configuration) is behavior, and the change ships a versioned test that runs it with a
violating input and a compliant input. The external audit found a cleartext guard in a
workflow whose only evidence was a log of one-off runs; the log did not include the
userinfo case, so the bypass passed unseen. The detector finds each changed artifact that
contains a guard (an `exit` with a non-zero status, an `exit $`, a `::error` annotation, or
`|| exit`) and requires a tracked test file that names the artifact. A file counts as a test
only when its path names a test (`test`, `spec`, `__tests__`), so a log never counts. When the
guard reads a URL (`http://` or `https://` in the artifact), the test must also contain a
userinfo URL, an `https://` with an empty host, and a `localhost` host-prefix URL; a missing
case is a `FAIL` naming the case. A test that only names the guard passes the naming check
and proves none of these cases, so it is a `FAIL` of GUARD-001 for each missing case. The
test's execution is read by the reviewer: a test that names the artifact and never runs it
is not a pass. Limit: the detector reads the name and
the presence of the three cases, not the assertion; the twin and the violating input are
verified in the behavioral part of `workflows/system-validation.md` V2.92.

```sh
# $1 = repository root; $2... = changed shell or workflow files, then -- and the tracked test files.
guard_test_check() {
  root="$1"; shift
  changed=""
  while [ "$#" -gt 0 ] && [ "$1" != "--" ]; do changed="$changed $1"; shift; done
  [ "$#" -gt 0 ] && shift
  for f in $changed; do
    case "$f" in *.sh|*.bash|*.zsh|*.yml|*.yaml) ;; *) continue ;; esac
    g="$(grep -nE 'exit [1-9]|exit \$|::error|\|\| *exit' "$root/$f" 2>/dev/null | head -1)"
    [ -n "$g" ] || continue
    b="$(basename "$f")"; covered=""
    for t in "$@"; do
      [ -f "$root/$t" ] || continue
      case "$t" in *test*|*spec*|*__tests__*) ;; *) continue ;; esac
      grep -qF -- "$b" "$root/$t" && { covered="$t"; break; }
    done
    if [ -z "$covered" ]; then
      echo "FAIL GUARD-001: $f:${g%%:*}: a guard in a non-TypeScript artifact has no versioned test that names it (M-113)"
      continue
    fi
    # M-113: a guard that reads a URL is tested with the cases it must reject: userinfo,
    # an empty https host, and a host prefix that matches a local name. A test that only
    # names the guard passes the naming check and proves none of these.
    grep -qE 'https?://' "$root/$f" 2>/dev/null || continue
    grep -qE 'https?://[^/ ]*@' "$root/$covered" 2>/dev/null ||
      echo "FAIL GUARD-001: $covered: the test of $f has no userinfo URL case (M-113)"
    grep -qE 'https://([^A-Za-z0-9_.-]|$)' "$root/$covered" 2>/dev/null ||
      echo "FAIL GUARD-001: $covered: the test of $f has no empty-host https case (M-113)"
    grep -qE 'https?://localhost[.-]' "$root/$covered" 2>/dev/null ||
      echo "FAIL GUARD-001: $covered: the test of $f has no host-prefix case (M-113)"
  done
}
```

### TRACE-001 — A document and its catalog cite each other

M-106: a document and its catalog are traceable in both directions. The external audit found
`guias/datos/DB-12.md` citing `R-35` and `R-07`
while the catalog did not list `DB-12` under them. The check is given the catalog
file, the prefix of its requirement identifiers (for example `R`), and the documents.
For each requirement a document cites, the catalog must have a line that names both
the requirement and the document; for each catalog line that names the document and
a requirement, the document must cite that requirement. A catalog that is absent is
`NOT VERIFIED`. The check depends on the catalog putting the document and its
requirements on one line, which is the convention it reads; a catalog with another
layout is `NOT VERIFIED` for the lines it cannot read.

```sh
# $1 = repository root; $2 = catalog, relative to it; $3 = requirement prefix (for example R);
# $4... = documents, relative to the root. The document id is the file name without .md.
trace_check() {
  root="$1"; catalog="$2"; prefix="$3"; shift 3
  [ -f "$root/$catalog" ] || { echo "NOT VERIFIED TRACE-001: $catalog: catalog not found, so the links were not checked"; return 0; }
  for d in "$@"; do
    [ -f "$root/$d" ] || continue
    id="$(basename "$d" .md)"
    grep -oE "${prefix}-[0-9]+" "$root/$d" | sort -u | while read -r r; do
      if ! grep -qE "(^|[^A-Za-z0-9_-])$r([^0-9]|\$)" "$root/$catalog" ||
         ! grep -E "(^|[^A-Za-z0-9_-])$r([^0-9]|\$)" "$root/$catalog" | grep -qE "(^|[^A-Za-z0-9_-])$id([^0-9A-Za-z-]|\$)"; then
        ln="$(grep -nE "(^|[^A-Za-z0-9_-])$r([^0-9]|\$)" "$root/$d" | head -n 1 | cut -d: -f1)"
        echo "FAIL TRACE-001: $d:${ln:-1}: $r is cited by $id, but $catalog does not list $id under $r (M-106)"
      fi
    done
    grep -nE "(^|[^A-Za-z0-9_-])$id([^0-9A-Za-z-]|\$)" "$root/$catalog" | while IFS=: read -r ln rest; do
      printf '%s\n' "$rest" | grep -oE "${prefix}-[0-9]+" | sort -u | while read -r r; do
        grep -qE "(^|[^A-Za-z0-9_-])$r([^0-9]|\$)" "$root/$d" ||
          echo "FAIL TRACE-001: $catalog:$ln: the catalog lists $r under $id, but $d does not cite $r (M-106)"
      done
    done
  done
}
```

### REPORT-001 — A verdict matrix is consistent with its own rules

M-80: a verdict matrix is consistent with its own rules. An audit, review, or gate
report that carries a matrix of dimensions (`| Dimension | Status | Reason |`) is
checked against the rules of `references/audit-pass-criteria.md` before it is
delivered. The check reads the report's own text and reports each defect with its
`file:line`:

- a `NOT VERIFIED` reason that is an unexecuted review (`no ejecuté`, `not run`,
  `not audited`) is a `FAIL`: the review is run before its verdict is written;
- a `PASS` row that names an open item (`pendiente`, `pending`, `not run`,
  `equivalent`, `diff leído`, `ambiguous`) is a `FAIL`: the dimension is reopened;
- a grouped dimension cell (`9, 10, 12, 13`) is a `FAIL`: each dimension needs its own
  row and its own reason; a dimension number that repeats is a `FAIL`;
- when the matrix numbers its dimensions, every one of the 14 dimensions has a row;
- a `PASS` row with no evidence (no `file:line`, no detector ID, no command in backticks,
  no exit code) is a `FAIL`: a PASS in a verdict matrix carries its evidence (M-98);
- a conclusion line that claims a closure (`cierra`, `closes`, `closed`, `resolved`)
  while a row stays `NOT VERIFIED`, `AMBIGUOUS`, or `FAIL` is a `FAIL` of that line;
- a red whole-project gate line (`npm run`, `format:check`, `lint`, `typecheck`, `test`
  with `falla`, `fails`, `failing`) is a `FAIL` unless the report attributes it
  `BASELINE_FAILURE` (`workflows/commit.md`); a word such as `preexistente` alone is not
  the attribution;
- a line that claims a runtime or version is equivalent (`equivalent`, `equivalente`
  with a runtime, language, or version name) is `NOT VERIFIED` until the claim is
  executed on that version and the evidence is cited (M-80).

The lexical check finds the phrases above; it does not judge whether a phrase is used
in its literal sense. A matrix that uses other words for the same defect is `NOT
VERIFIED` for the reviewer to decide, never `PASS`.

```sh
# $1 = repository root; $2... = report files, relative to it.
report_matrix_check() {
  root="$1"; shift
  for f in "$@"; do
    [ -f "$root/$f" ] || { echo "NOT VERIFIED REPORT-001: $f: report file not found"; continue; }
    awk -v f="$f" '
      function low(s) { return tolower(s) }
      function trim(s) { gsub(/^[ \t`*]+|[ \t`*]+$/, "", s); return s }
      {
        line = low($0)
        if (line ~ /baseline_failure/) {
          baseline = 1
          if (line ~ /(npm run|format|lint|typecheck|test|build)/ && $0 !~ /[0-9]/)
            print "FAIL REPORT-001: " f ":" NR ": a gate attributed BASELINE_FAILURE states no count of the failing files; state the count and the first location (M-80, project-validation)"
        }
        if ($0 ~ /^\|/) {
          n = split($0, c, "|")
          dim = trim(c[2]); st = toupper(trim(c[3])); mot = low(trim(c[4]))
          if (low(st) ~ /^(status|estado)$/ && low(dim) ~ /^dimensi|^dimension|^dim\.?$/) {
            curdim = 1; dimhdr = 1; next
          }
          # A numbered dimension row whose status is not exactly one enumerated verdict is a FAIL.
          # A status with a qualifier ("PASS, con observación", "PASS (parcial)") is never a PASS,
          # and a free-text status ("No ejecutadas en esta pasada") is never silently skipped (M-80).
          if (dim ~ /^[0-9]/ && st !~ /^(PASS|FAIL|WARN|NOT VERIFIED|NOT_VERIFIED|N\/A|AMBIGUOUS|BLOCKED|BASELINE_FAILURE|NOT_RUN|NOT RUN|SKIPPED|SIMULATED|NOT_CONFIGURED|NOT_APPLICABLE|SETUP_REQUIRED)$/) {
            if (st ~ /^PASS[ ,(]/)
              print "FAIL REPORT-001: " f ":" NR ": PASS carries a qualifier \"" trim(c[3]) "\"; grade it WARN with the observation, or reopen the dimension (M-80)"
            else
              print "FAIL REPORT-001: " f ":" NR ": unrecognized status \"" trim(c[3]) "\"; a verdict is one of the enumerated values, and a dimension that was not run is run before its verdict is written (M-80)"
            rows++; bad_rows++; if (curdim) dimrows++
            num = dim; sub(/[^0-9].*$/, "", num)
            if (!(num in seen)) { seen[num] = 1; numbered = 1 }
            next
          }
          if (st ~ /^(PASS|FAIL|WARN|NOT VERIFIED|NOT_VERIFIED|N\/A|AMBIGUOUS|BLOCKED|BASELINE_FAILURE|NOT_RUN|NOT RUN|SKIPPED|SIMULATED|NOT_CONFIGURED|NOT_APPLICABLE|SETUP_REQUIRED)$/ && dim != "") {
            rows++
            if (curdim) dimrows++
            if (st ~ /^(NOT VERIFIED|NOT_VERIFIED|AMBIGUOUS|FAIL|BLOCKED|BASELINE_FAILURE|NOT_RUN|NOT RUN|SKIPPED|SIMULATED|NOT_CONFIGURED|SETUP_REQUIRED)$/) open_rows++
            # M-98: a PASS row carries its evidence: file:line, a detector ID, a command in backticks, or an exit code.
            if (st == "PASS" && $0 !~ /[A-Za-z0-9_.\/-]+:[0-9]+|[A-Z][A-Z0-9]*-[0-9][0-9][0-9]|`[^`]+`|exit [0-9]/)
              print "FAIL REPORT-001: " f ":" NR ": PASS carries no evidence; cite file:line, a detector ID with its output, a command in backticks, or its exit code (M-98)"
            if (dim ~ /^[0-9]+ *(,|y |and )/) {
              print "FAIL REPORT-001: " f ":" NR ": grouped dimension \"" dim "\" gets one verdict; each dimension needs its own row (M-80)"
            } else if (dim ~ /^[0-9]+/) {
              num = dim; sub(/[^0-9].*$/, "", num)
              if (num in seen) print "FAIL REPORT-001: " f ":" NR ": dimension " num " has a second row (M-80)"
              seen[num] = 1; numbered = 1
            }
            if (st ~ /^NOT[ _]VERIFIED$/ && mot ~ /(no ejecut|no se ejecut|no auditad|sin auditar|not executed|not run|not audited|no review)/)
              print "FAIL REPORT-001: " f ":" NR ": NOT VERIFIED reason is an unexecuted review; run the review before the verdict (M-80)"
            if (st == "PASS" && mot ~ /(pendiente|pending|not run|no ejecut|sin ejecutar|equivalen|equivalent|diff le|solo el diff|only the diff|ambig|por confirmar|to be confirmed|simulat)/)
              print "FAIL REPORT-001: " f ":" NR ": PASS carries an open item; the dimension is reopened (M-80)"
          }
          next
        }
        curdim = 0
        # M-80 (prose): a PASS verdict stated outside the matrix (a summary line, "Pipeline green: PASS")
        # is checked like a row. It carries no open item in its own line, and no open item may remain
        # in the report while it stands.
        if ($0 ~ /(^|[^A-Z_])PASS([^A-Z_]|$)/) {
          pp++; ppl = ppl NR " "
          if (line ~ /(pendiente|pending|not run|no ejecut|sin ejecutar|(^|[^a-z])(open|abiert)|equivalen|equivalent|diff le|solo el diff|only the diff|ambig|por confirmar|to be confirmed|simulat)/)
            print "FAIL REPORT-001: " f ":" NR ": a PASS verdict in prose carries an open item; the dimension is reopened (M-80)"
        }
        if (line ~ /(not verified|no verificad|(^|[^a-z])(open|abiert)|still open|sigue abierto|pendiente)/ && $0 !~ /(^|[^A-Z_])PASS([^A-Z_]|$)/) {
          op++; opl = opl NR " "
        }
        if (line ~ /(cierra|cerrad|closes|closed|resolved|resuelto)/) closure = closure NR " "
        if (line ~ /(verdict|veredicto|conclusi|resumen|summary|recommendation|recomendaci)/ && $0 ~ /(^|[^A-Z_])PASS/) closure = closure NR " "
        if (line ~ /(npm run|format:check|lint|typecheck|test)/ && line ~ /(falla|fails|failing|rojo)/) {
          if (!red) red = NR
        }
        if (line ~ /(equivalen|equivalent)/ && line ~ /(node|python|java|ruby|php|\.net|version|versi|runtime)/)
          print "NOT VERIFIED REPORT-001: " f ":" NR ": a runtime or version equivalence is claimed; keep it only with its execution on that version (M-80)"
      }
      END {
        if (numbered && dimrows != 14) print "FAIL REPORT-001: " f ": the numbered dimension matrix has " dimrows " rows; the code audit has 14 dimensions (M-72, M-80)"
        if (open_rows > 0 && closure != "")
          print "FAIL REPORT-001: " f ": lines " closure "claim a closure or a PASS verdict while " open_rows " row(s) stay open (M-80)"
        if (pp && (open_rows > 0 || op > 0))
          print "FAIL REPORT-001: " f ": PASS verdict(s) in prose at lines " ppl "stand while open items remain (" open_rows " matrix row(s), open lines " opl ") (M-80)"
        if (numbered) for (i = 1; i <= 14; i++) if (!(i in seen)) print "FAIL REPORT-001: " f ": dimension " i " has no row (M-72, M-80)"
        if (red && !baseline) print "FAIL REPORT-001: " f ":" red ": a red whole-project gate is not attributed BASELINE_FAILURE (M-80)"
      }' "$root/$f"
  done
}
```

## Procedure

1. Resolve the target project root, the base revision, and the change set
   (staged, unstaged, untracked). Record HEAD and the branch.
2. Exclude generated output, dependencies, and build artifacts.
3. Run **every row of the coverage table below whose "Applies when" holds**.
   The table is the complete list of the detectors this file defines. A detector or check that a workflow defines is a control only when the procedure that a verdict cites runs it on the scope it covers (M-75). A row
   that applies and is not run is a skipped check: it is reported as
   `NOT VERIFIED` with its reason and never as passing. A detector that
   exists in this file and is absent from the table is a defect of this file,
   and `V2.77` in `workflows/system-validation.md` reports it.
4. Run `TYPE-002` over the contract files identified by `TYPE-001`.
5. When a claim, a contract, or a pattern is corrected, search its text across
   the change set and the project sweep. Every surviving occurrence is a finding
   with its `file:line`, so a correction in a test or a contract is not complete
   while a source comment still states the old claim (M-75, M-68).
6. Review each match against the allowed reasons before reporting it.
7. Report every finding with `file:line`, the rule id, the severity, and the
   concrete fix.

### Coverage table

| Rule | Scope | Applies when |
|---|---|---|
| `FUNC-001` | change set | TypeScript or JavaScript source is created or modified |
| `TYPE-001` | change set | TypeScript or JavaScript source is created or modified |
| `TYPE-002` | contract files named by `TYPE-001`; `consumer_group_check` | a contract file is in the change set |
| `TYPE-003` | change set | a contract is forwarded by a re-export or alias |
| `TYPE-004` | source tree | any contract is declared in the change set |
| `FILE-001` | change set and source tree | source is created or modified |
| `STRUCT-001`, `STRUCT-006` | every feature folder | the project is feature-driven |
| `STRUCT-002`, `STRUCT-003` | feature imports and public APIs | the project is feature-driven |
| `STRUCT-004` | created tree | a new tree is created |
| `STRUCT-005` | repository topology | the project is a monorepo |
| `STRUCT-007` | contract files in feature folders | the project is feature-driven |
| `CONFIG-001` | added configuration and lint files | the change adds lint or configuration files |
| `LAYER-001` | source tree | the architecture declares layers |
| `TEST-001`, `TEST-002` | created and modified test files | a test file is created or modified |
| `TEST-003` | test and test-support files, change set and sweep | a test or test-support file is created or modified |
| `DEAD-EXPORT-001` | modules that add an export | an export is added |
| `SAFE-001`, `SAFE-002`, `SAFE-003` | changed files of any type, including shell scripts and workflow YAML (`safe_cleartext_check`, `safe_bound_check`) | outbound calls, URLs, endpoints, or temporary paths change in any file type (M-54, M-55, M-58) |
| `GUARD-001` | changed shell and workflow YAML files, against the tracked test files | a changed shell or workflow file contains a guard (M-113) |
| `SAFE-004`, `SAFE-005`, `SAFE-006`, `SAFE-007`, `AUTH-001` | changed files | URLs, cache, credentials, backup, or identity change |
| `DEP-001` | manifest and lockfile diff | a manifest or lockfile changes |
| `SAFE-008`, `SAFE-009` | changed files | cross-origin, development tools, uploads, or file writes change |
| `I18N-001`, `I18N-002`, `I18N-003`, `I18N-004` | changed UI and formatter files | the approved requirement includes locales or RTL |
| `CONSUME-001` | changed units | a changed unit consumes another unit |
| `DEC-001` | decision records in the change set | a record or pattern table is in the change set |
| `CLAIM-001` | documents and source comments in the change set | a changed file contains text |
| `ERR-001` | source files in the change set and the sweep; the typed-class search reads the whole project | a source file is created or modified in a project that declares a typed error class |
| `ERR-002` | contract documents, source and test files in the sweep | a project declares error codes in a contract document |
| `ERR-003` | source files in the sweep, contract documents | a source file quotes an upper-snake error code |
| `ENTRY-001` | source files and test files in the change set and the sweep | a source file reads `process.argv`; each exit code it sets is asserted (M-118) |
| `ERR-002` per site | source files and test files in the change set and the sweep | a code is emitted from several sites (M-118) |
| `DELIVERY-001` | workflow YAML in the change set and the sweep | a workflow names an approval (M-114) |
| `DELIVERY-002` | workflow YAML in the change set and the sweep | a workflow checks out a dispatched ref and deploys (M-116) |
| `IMPORT-001` | source files in the change set and the sweep | a source module is changed (M-117) |
| `EXEC-001` | reports and documents in the change set and the sweep | a document claims that an agent or a prompt ran (M-115) |
| `VAL-001` | source files in the change set and the sweep | a source file reads `process.argv` |
| `DB-003` | source files in the change set and the sweep | a source file holds a page query inside a loop |
| `DB-001` | source files in the change set and the sweep (a pre-existing runtime DDL is a finding, not a blocker) | any source file is created or modified |
| `DB-002` | migrations and tests in the change set and the sweep | a migration is created or modified |
| `DB-004` | source files and migrations in the change set and the sweep | a source file issues a `SELECT` with a `WHERE` or `AND` equality, or a migration creates an index (M-85) |
| `DB-005` | migrations and tests in the change set and the sweep | a migration adds a visibility column such as `deleted_at` (M-100) |
| `DB-006` | source files in the change set and the sweep | a source file captures a schema probe in a field or module constant (M-102) |
| `CACHE-001` | source files in the change set and the sweep | a source file reads a cache and decides a charge, authorization, or revocation (M-101) |
| `AUTH-002` | changed authorization, permission, role, or scope files | such a file is created or modified (M-86) |
| `COMMIT-002` | commits in `base..HEAD` against the approved scope manifest | a commit proposal is prepared (M-87) |
| `SCOPE-001` | change set, tracked and untracked, against the approved manifest | any change set exists; a missing manifest is a `FAIL` (M-90, M-91) |
| `SCOPE-002` | change set diff against the approved manifest | a source, configuration, or test file is modified |
| `APPROVAL-001` | reports, decision records, checkpoints, and proposals in the change set, and every Markdown document of the sweep (M-92, M-111) | such a file claims a human approval (M-92) |
| `EVIDENCE-001` | documents in the change set and the sweep | a document claims a time window, counts a set, or says a tool covers files without the tool's file resolution (M-109) |
| `PRECOND-001` | documents in the change set and the sweep | a document declares an initial state or a precondition (M-108) |
| `CI-001`, `NODE-001`, `DEP-RANGE-001` | the whole project, from the health probe | the health probe runs (M-112) |
| `TEARDOWN-001` | the task checkpoint in the handoff directory | a commit proposal is prepared (M-93) |
| `INV-001` | reports or analyses that list an inventory | the change set contains such a report |
| `STATE-001` | read-only gates | a read-only gate runs |
| `A11Y-001`, `A11Y-002` | changed UI and platform files | accessibility applies |
| `DIP-002` | changed logic modules | a logic module imports platform code |
| `DIP-001` | every source file of the change set and the sweep; the `DEC-001` cross-check reads the whole source tree | a source file is created or modified, or a decision record claims APPLIES or PRESERVED for DIP (M-96) |
| `ARTIFACT-001` | documents in the change set and the sweep | a document cites an artifact path (M-97), a path that climbs out of the repository, a home or absolute path, or a directory (M-110) |
| `BEHAVIOR-001` | modified test files against the base revision, and the approved manifest | a test file is modified (M-94) |
| `PREFLIGHT-001` | the target repository, before alignment | a task begins alignment (M-95) |
| `COMMIT-001` | each proposed commit group, from `workflows/commit.md` Phase 11 | a commit group is proposed |
| `BOUND-001` | changed source files that declare a bound, and the test files | a changed source declares a timeout, a busy or lock wait, a deadline, or a retry budget |
| `COMMENT-001` | changed TypeScript or JavaScript files | a changed file contains a comment |
| `DEC-002` | decision records and contract documents, every line with a citation | a record or contract document is in the change set (R-136, R-145) |
| `DEL-001` | source files in the change set and the sweep | a source file removes a path recursively or overwrites a supplied path (M-103) |
| `PARSE-001` | source files in the change set and the sweep | a source file assigns the result of `JSON.parse(` (M-104) |
| `SRP-001` | source files and workflow YAML in the change set and the sweep; `srp_record_check` and `srp_claim_check` for each decision record | a source file or workflow contains three or more kinds of responsibility, or a decision record names a listed unit or claims one responsibility for it (M-107) |
| `DUP-001` | source files in the change set against the source tree; `dup_block_check` over shell and workflow YAML | a changed file declares a helper of three or more lines that another unit also declares, or a changed shell or workflow file repeats a six-line block (M-105) |
| `TRACE-001` | catalog and documents in the change set; `trace_check` takes the catalog and its prefix | a document cites a requirement identifier of a catalog, or a catalog lists a document (M-106) |
| `REPORT-001` | every delivered audit, review, or gate report that carries a verdict matrix | a report with a dimension matrix is delivered (M-80: a verdict matrix is consistent with its own rules) |
| `CHECK-001`, `CHECK-002` | phase reports | the work reports phases or checklists |
| `GATE-001` | installation proposals | a dependency or tooling installation is proposed |
| Whole-project sweep | project, health probe | the health probe runs; the sweep is not a change-set check |

## Reporting

```
### OUTPUT COMPLIANCE

- Scope: <paths, branch, commit>
- Checks executed: every coverage-table row whose condition held, with its result
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
