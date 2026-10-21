# System Audit Workflow

## Purpose

Apply an adjustment to the AI Engineering System and prove, in the same
run, that the result is correct.

It has two halves, in this order:

1. **Implement** the adjustment, following the normal engineering flow.
2. **Audit** the result, with a structural audit of the repository and a
   behavioral audit that builds a real artifact, proves the system accepts
   it, and proves it rejects the violation the adjustment was meant to
   prevent.

The behavioral half is what separates "the document says so" from "it
works". A rule that was never exercised against real code is not verified.

`commands/audit-e2e.md` is the OpenCode adapter that delegates here.

## Input

The adjustment itself, in plain language. The text is the only input. It
defines the scope: what changes, which area of the system it touches, and
therefore which checks and which artifacts the audit must use.

There is no flag, no mode, and no implicit scope. The command always does
the same thing: implement the adjustment, then audit it.

## Phase 1 — Understand the Adjustment

1. Resolve the canonical system root.
2. Read the adjustment and determine:
   - **what changes**: the rule, flow, artifact, or reference affected;
   - **which area** of the system it belongs to;
   - **which rules** are in scope;
   - **which artifacts** the behavioral audit must build.
3. Record the scope before editing anything. State it in the working
   interpretation, per `skills/task-context/SKILL.md`.
4. If the adjustment is ambiguous in a way that changes the outcome, ask
   the user before implementing. Do not ask about anything the repository
   already settles.

### Area to check mapping

This table is what makes the audit work for any adjustment. The area comes
from the adjustment text, never from an external scope argument.

| Adjustment is about | Required checks | Artifact to build |
|---|---|---|
| A policy or a rule in `AGENTS.md` | The rule has a canonical wording in `AGENTS.md`, a registry row in `references/mandatory-rules.md`, and a reachability entry. A rule that lives only in a Skill fails | The rule exercised on a synthetic case that violates it and on one that satisfies it |
| The rule registry | Every anchor resolves on every required surface, and every canonical wording is in `AGENTS.md` | A rule whose reachability was broken, to prove the check detects it |
| The regression matrix | No duplicate or missing identifier, and every changed rule has a row stating its evidence | The evidence the row claims, executed |
| The validation workflow | The index and the sections match exactly, every required file exists, and every new check is executable | The new check, run against a real violation |
| The linter artifacts | The rules install and run in a real project: a compliant artifact passes, and each targeted violation is detected with `file:line` | A small service per affected ecosystem, with the rules installed from `references/linter-rules.md` |
| The compliance checks | Each detection pattern finds the violation it claims to find, including the aliased and relative forms | An artifact that contains each violation in both forms |
| The project creation order | A real artifact per affected project type can be created following the order, passes the structural rules, and the checklist is completable | A service, a frontend, a library, and a mobile app, per the affected types |
| The startup decision catalog | Every project type in scope has its decisions, each with a default, a user-decision condition, and a source | The decision resolution applied to a synthetic scope |
| A workflow or a flow | The phases are ordered, every referenced file exists, and the flow is reachable from its command or from the orchestrator | A trace of the flow: the entry point, the phases in order, and the files each phase reads |
| A Skill | The description names the concrete triggers, the canonical source is referenced instead of restated, and the skill is reachable | The skill loaded in the situation its description claims to match |
| A command | It delegates to a workflow and copies no block from it | The command run against its workflow |
| An agent | It references the shared workflows and skills instead of restating them | The agent's task dispatch |
| A reference or a document | Every URL resolves and every referenced file exists | The document's referenced artifact |
| The token efficiency policy | The budget, the stop rules, and the measurement are defined, and the reachability check passes | A session-like workload with a known exploration profile |

If the adjustment does not fit any row, derive the check from the rule it
implements and state it in the report.

## Phase 2 — Implement

Apply the adjustment with the normal engineering flow, not with a shortcut:

- Follow `workflows/task-to-code.md`: understand, align, prepare the branch,
  implement, validate, review.
- Keep the canonical wording in `AGENTS.md` and make every other surface
  reference it instead of restating it.
- A new rule needs its canonical wording, its required surfaces, its
  registry row, its regression row, and its check.
- Do not create a parallel flow, command, or file that duplicates what
  already exists.
- Do not commit without the explicit approval required by the commit
  approval policy.

## Phase 3 — Structural Audit

Always executed after the implementation. Every check reports `PASS`,
`FAIL`, or `NOT VERIFIED` with the reason and the evidence.

### 3.1 Reference integrity

Every local Markdown link resolves, and every repository file path in
backticks exists. A reference to an artifact installed in a target project,
such as an ESLint configuration, is not a failure.

```sh
grep -rhoE '\]\([^)#[:space:]]+\.md\)' --include='*.md' . |
  sed 's/^](//; s/)$//' | sort -u
```

