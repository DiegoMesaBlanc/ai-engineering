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
declares it (`vitest`, or `@jest/globals` for Jest), so the type checker
accepts it without a global test declaration (M-47, `AGENTS.md`). Without
the import the file fails with `Cannot find name 'describe'` (TS2593). A
test case is declared with `test`, not `it`.

**Scope.** The check reads every test file the change set **creates** in full,
and every test file it **modifies** only on the lines the change set adds. A
violation that already exists in the base revision of a modified file is a
recorded finding, never a `FAIL` (`AGENTS.md`, pre-existing violations). A test
file the task does not touch is never read.

**Detection.** The function reads each file and reports every line where a
test API is called or read as a member, `(?<![.\w$])NAME(` or `NAME.identifier`,
and every `it(` case. The name must appear in an import from `vitest` or
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
    while ($s =~ /import\s+(?:type\s+)?\{([^}]*)\}\s*from\s*["\x27](?:vitest|\@jest\/globals)["\x27]/g) {
      for my $n (split /,/, $1) { $n =~ s/^\s+|\s+$//g; $n =~ s/\s+as\s+\S+$//; $imp{$n} = 1 if length $n; }
    }
    for my $n (qw(describe test it expect vi beforeEach afterEach beforeAll afterAll)) {
      while ($s =~ /(?<![.\w\$])\Q$n\E(?:\s*\(|\.[A-Za-z_])/g) {
        my $line = 1 + (substr($s, 0, $-[0]) =~ tr/\n//);
        next if $filter && !$on{$line};
        print "FAIL TEST-001: $f:$line: $n is used without an explicit import\n" unless $imp{$n};
      }
    }
    while ($s =~ /(?<![.\w\$])it\s*\(/g) {
      my $line = 1 + (substr($s, 0, $-[0]) =~ tr/\n//);
      next if $filter && !$on{$line};
      print "FAIL TEST-002: $f:$line: the case is declared with it, not test\n";
    }
  ' "$1" "$filter" "$added")
  [ -n "$findings" ] && { printf '%s\n' "$findings"; return 1; }
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
- **SAFE-002 (M-55).** `http://` is rejected for every non-local build. A string
  literal that starts with `http://` in a changed source file is a `FAIL`, unless
  its host is `localhost`, `127.0.0.1`, `10.0.2.2` (the Android emulator loopback),
  or a documentation domain (`example.com`, `example.org`), and the literal is
  named as a local development address.
- **SAFE-003 (M-58).** No hardcoded environment endpoint and no fixed shared
  temporary path. A `localhost` or `127.0.0.1` URL in non-test TypeScript or
  JavaScript source is a `FAIL`, and a fixed temporary-directory path literal in a changed script is
  a `FAIL` unless the line also reads `TMPDIR` (the `${TMPDIR:-/tmp}` fallback of
  a `mktemp` call is not a fixed path).

**Scope limit.** The detectors read TypeScript and JavaScript and POSIX shell.
Other ecosystems are `NOT VERIFIED` for SAFE-001 and SAFE-002 by these detectors
and are reviewed under `references/audit-pass-criteria.md` Dimension 6 and
Dimension 8 instead. A detection is a candidate: the reviewer confirms each match
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

# $1 = repository root; $2... = changed TypeScript, JavaScript, or shell files.
safe_cleartext_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.md) continue ;; esac
    grep -nE "[\"'\`]http://" "$root/$f" 2>/dev/null |
      grep -vE 'localhost|127\.0\.0\.1|10\.0\.2\.2|example\.(com|org)' |
      sed "s#^#FAIL SAFE-002: $f:#"
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
  migrations=$(printf '%s\n' "$files" | grep -E '(^|/)(migrations?|db/migrate)/')
  tests=$(printf '%s\n' "$files" | grep -E '(\.test\.|\.spec\.|(^|/)(tests?|__tests__)/)')
  echo "SWEEP scanned: $(printf '%s\n' "$src" | sed '/^$/d' | wc -l | tr -d ' ') source files, $(printf '%s\n' "$shells" | sed '/^$/d' | wc -l | tr -d ' ') shell scripts, $(printf '%s\n' "$manifests" | sed '/^$/d' | wc -l | tr -d ' ') build manifests"
  echo "SWEEP excluded: node_modules, dist, build, .next, coverage, .git, and declaration files (*.d.ts)"
  safe_timeout_check . $src
  safe_cleartext_check . $src $shells
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
  decision_record_check . $decisions
  claim_check . $mds
  db_runtime_ddl_check . $src
  db_deploy_order_check . $migrations -- $tests
  [ -n "$manifests" ] || echo "NOT VERIFIED M-61: no build manifest or app configuration is tracked, so backup is not verified"
  echo "NOT VERIFIED M-57 (downgrade): needs a base revision; run dependency_downgrade_check on the change set"
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

M-50 and M-45: its record is in the change set and names its patterns. A record in
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
  done
}
```

### CLAIM-001 — Universal, exclusive, and artifact claims are enumerated first

M-68: a claim that is verified before it is kept. A README, document, or comment that says `only`, `every`, `each`, `all`,
or the equivalent (`solo`, `único`, `cada`, `todos`), or that cites a
screenshot, capture, log, or image, makes a claim that must be checked against
the repository before it is kept. The check cannot decide a claim, so every
match is reported as `NOT VERIFIED` with its line: the verifier enumerates every element of the
scope with a search, or confirms that the artifact exists, and cites the result.

```sh
# $1 = repository root; $2... = documents or source files, relative to it.
claim_check() {
  root="$1"; shift
  for f in "$@"; do
    case "$f" in *.md|*.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs|*.sh) ;; *) continue ;; esac
    grep -nEi "(^|[^A-Za-z])(only|every|each|all|solo|todos|todas|cada|único|unico)([^A-Za-z]|$)" "$root/$f" 2>/dev/null |
      sed "s#^#NOT VERIFIED CLAIM-001: $f:#; s#\$#  (universal or exclusive claim: enumerate its scope before keeping it, M-68)#"
    grep -nEi "capturas?|screenshots?|\.png|\.jpe?g|\.gif|evidence|evidencia" "$root/$f" 2>/dev/null |
      sed "s#^#NOT VERIFIED CLAIM-001: $f:#; s#\$#  (claim that cites an artifact: confirm the artifact is in the repository, M-68)#"
    grep -nEi "order-safe|order safe|either order|any deployment order|both directions|ambas direcciones|cualquier orden|en ambos sentidos" "$root/$f" 2>/dev/null |
      sed "s#^#NOT VERIFIED CLAIM-001: $f:#; s#\$#  (compatibility claim: keep it only with a test that executes each deployment order, M-69)#"
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
      echo "NOT VERIFIED DB-002: $m:1: a deploy-order test is named ($(echo $found | cut -d' ' -f1)); execute it in both orders before claiming order-safety (M-69)"
    else
      echo "FAIL DB-002: $m:1: no test executes this migration in either deployment order (M-69)"
    fi
  done
}
```

### SCOPE-001 — Every changed file is in the approved scope manifest

M-71: the change set is compared with the scope manifest the user approved. This
check runs on the change set, not on the project, because the approved scope is a
property of one task. A changed file the manifest does not name is a `FAIL`,
which catches the formatter rewrite of an unrelated file. The check reads file
names; a hunk that changes an unrelated line of an approved file is judged by
review under Dimension 14 of `references/audit-pass-criteria.md`.

```sh
# $1 = repository root; $2 = scope manifest file, one approved path per line.
scope_file_check() {
  root="$1"; manifest="$2"
  git -C "$root" diff --name-only HEAD 2>/dev/null | sed '/^$/d' | while read -r f; do
    grep -qxF -- "$f" "$manifest" || echo "FAIL SCOPE-001: $f:1: changed but not named in the approved scope manifest (M-71)"
  done
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

## Procedure

1. Resolve the target project root and the change set (staged, unstaged,
   untracked).
2. Exclude generated output, dependencies, and build artifacts.
3. Run `FUNC-001` over the change set.
4. Run `TYPE-001` over the change set.
5. Run `TYPE-002` over the contract files identified in `TYPE-001`.
6. Run `TYPE-003` over the change set, in the form the detected ecosystem
   uses to forward a contract, and `TYPE-004` over the source files, so a
   contract declared in two files is found.
7. Run `FILE-001` over the change set and the source tree: unit folders, test
   placement, and support locations.
8. Run `STRUCT-001` and `STRUCT-006` over every feature folder.
9. Run `STRUCT-007` over every contract file in a feature folder.
10. Run `STRUCT-002` and `STRUCT-003` when the project is feature-driven.
11. Run `LAYER-001` over the source tree when the architecture declares layers.
12. Run `SAFE-001`, `SAFE-002`, and `SAFE-003` over the changed files, listed
    from `git` (M-54, M-55, M-58), and `SAFE-004` to `SAFE-007`, `AUTH-001`, and
    `DEP-001` over the same files (M-52, M-56, M-57, M-59, M-60, M-61). The whole
    project is swept by `project_safety_sweep` in the health probe, not only the
    change set.
13. Review each match against the allowed reasons before reporting it.
14. Report every finding with `file:line`, the rule id, the severity, and the
    concrete fix.

## Reporting

```
### OUTPUT COMPLIANCE

- Scope: <paths, branch, commit>
- Checks executed: FUNC-001, TYPE-001, TYPE-002, TYPE-003, TYPE-004, FILE-001, STRUCT-001, STRUCT-006, STRUCT-007, LAYER-001, SAFE-001 to SAFE-007, AUTH-001, DEP-001
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