### 3.2 Command catalog

Every command in `commands/` has its own description in the README, so the
catalog does not drift from the commands that exist.

```sh
for command in commands/*.md; do
  name=$(basename "$command" .md)
  grep -q "^- \`/$name\`:" README.md ||
    echo "FAIL command /$name has no description in README.md"
done
```

A command without a README description is a `FAIL`: the catalog a reader
consults is incomplete.

### 3.3 Index, inventory, and numbering

- The `V2.x` index and the `V2.x` sections match exactly: no orphan entry
  and no unindexed section.
- Every file in the required-file inventory exists.
- The `R-nn` identifiers have no gap and no duplicate.

### 3.4 Rule reachability and cross-client reach

For every row of `references/mandatory-rules.md`, the anchor is present on
every required surface, and the canonical wording is present in `AGENTS.md`.
`AGENTS.md` is the only file every client reads, so a rule absent from it
does not reach every client.

```sh
normalize() { tr '\n' ' ' | tr 'A-Z' 'a-z' | tr '-' ' ' | tr -s ' ' ' '; }
trim() { sed 's/`//g; s/^[[:space:]]*//; s/[[:space:]]*$//'; }

grep -E '^\| M-[0-9]+ \|' references/mandatory-rules.md |
while IFS='|' read -r _ id _ _ anchor surfaces; do
  id="$(printf '%s' "$id" | trim)"
  anchor="$(printf '%s' "$anchor" | trim)"
  normalized_anchor="$(printf '%s' "$anchor" | normalize)"
  printf '%s' "$surfaces" | tr ',' '\n' | while read -r surface; do
    surface="$(printf '%s' "$surface" | trim)"
    [ -z "$surface" ] && continue
    if [ ! -f "$surface" ]; then
      echo "FAIL $id [$anchor] surface missing: $surface"
      continue
    fi
    if ! normalize < "$surface" | grep -qF "$normalized_anchor"; then
      echo "FAIL $id [$anchor] not reachable in: $surface"
    fi
  done
done
```

### 3.5 Duplication and contradiction

- Two files must not carry the same normative wording. A surface that
  restates a rule is a drift risk; it must reference the canonical source.
- Two files must not carry incompatible wording of the same rule. Search the
  superseded phrasings when a rule changed, and confirm each is gone.
- A command must not copy a block from the workflow it delegates to.

### 3.6 Coverage

When the adjustment affects a rule that applies to several ecosystems or
project types, confirm the coverage is complete for all of them, not only
the one named in the adjustment. Report the count per ecosystem or type.

## Phase 4 — Behavioral Audit

This is the part that proves the system works. It runs in a temporary
directory outside the repository, and it always follows the same cycle:

1. **Build** a real artifact following the system flow, using the canonical
   artifacts from the repository.
2. **Verify** the compliant artifact passes every rule.
3. **Inject** the violation the adjustment was meant to prevent.
4. **Verify** the violation is detected with `file:line` evidence.
5. **Fix** the artifact and re-verify that it passes.
6. **Clean** the temporary directory.

If a check finds a defect in the system's own artifact, report it as a
defect, correct it, and repeat the cycle. A check that finds nothing
because it never looks is not evidence.

### 4.1 Rules for the behavioral artifacts

- Build the artifact from the canonical artifacts of the repository, not from
  a paraphrase of them. A behavioral audit of a paraphrase proves nothing.
- Use the real tooling when the ecosystem's tooling is available in the
  environment. When it is not, report the check as `NOT VERIFIED` with the
  reason. Never report an unexecuted check as passed.
- Keep the temporary directory outside the repository and delete it when
  finished.
- The artifacts are disposable. Never copy them into the repository, and
  never modify the audited files.

## Phase 5 — Report

Produce:

- **The adjustment**: what was understood, and what was implemented.
- **Structural audit**: every check with `PASS`, `FAIL`, or `NOT VERIFIED`
  and the evidence.
- **Behavioral audit**: for each artifact, what was built, what passed, what
  was injected, what was detected with `file:line`, and what was fixed.
- **Defects found in the system's own artifacts**, if any.
- **Limits**: what could not be verified and why. Distinguish what the
  mechanism guarantees from what depends on an agent following the flow.
- **The commit text**, ready to use. Do not create the commit.

## Related

- `workflows/system-validation.md` — the consistency audit of the whole
  system, independent of any adjustment.
- `workflows/task-to-code.md` — the engineering flow this workflow follows
  in Phase 2.
- `workflows/output-compliance.md` — the structural compliance checks the
  behavioral artifacts are verified against.
- `references/mandatory-rules.md` — the registry of mandatory rules and
  their required surfaces.
- `references/system-workflow-regression-matrix.md` — the rules, their
  files, their expected behavior, and their evidence.
