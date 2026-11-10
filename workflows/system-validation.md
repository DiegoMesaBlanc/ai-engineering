# System Validation Workflow

## Purpose

Audit the consistency of the AI Engineering System and its client
integration without modifying files or remote services.

This workflow audits the **whole system** as it stands. It is not
`workflows/system-audit.md`, which applies a change and then audits that
change. Use this one for a consistency sweep; use the other one when a
change is being applied.

## 1. Resolve the Canonical Root

Determine the system root using this preference:

1. AI_ENGINEERING_HOME when configured and valid.
2. The repository root of the current working directory.
3. The resolved target of the global OpenCode AGENTS.md link.
4. $HOME/Documents/ai-engineering, only if that directory exists.

Do not assume the current working directory is the system root.

If the root cannot be established, report the missing information.

## 2. Validate the Core

Check for the existence of:

- AGENTS.md
- CLAUDE.md
- README.md
- agents/orchestrator.md
- agents/reviewer.md
- agents/auditor.md
- agents/external-auditor.md
- commands/task.md
- commands/generate-pr.md
- commands/review-pr.md
- commands/commit.md
- commands/audit-e2e.md
- commands/create-project.md
- commands/adopt-baseline.md
- commands/system-check.md
- commands/validate.md
- commands/code-audit.md
- commands/checkpoint.md
- commands/resume-work.md
- commands/provider-check.md
- commands/delivery-check.md
- commands/estimate.md
- contracts/task-provider.md
- contracts/design-provider.md
- contracts/git-provider.md
- integrations/access-strategy.md
- integrations/provider-policy.md
- integrations/provider-matrix.md
- integrations/external-skill-registry.md
- integrations/skill-catalog.md
- references/official-documentation.md
- references/linter-rules.md
- references/system-workflow-regression-matrix.md
- references/design-patterns.md
- references/token-efficiency.md
- references/mandatory-rules.md
- references/project-startup-decisions.md
- references/project-creation-order.md
- workflows/task-to-code.md
- workflows/pull-request-review.md
- workflows/code-audit.md
- workflows/commit.md
- workflows/system-validation.md
- workflows/project-validation.md
- workflows/session-resilience.md
- workflows/self-learning.md
- workflows/provider-integration-proof.md
- workflows/secure-delivery.md
- workflows/dependency-assessment.md
- workflows/task-planning.md
- workflows/estimation.md
- workflows/project-initialization.md
- workflows/engineering-baseline.md
- workflows/system-audit.md
- workflows/output-compliance.md

Check that CLAUDE.md references AGENTS.md rather than duplicating
the complete shared instructions.

Do not hardcode a required-file total. The inventory above is the
maintained list; report the count as derived from the actual file
existence check, not from a fixed constant. A partial commit that removes
one of the required files must fail this check.

## 3. Validate Skills

Check that every referenced Skill exists at its canonical location.

At minimum inspect:

- discovery
- project-analysis
- stack-detection
- task-context
- design-analysis
- complexity-advisor
- architecture-advisor
- implementation-recipes
- api-contract-design
- database-engineering
- internationalization
- testing-strategy
- engineering-review
- git-workflow
- repository-strategy
- pull-request
- review-memory
- accessibility
- seo
- performance
- core-web-vitals
- best-practices
- engineering-baseline
- security-review
- vulnerability-catalog

Do not require a dedicated Next.js Skill.

Next.js must be covered by stack detection.

## 3.1 Validate External Skills

Read:

- integrations/external-skill-registry.md

For every registered external Skill, verify:

- canonical local directory exists
- SKILL.md exists and contains valid metadata
- source repository and source path are recorded
- upstream commit is recorded as a full Git commit SHA
- license identifier is recorded
- required license file exists
- supporting files referenced by SKILL.md exist
- local modifications are documented
- no conflicting internal Skill or policy is introduced

For the initially selected Skills, verify:

### frontend-design

- Path: skills/frontend-design/
- Required files: SKILL.md and LICENSE.txt
- License: Apache-2.0

### test-driven-development

- Path: skills/test-driven-development/
- Required files: SKILL.md, references/writing-good-tests.md, and LICENSE-MIT.txt
- License: MIT
- Verify that the local language-neutral adaptation is documented.

### accessibility

- Path: skills/accessibility/
- Required files: SKILL.md, references/WCAG.md,
  references/A11Y-PATTERNS.md, and LICENSE.txt
- License: MIT
- Verify that the removed orchestration reference is documented.

### seo

- Path: skills/seo/
- Required files: SKILL.md, references/STRUCTURED-DATA.md, and LICENSE.txt
- License: MIT
- Verify the application-type gate and the removed orchestration reference
  are documented.

### performance

- Path: skills/performance/
- Required files: SKILL.md, references/MEASUREMENT.md, references/RUM.md,
  and LICENSE.txt
- License: MIT
- Verify the documented memory-profiling limitation.

### core-web-vitals

- Path: skills/core-web-vitals/
- Required files: SKILL.md, references/LCP.md, references/INP.md,
  references/CLS.md, and LICENSE.txt
- License: MIT
- Verify the application-type gate is documented.

### best-practices

- Path: skills/best-practices/
- Required files: SKILL.md, references/SECURITY.md, and LICENSE.txt
- License: MIT
- Verify that the removed orchestration reference is documented.

Possible statuses:

- VERIFIED
- NOT INSTALLED
- LICENSE MISSING
- SOURCE REVISION MISSING
- SUPPORTING FILE MISSING
- POLICY CONFLICT
- NOT VERIFIED

Never report an external Skill as VERIFIED based only on its registry entry.

Report Required External Skill readiness as BLOCKED while a required
external Skill is missing or its license and source cannot be verified.

## License Metadata Interpretation

The registry records the canonical SPDX license identifier.

The upstream Skill frontmatter may use a license-file reference, an SPDX
identifier, or omit a license field.

Do not require the frontmatter value to match the registry SPDX identifier
literally.

For frontend-design:

- Accept the upstream metadata "Complete terms in LICENSE.txt".
- Verify that LICENSE.txt exists.
- Verify that the registry records Apache-2.0.

For test-driven-development:

- A missing license field in SKILL.md is not automatically an error.
- Verify that LICENSE-MIT.txt exists and contains the MIT license.
- Verify that the registry records MIT.

Report LICENSE METADATA INCONSISTENCY only if the metadata contradicts
the actual license evidence or points to a missing license file.

Do not modify upstream Skill metadata solely to satisfy a simplistic
string-equality check.

## 4. Validate Client Adapters

For OpenCode, inspect:

- global AGENTS.md configuration
- global command discovery
- global agent discovery
- global Skill discovery
- command descriptions
- referenced agent names
- agent frontmatter
- relevant permissions

Inspect existing paths before suggesting any link changes.

Never overwrite an existing configuration.

For Codex CLI, inspect:

- the installed `codex --version`
- the canonical README's Codex CLI setup instructions
- the model-visible input from `codex debug prompt-input` when available
- whether instructions are auto-loaded from the current project's
  `AGENTS.md` and whether shared instructions and Skills are explicitly
  referenced when the canonical repository is outside the project
- whether the setup incorrectly claims that files in `commands/` become
  Codex slash commands

Classify `debug prompt-input` as STATIC evidence: it shows what Codex would
receive, not that an agent read the referenced files or completed a task. A
successful `codex exec` response is REAL behavioral evidence. If execution
fails before the agent responds, report behavior as NOT VERIFIED and include
the observed failure. Do not initiate billable execution without
authorization. Do not modify global Codex configuration during validation.

### External-Directory Permissions

Verify against the actual global client configuration, not only the policy
text, that external-directory access remains narrowly scoped:

- external-directory access does not default to an unrestricted wildcard
  `allow`
- the default action for unmatched external paths is `ask` or stricter,
  never a blanket `allow`
- the checkpoint handoff directory defined in
  `workflows/session-resilience.md` remains accessible under an explicit,
  narrow rule
- more-specific rules follow the wildcard so the intended rule wins
- no broad allowance is granted for the entire home directory, all projects,
  or the whole filesystem
- reviewer and auditor agent permission restrictions remain in effect and
  are not widened by an inherited wildcard

Report the actual configured values. Do not infer them from policy text, and
do not modify the client configuration during validation.

## 5. Validate Policies

Confirm consistency across the Core, Skills, commands, and workflows:

- Ask for the branch name before implementation when not supplied.
- Preserve uncommitted changes.
- Use the correct base branch.
- Do not create a PR or publish review comments remotely.
- Generate copy-ready PR and review content.
- Keep review workflows read-only.
- Select architecture and design patterns from evidence and the task's
  design forces, never from a quota or a stack.
- Support direct URLs and local artifacts.
- Treat MCP as optional.
- Keep credentials out of version control.

## Policy Conflict Detection

Inspect active instructions, not only headings and file references.

Verify that:

- provider-policy.md contains exactly one Cost Classification section
- provider-policy.md and provider-matrix.md use the same cost taxonomy
- TDD does not authorize deleting existing work
- TDD does not require fixing unrelated test failures automatically
- TDD test requirements are proportional to task complexity and risk
- the external Skill registry accurately documents local modifications
- /system-check validates the Engineering System, not application code
- /validate validates the current software project, not the system configuration
- validation commands are discovered from actual project configuration
- no universal test runner or coverage threshold is assumed
- validation scope is proportionate to task complexity and risk
- missing dependencies are reported rather than installed automatically,
  except for the mandatory formatting and linting setup
- validation does not modify application behavior; the only permitted
  source mutation is automated formatting of files within the declared
  task scope per the mandatory formatting and linting policy
- potentially destructive commands require appropriate authorization
- formatting and linting are configured and treated as mandatory
- missing formatter or linter tooling is reported as SETUP_REQUIRED,
  never as NOT_APPLICABLE
- workflows authorize the mandatory formatter and linter setup instead of
  unconditionally forbidding dependency installation and configuration
- AGENTS.md, the testing Skill, and the validation workflows agree on the
  formatting and linting policy
- the formatting and linting policy is scoped to the application code of
  software projects the system works on, and does not require converting
  documentation-only knowledge repositories into software projects
- no active instruction requires creating a package.json or installing
  formatter or linter tooling in the canonical ai-engineering repository
  solely to satisfy the policy
- reproducible lint, format, and format-check commands are required, not
  optional
- no active instruction claims validation passed without evidence or
  treats a successful build as a substitute for formatting and linting
- the required-file inventory above matches the actual maintained set of
  canonical files
- `agents/orchestrator.md` delegates the detailed task and PR-review
  sequences to the canonical workflows and does not define a parallel,
  contradictory order
- `/task` continues from the first incomplete phase of the canonical
  Task-to-Code workflow and does not repeat analysis already completed
  before branch preparation

## Review Quality and Evidence

Verify the Pull Request review workflow maintains evidence quality and
remains read-only and copy-only:

- every finding requires supporting evidence and an actionable explanation
- file references and line ranges are grounded in the reviewed revision,
  not assigned without inspection
- severity is proportionate to demonstrated impact and is not inflated for
  sensitive domains alone
- confirmed defects are separated from observations, questions, and
  design-review items
- findings are scoped to the requested change unless a broader audit is
  explicitly requested
- test or command execution is not claimed without an actual result
- /review-pr is read-only and copy-only
- no remote comments, approvals, merges, commits, or pushes are authorized
  by the review workflow
- the review workflow respects provider access and authorization rules

Distinguish active instructions from quoted examples or explanatory text.

If an active instruction conflicts with AGENTS.md, report POLICY CONFLICT.

A successful file-existence check alone does not establish policy
consistency.

## Session Resilience and Checkpoint Safety

Verify that checkpoint and recovery behavior is defined once and remains
safe:

- `workflows/session-resilience.md` is the single canonical definition
- `/checkpoint` and `/resume-work` delegate to that workflow instead of
  duplicating its logic
- checkpoints are written to a provider-neutral user-level handoff directory
  outside the target project's source tree, using the same state root as
  self-learning and no single-client path
- checkpointing does not modify source files, commits, branches, or remote
  state
- a checkpoint never overwrites a different task's checkpoint
- checkpoints never store credentials, tokens, or secrets
- recovery verifies the actual repository, branch, HEAD, and working-tree
  state instead of trusting the checkpoint as proof
- recovery treats the checkpoint as a guide, detects divergence, preserves
  current work, and continues from the first incomplete action instead of
  repeating completed phases
- uncommitted changes are preserved during both checkpointing and recovery
- context continuity across `/compact` and `/new` is preserved without
  requiring the terminal to be closed and reopened
- no instruction claims a prior validation passed without recorded evidence
  or re-execution
- automatic compaction remains enabled and no undocumented context-threshold
  configuration keys are introduced
- no active instruction requires closing and reopening the terminal as the
  normal recovery path
- no experimental compaction plugin is required by the policy
- command references in AGENTS.md and README.md match the actual command
  files

### Checkpoint Identity and Duplicate Detection

Verify checkpoint slug and overwrite rules statically:

- the literal handoff directory path is defined in exactly one place,
  `workflows/session-resilience.md`
- the provider-neutral state root is used by both the handoff directory and
  the self-learning store, with no client-specific path such as a fixed
  OpenCode, Claude, Codex, or Aider directory
- AGENTS.md delegates to that workflow instead of repeating the literal
  path, and `/checkpoint` and `/resume-work` reference the workflow's path
  without duplicating it
- checkpoint identity uses the task identifier or a generated checkpoint
  identifier and never relies on the branch name alone
- task and checkpoint identifiers are normalized into safe file names:
  path separators, `.`/`..` components, and other unsafe characters are
  rejected before any path is derived
- `/checkpoint` resolves the final checkpoint path and verifies it stays
  inside the authorized handoff directory, rejecting identifiers, paths,
  or symlinks that would escape it
- `/resume-work` applies the same normalization when resolving a supplied
  task identifier or checkpoint path
- no two checkpoint-writing implementations exist; `/checkpoint` and
  `/resume-work` delegate to the workflow and do not duplicate its logic
- `/checkpoint` rejects checkpoint paths outside the authorized handoff
  directory
- `/resume-work` lists multiple matching checkpoints and asks for a
  selection instead of picking one by branch name
- no instruction authorizes creating or updating another task's checkpoint
- checkpoint contents require a task identifier or checkpoint identifier,
  so two tasks on the same branch remain distinguishable

## 6. Reference Integrity

Inspect references to Skills, workflows, contracts, agents, and commands.

Identify:

- referenced but missing files
- inconsistent filenames
- obsolete command names
- contradictory lifecycle instructions
- duplicate canonical instructions
- official-documentation.md contains documentation references for all
  supported technology families
- skill-catalog.md contains a decision for every supported technology
- every installed external Skill appears in external-skill-registry.md
- candidate Skills are not reported as installed
- installed external Skills have verified licenses and recorded source
  revisions
- `commands/delivery-check.md` and `commands/provider-check.md` reference
  existing workflows and contain no deployment, publish, or remote-write
  instruction for their read-only mode
- `workflows/secure-delivery.md` references only existing commands,
  workflows, and policies
- `references/system-workflow-regression-matrix.md` references only
  existing workflows, commands, and policies and does not restate a
  duplicated literal handoff path
- `skills/implementation-recipes/SKILL.md`,
  `skills/api-contract-design/SKILL.md`, and
  `skills/database-engineering/SKILL.md` exist and are referenced
  consistently by `agents/orchestrator.md`, `workflows/task-to-code.md`, and
  `README.md`; they do not duplicate the architecture decision policy owned
  by `skills/architecture-advisor/SKILL.md`
- `workflows/self-learning.md` exists and is referenced consistently by
  `AGENTS.md`, `agents/orchestrator.md`, and `README.md`; it defines the
  neutral state root once and never duplicates the handoff path or a
  client-specific directory
- `workflows/estimation.md` and `commands/estimate.md` exist and are
  referenced consistently by `AGENTS.md`, `agents/orchestrator.md`,
  `workflows/task-to-code.md`, and `README.md`; estimation is described as a
  standalone activity scoped to complete projects from start to finish (never
  to tickets, stories, or tasks) that never implements, and
  `commands/estimate.md` delegates to the workflow instead of restating its
  procedure
- `skills/internationalization/SKILL.md` exists and is referenced
  consistently by `agents/orchestrator.md`, `workflows/task-to-code.md`,
  `commands/task.md`, `skills/implementation-recipes/SKILL.md`, and
  `README.md`; localization is settled early for work with user-facing text
  and the Skill is not loaded for work without it

Do not flag a reference as broken merely because a file is intentionally
provided by the client rather than by the canonical repository.

### 6.0 Executable structural checks

These checks are the structural audit of the whole repository. They are
executed by every run of `workflows/system-audit.md` (its Phase 3) and by
`/system-check`. A check reports `PASS`, `FAIL`, or `NOT VERIFIED` with the
reason and the evidence. Run them over the whole repository, not only the
changed files.

**Verdict rule.** Several blocks below report a finding as a line that begins
with `FAIL` and do not always set a non-zero exit status, so the exit status
is not evidence on its own. The verdict of a block is read from its output:
any line beginning with `FAIL` is a `FAIL`; no output with a successful
status is a `PASS`; a block that cannot run is `NOT VERIFIED` with the reason.

### 6.1 Reference integrity

Every local Markdown link resolves, and every repository file path in
backticks exists. A reference to an artifact installed in a target project,
such as an ESLint configuration, is not a failure.

A path cited precisely **because it must not exist** is a negative
assertion, not a broken reference. It is a `FAIL` when the file exists, and
a `PASS` when it does not. Read the sentence before classifying it.

```sh
grep -rhoE '\]\([^)#[:space:]]+\.md\)' --include='*.md' . |
  sed 's/^](//; s/)$//' | sort -u
```

**The version guide is consulted.** The **version guide** of the ecosystem
is the source of truth for what is current. An audit that checks a
framework-specific convention confirms that the guide was consulted for the
target version, and that the consulted source covers it.

**Pinned sources are registered.** A source that documents one specific
release can silently become stale. Every URL that contains a version number
must appear in the **Version-Pinned Sources** table of
`references/official-documentation.md`, with its version and its review
rule. An unregistered pinned URL is a `FAIL`: it can age without anyone
noticing.

```sh
grep -oE 'https?://[^ )]+' references/official-documentation.md |
  tr -d '`' | sed 's/[.,]$//' | sort -u |
  grep -E '/v?[0-9]+\.[0-9]+|version-[0-9]+|/en/[0-9]+\.[0-9]+/' |
  while read -r url; do
    grep -qF "$url" references/official-documentation.md ||
      echo "FAIL pinned source not registered: $url"
  done

# The extraction above can carry trailing markdown; compare on the URL alone.
# Compare against the pinned table, not against the whole document.
# No temporary file: the table is piped straight into the comparison.
pinned_table() {
  awk '/^## Version-Pinned Sources/{found=1} found' \
    references/official-documentation.md
}

grep -oE 'https?://[^ )[:space:]]+' references/official-documentation.md |
  tr -d '`' | sed 's/[.,]$//' | sort -u |
  grep -E '/v?[0-9]+\.[0-9]+|version-[0-9]+|/en/[0-9]+\.[0-9]+/' |
  while read -r url; do
    pinned_table | grep -qF "$url" ||
      echo "FAIL pinned source not registered: $url"
  done
```

**External URLs resolve.** A documentation reference that points nowhere
makes the consultation mandate unsatisfiable, and the agent falls back to
guessing. Check every URL in the changed files, and every URL in
`references/official-documentation.md` when that file is touched:

```sh
grep -rhoE 'https?://[^ )[:space:]]+' references/official-documentation.md |
  tr -d '`' | sed 's/[.,]$//' | sort -u | while read -r url; do
    code=$(curl -s -o /dev/null -w "%{http_code}" -L --max-time 20 "$url")
    [ "$code" = "200" ] || echo "FAIL $code $url"
  done
```

A status that is not `200` is a finding to investigate. A `403` from a known
host is bot protection rather than a dead link, and a `000` is a transient
failure; record those as `NOT VERIFIED` with the reason instead of silently
passing them.

When the environment has no `curl`, the URL check is `NOT VERIFIED` with the
tool named. It is never a pass. The same applies to any other tool the
environment lacks.

### 6.2 Command catalog

Every command in `commands/` has its own description in the README, so the
catalog does not drift from the commands that exist.

```sh
for command in commands/*.md; do
  name=$(basename "$command" .md)
  grep -q "^- \`/$name\`:" README.md ||
    echo "FAIL command /$name has no description in README.md"
done

for name in $(grep -oE '^- `/[a-z-]+`:' README.md | sed 's/^- `//; s/`:$//'); do
  [ -f "commands/$name.md" ] ||
    echo "FAIL README describes /$name but commands/$name.md does not exist"
done
```

A command without a README description is a `FAIL`: the catalog a reader
consults is incomplete. A README entry without a command is a `FAIL` too: it
is a dangling reference left by a rename, and a reader would invoke a
command that does not exist.

### 6.3 Portability

The system must work on every client and operating system. The shared
procedures assume a **POSIX shell**; when the environment does not provide
one, the check is `NOT VERIFIED` with the reason. Verify:

- Every workflow is reachable **without** the `commands/` adapters, so a
  client without slash commands still reaches it.
- No shared procedure contains any of the **non-portable constructs**
  listed in `references/portability.md`. That list is the single source;
  it is not repeated here.
- No shared procedure hardcodes an operating-system path or a client
  configuration directory.

```sh
# Workflows reachable without the commands/ adapters
for workflow in workflows/*.md; do
  name="workflows/$(basename "$workflow")"
  grep -rl "$name" --include='*.md' . |
    grep -v '^./commands/' | grep -q . ||
    echo "FAIL $name is only reachable from commands/"
done

# Non-portable constructs in shared procedures.
# The patterns live in references/portability.md, so they are defined once.
# The character classes keep this pattern from matching itself.
grep -rnE '/[t]mp/|/var/[f]olders/|~?/\.[c]onfig/opencode|~?/\.[c]laude' \
  workflows/ references/ skills/ commands/ 2>/dev/null |
  grep -vE 'references/+portability\.md:' |
  grep -v 'README.md' |
  sed 's/^/  /'
```

A hardcoded path is a `FAIL` when it appears in a shared procedure, and a
`PASS` when it appears in the setup instructions, which are per environment
by nature.

### 6.4 Index, inventory, and numbering

- The `V2.x` index and the `V2.x` sections match exactly: no orphan entry
  and no unindexed section.
- Every file in the required-file inventory exists.
- The `R-nn` identifiers have no gap and no duplicate.

### 6.5 Rule reachability and cross-client reach

For every row of `references/mandatory-rules.md`, the anchor is present on
every required surface, and the canonical wording is present in `AGENTS.md`.
`AGENTS.md` is the only file every client reads, so a rule absent from it
does not reach every client.

```sh
normalize() { tr '\n' ' ' | tr 'A-Z' 'a-z' | tr '-' ' ' | tr -s ' ' ' '; }
trim() { sed 's/`//g; s/^[[:space:]]*//; s/[[:space:]]*$//'; }

# The findings are captured so that the exit status reports them: a FAIL line
# inside the pipeline would otherwise leave the status at 0.
reachability_failures=$(grep -E '^\| M-[0-9]+ \|' references/mandatory-rules.md |
while IFS='|' read -r _ id _ _ anchor surfaces; do
  id="$(printf '%s' "$id" | trim)"
  anchor="$(printf '%s' "$anchor" | trim)"
  normalized_anchor="$(printf '%s' "$anchor" | normalize)"
  printf '%s\n' "${surfaces%%|*}" | tr ',' '\n' | while read -r surface; do
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
done)
printf '%s\n' "$reachability_failures" | sed '/^$/d'
[ -z "$reachability_failures" ]
```

### 6.6 Retired wordings and duplication

**A retired wording must not come back.** The registry lists the superseded
phrasings in `references/mandatory-rules.md` ("Superseded Phrasings"). Each
one is a rule that changed, and its previous wording must not reappear in the
canonical content.

```sh
awk '/^## Superseded Phrasings/{found=1; next} /^## /{found=0} found' \
  references/mandatory-rules.md |
  grep -E '^\| [^-|]' | tail -n +2 | cut -d'|' -f2 |
  sed 's/^ *//; s/ *$//' |
  while read -r retired; do
    [ -z "$retired" ] && continue
    if grep -rqiF "$retired" --include='*.md' . \
      --exclude='mandatory-rules.md'; then
      echo "FAIL retired wording is back: $retired"
    fi
  done
```

The check reads the retired wordings from the registry, so it never drifts
from it, and it excludes the registry itself, where they are recorded. A retired wording that reappears is a `FAIL`: the system would
apply a rule that no longer exists.

### 6.7 Duplication and contradiction (judgment)

- Two files must not carry the same normative wording. A surface that
  restates a rule is a drift risk; it must reference the canonical source.
- Two files must not carry incompatible wording of the same rule. Search the
  superseded phrasings when a rule changed, and confirm each is gone.
- A command must not copy a block from the workflow it delegates to.

## 7. Provider State

Distinguish:

- supported by the architecture
- documented as a candidate
- locally configured
- successfully tested

Do not claim a provider works merely because its name appears in a table.

Do not print credentials or secret environment-variable values.

### Provider Integration Proof

Verify against `workflows/provider-integration-proof.md`,
`integrations/provider-policy.md`, `integrations/access-strategy.md`,
`integrations/provider-matrix.md`, and the provider contracts that:

- provider statuses are supported by observed evidence
- simulation is never reported as real verification
- authentication and read verification are distinguished
- read and write capabilities are tracked independently
- no remote write operation is performed by the verification workflow
- credentials and secrets are not exposed
- absent optional providers do not incorrectly fail the provider-agnostic
  core
- `/provider-check` references the canonical workflow
  (`workflows/provider-integration-proof.md`) and respects read-only agent
  permissions
- provider policies remain consistent with the contracts and the access
  strategy
- provider verification never triggers unapproved paid usage or installs
  unnecessary integrations
- the documented commands and the required-file inventory remain correct
- the required-file count is derived from the actual inventory, never a
  hardcoded total

## 7.1 Secure Delivery and CI/CD Safety

Verify against `workflows/secure-delivery.md`, `commands/delivery-check.md`,
`workflows/project-validation.md`, and the existing command boundaries that:

- `/delivery-check` references `workflows/secure-delivery.md` and respects
  read-only agent permissions
- `secure-delivery.md` distinguishes read-only assessment, authorized
  implementation, execution of tests and scans, and publication or
  deployment that requires explicit authorization
- `/system-check`, `/validate`, `/provider-check`, and `/delivery-check`
  keep distinct responsibilities with no duplicate canonical policy
- tooling and CI decisions are project-specific and evidence-based rather
  than universal defaults or familiar-platform bias
- the mandatory software-project formatter/linter policy in AGENTS.md is
  unchanged and remains intact in the delivery workflow
- security checks never print, reproduce, or store discovered secret
  values; sensitive evidence is redacted to location and type
- configured checks and actually executed or passed checks are clearly
  distinguished
- a clean dependency audit is reported only when the audit command
  executed successfully and its vulnerability database was available
- simulated scans are never reported as real security results
- absent optional CI/CD platforms or providers are reported as
  NOT_CONFIGURED and do not incorrectly fail the provider-agnostic core
- no unapproved installations, billable integrations, remote writes,
  deployments, or publisher actions are authorized by the delivery
  workflow
- the delivery workflow never weakens required checks or security
  controls to obtain a green pipeline
- declarations of pipeline security or production readiness require
  supporting evidence, not static review alone
- NOT_APPLICABLE is used only when a control genuinely does not apply;
  missing in-scope mandatory tooling remains SETUP_REQUIRED
- a read-only assessment is not silently turned into implementation work
- pipeline authoring is an authorized mode distinct from the read-only
  `/delivery-check`; it promotes one artifact across environments, keeps
  environment differences out of the artifact, runs the required validation
  gates before every deployment, requires an explicit approval gate for
  production, and never weakens gates to obtain a green pipeline
- a failing pipeline is handled by proposing a bounded fix or a rollback,
  never by silently bypassing or disabling gates
- the required-file count is derived from the actual inventory, never a
  hardcoded total

## 7.2 Dependency Decision and Approval

Verify against `workflows/dependency-assessment.md`, AGENTS.md, the task
workflows, `commands/task.md`, and `skills/stack-detection/SKILL.md` that:

- every software implementation task performs the dependency assessment
  before its implementation plan is finalized
- the dependency assessment is defined once, canonically, in
  `workflows/dependency-assessment.md`, and other files reference it
  instead of duplicating its decision policy
- exactly one dependency outcome is recorded: `NO_NEW_DEPENDENCY`,
  `REUSE_EXISTING`, `NEW_DEPENDENCY_PROPOSED`, or `BLOCKED`
- existing dependencies are checked before a new one is recommended, and a
  dependency-free option is considered where reasonable
- a new-dependency recommendation includes evidence-based tradeoffs and
  does not invent versions, security status, or maintenance claims
- installation of a new runtime or development dependency requires explicit
  authorization, except for the established mandatory formatter/linter setup
- a general task approval is not treated as blanket authorization to
  introduce libraries
- a user refusal prevents installation, and the no-approval path does not
  silently change the task scope
- approved installation uses the project's package manager, updates the
  manifest and lockfile together without upgrading unrelated dependencies,
  and reports actual results
- an installation failure is reported rather than silently substituting a
  different dependency
- package manifests and lockfiles remain consistent
- no credentials, registry tokens, paid usage, or billable integrations are
  introduced by the assessment or installation
- the documentation-only ai-engineering repository is not converted into a
  software package to implement this workflow
- no new slash command was added unless a demonstrated use case required one

## 7.3 Orchestrator and Workflow Sequence Consistency

Verify against `agents/orchestrator.md`, `workflows/task-to-code.md`,
`workflows/pull-request-review.md`, and `commands/task.md` that:

- `agents/orchestrator.md` delegates the detailed, ordered task sequence to
  `workflows/task-to-code.md` instead of maintaining a parallel sequence
- the orchestrator project lifecycles do not contradict the canonical
  workflow's branch-preparation, analysis, planning, and implementation
  order
- the orchestrator delegates PR-review sequencing to
  `workflows/pull-request-review.md`
- `/task` follows the canonical Task-to-Code sequence, continues from the
  first incomplete phase, and reuses completed analysis instead of repeating
  task, design, repository, or stack inspection
- `workflows/task-to-code.md` defines the technical complexity and
  architecture evaluation in exactly one phase, reuses a completed result
  instead of repeating the affected-modules, architecture, pattern, and risk
  checklist, and extends it only when new evidence justifies it; the
  post-implementation architectural validation in the review phase is
  distinct from that evaluation and does not duplicate it
- architecture-only and estimation-only requests remain free of branch
  preparation, and software implementation still requires branch preparation
  before modifying application code
- no file in the core execution path defines a competing ordering of the
  lifecycle

## 7.4 System Workflow Regression Matrix

Verify that `references/system-workflow-regression-matrix.md` remains a
lightweight reference and not a competing procedure:

- it exists at its canonical path and references only existing workflows,
  commands, and policies
- it is explicitly described as a reference, not a mandatory procedure for
  every task, and it adds no required acceptance phase
- it states that scenarios are selected according to the files, policies, and
  behaviors a change affects, rather than run wholesale by default
- it states that it does not define a closed architecture, a fixed process
  order, or a new layer
- it reserves broader integral validation for significant changes: the
  orchestrator, permission boundaries, or interaction between several
  workflows
- it protects the critical transversal behaviors: the branch policy,
  preservation of uncommitted work, the dependency approval gate and refusal
  path, the mandatory formatter and linter scope together with its tool
  lifecycle states, honest validation reporting, provider and delivery
  read-only boundaries, PR and review copy-only character, checkpoint and
  recovery safety, contract-before-implementation, the quality-not-proportional
  baseline, database safety and destructive-operation gates, delivery pipeline
  authoring with environments and pre-deploy gates, provider-neutral
  self-learning with promotion gates, bounded context spend, standalone,
  project-scoped, and ranged estimation, early localization with correct implementation, mobile
  analysis and implementation, and proportional decision records
- REAL, SIMULATED, and STATIC evidence are defined and never conflated
- it does not duplicate the decision policy of the task, dependency,
  validation, provider, delivery, review, or session-resilience workflows,
  and references them instead
- it does not require creating a package.json, formatter or linter tooling,
  or a testing framework in the canonical repository
- no new command was introduced solely to host the regression matrix
- no active instruction in the regression matrix contradicts AGENTS.md or a
  canonical workflow

## 8. Output

Return:

### SYSTEM VALIDATION

- Canonical root:
- Client:
- Required files:
- Missing files:
- Invalid references:
- Policy contradictions:
- OpenCode discovery:
- Codex CLI instruction loading:
- Codex CLI behavioral execution:
- Provider configuration status:
- Delivery and CI/CD state:
- Dependency decision state:
- License status:
- External Skill registry:
- External Skills installed:
- External Skill licenses:
- Upstream commit verification:
- External Skill policy conflicts:
- System workflow regression matrix:
- V2.5 interactive autonomy and automatic task interpretation checks:
- V2.6 architecture-first checks:
- V2.7 architecture decision gate checks:
- V2.8 architecture catalog and evidence boundary checks:
- V2.9 automatic task interpretation checks:
- V2.12 web quality checks:
- V2.13 proportional planning checks:
- V2.14 formatter/linter setup checks:
- V2.15 contract and recipe checks:
- V2.16 database engineering checks:
- V2.17 delivery pipeline authoring checks:
- V2.18 neutral state root and single orchestrator checks:
- V2.19 estimation checks:
- V2.20 internationalization and mobile UI checks:
- V2.21 refactoring and abstraction justification checks:
- V2.22 code audit checks:
- V2.23 folder architecture and type ownership checks:
- V2.24 commit generation checks:
- V2.26 folder architecture by complexity, dumping ground prohibition, and reuse-based type ownership checks:
- V2.27 commit approval checks:
- V2.28 primary unit per file, unit folder, and test co-location checks:
- V2.29 target project and evidence-only path checks:
- V2.30 minimum sufficient context and token-efficiency checks:
- V2.31 structural compliance verification checks:
- V2.32 mandatory rule reachability checks:
- V2.33 new project initialization checks:
- V2.34 end-to-end audit checks:
- V2.35 progressive project creation checks:
- V2.36 target version and official documentation checks:
- V2.37 documentation freshness checks:
- V2.38 self-learning execution checks:
- V2.39 portability checks:
- V2.40 feature-driven structure checks:
- V2.41 folder architecture checks:
- V2.42 topology decision checks:
- V2.43 source grounding checks:
- V2.44 architecture matrix checks:
- V2.45 installation gate checks:
- V2.46 phase checklist checks:
- V2.47 phase plan, phase checklist, and phase gate checks:
- V2.48 feature responsibility folders checks:
- V2.49 design-pattern selection checks:
- V2.50 external audit checks:
- V2.51 engineering baseline checks:
- V2.52 design quality and dependency direction checks:
- V2.53 Clean Architecture mapping and unit style checks:
- V2.54 live phase checklist and native task surface checks:
- V2.55 entry-point independence and operating contract checks:
- V2.56 arrow function scope and legacy code checks:
- V2.57 delegated work mirrored into the delegating panel checks:
- V2.58 new task starts a new list checks:
- V2.59 discipline reinforcement behavioral scenario checks:
- V2.60 audit criteria, implementation rules, and decision record checks:
- V2.61 selected pattern presentation and SOLID coverage checks:
- V2.62 stage is one concern and concern measurement checks:
- V2.63 styling approach by framework and timing checks:
- V2.64 audit criteria read gate and platform criteria checks:
- V2.65 test API import and test-case form checks:
- V2.66 unwired control, dead export, decision record, and modified-test scope checks:
- V2.67 contract duplication, scan coverage, probe isolation, and comment accuracy checks:
- V2.68 identity, health probe, and audit signal checks:
- V2.69 delivery safety checks: timeouts, cleartext, client cache, downgrade, hardcoded endpoints, temporary paths, and probe dead exports:
- V2.70 path encoding, rejected credentials, backup, and whole-project sweep checks:
- V2.71 cross-origin, development tools, owner-only writes, and absence claim checks:
- V2.72 visible failures, verified claims, test-only consumers, entry identity, and composition checks:
- V2.73 locale direction, consumed-code inheritance, enumerated claims, decision-record patterns, and platform live-region checks:
- V2.74 deploy order, application-side schema ownership, and scope-manifest checks:
- V2.75 inventory enumeration, repository state, and unexecuted-step checks:
- V2.76 performance claims, technical-property claims, and sweep coverage checks:
- V2.77 detector coverage in the change-set procedure, and the AMBIGUOUS and evidence rules of the code audit:
- V2.78 controlled clocks, edge and error coverage, design-record claims, and scope values (M-76 to M-79):
- V2.79 commit-group closure, bound tests, comment claims, and citation resolution:
- V2.80 verdict matrix consistency, legacy records, and runtime equivalence (M-80):
- V2.81 typed errors, universal and cost claims, and the detectors' coverage (M-81):
- V2.82 project-wide error model, error codes in both directions, entry-point tests, validation order, read transactions, and retry budgets (M-54, M-77, M-81 to M-84):
- V2.83 structural gate in the commit workflow, pre-existing test findings, consumer groups, citations outside tables, index coverage, permission grants, scope of commits, performance metrics, and the teardown in the plan (M-08, M-47, M-78, M-85 to M-89):
- V2.84 untracked scope and missing manifest, analysis-only writes, approval quotes, teardown gate, and verdict-status enumeration (M-80, M-90 to M-93):
- V2.85 native task mirror at stage level: stage start and end rewrites, the `Stage:` marker, and the panel completeness check (M-35, R-156):
- External Skill readiness:

The numbering skips V2.9 to V2.11: those identifiers were never used, and
no rule, section, or reference depends on them. V2.25 was removed when its
checks were folded into V2.26 and V2.28; its index entry was left behind
and has been removed.

### Findings

Classify:

- P0: unsafe configuration or destructive risk
- P1: core workflow broken
- P2: important inconsistency
- P3: optional improvement

### Recommended Actions

Provide the smallest changes that resolve actual findings.

## Safety

This workflow is read-only.

Never create, delete, overwrite, commit, push, or publish anything.
Never expose secrets.

Report checks as NOT VERIFIED when the available tools cannot establish
their result.

## Documentation and Skill Selection

Verify that:

- technology and version detection precedes documentation selection
- only relevant documentation sources are retrieved
- existing project tooling is preferred
- candidates are not automatically installed
- repository-specific Skills are not treated as universal engineering rules
- external Skills cannot override AGENTS.md
- the absence of a dedicated framework Skill is not reported as a failure
  when official documentation provides adequate guidance

Verify that every technology family in
references/official-documentation.md has a corresponding decision in
integrations/skill-catalog.md.

A decision may be DOCS_ONLY, CANDIDATE, TASK_SPECIFIC, INSTALLED,
REJECTED, or REASSESS.

Do not require a dedicated external Skill when official documentation
is sufficient.

Report a missing catalog entry as P3 unless it causes an actual workflow
failure.

Also verify that Markdown code fences are balanced in the Skills,
workflows, and references. An unbalanced fence changes which instructions
are interpreted as text or as code, so treat it as a defect even when the
content remains readable.

Report an unbalanced fence in a Skill that is actively loaded as P2, and
in other documentation as P3.

## V2.5 Regression Coverage

Verify the following controls exist and are not contradictory:

- The user is not asked to make routine implementation decisions.
- The agent resolves discoverable technical choices independently.
- Material business ambiguities trigger concise clarification.
- The implementation approach is aligned before code changes.
- Routine code adjustments and validation proceed without repeated approval requests.
- Significant architectural decisions and new dependencies retain their existing authorization gates.
- Architecture-only and estimation-only requests do not silently generate application code.
- Status updates report progress but do not create artificial approval gates.
- The agent reuses previously completed analysis after alignment.
- Client-specific interaction features are used only where supported.
- Architecture-only and estimation-only requests do not require branch preparation or Git state changes unrelated to the requested analysis.
- Software implementation requests still require branch preparation before modifying application code per the branch policy.
- No conflicting rules have been introduced in AGENTS.md, the orchestrator, the task workflow, or commands.

## V2.9 Refinement — Automatic Task Interpretation

Verify the following controls exist and are not contradictory across
`AGENTS.md`, `agents/orchestrator.md`, `commands/task.md`,
`workflows/task-to-code.md`, and `skills/task-context/SKILL.md`:

- a short, ordinary task description is accepted as a complete task input,
  with no requirement to rewrite it into an elaborate prompt
- short tasks trigger repository-aware interpretation: the repository,
  code, conventions, configuration, tests, and designs are inspected
  before technical questions are asked of the user
- explicit task requirements and inferred requirements are distinguishable,
  and inferred requirements are never presented as explicit task text
- acceptance criteria are derived from task and repository evidence and
  are validated with the user only when a material ambiguity exists
- missing business or product decisions trigger selective, one-at-a-time
  questions instead of guesses
- the user is not asked to author or maintain a separate Markdown task
  specification, and no Markdown artifact is produced for the internal
  working interpretation by default
- the working interpretation is persisted only through the existing
  checkpoint mechanism when continuity requires it, or when the user
  requests a durable artifact
- routine implementation details remain autonomous after alignment
- branch preparation, dependency approval, architecture approval,
  security, and validation gates remain intact and unchanged
- repeated repository analysis is avoided by reusing completed analysis
- architecture-only and estimation-only requests do not automatically
  modify application code
- interpretation extends the existing task-context Skill and Task-to-Code
  workflow; no duplicate canonical workflow, task-transcription subsystem,
  or task-interpreter command was introduced
- task retrieval that fails never results in a claim that inaccessible
  task content was read; only the minimum missing information is requested

## V2.6 Regression — Architecture-First Engineering

Verify that architecture is evaluated before implementation and validated
after it, across frameworks, languages, and application types:

- `workflows/task-to-code.md` performs the architecture evaluation before
  the implementation phase and does not implement first
- the architecture evaluation covers code organization and placement,
  cohesion, reuse, module conventions (public exports, barrels, import
  aliases), design patterns selected from the task's design forces, and
  styling decisions
- the architecture knowledge is defined once in
  `skills/architecture-advisor/SKILL.md`; no duplicate architecture Skill
  was created and no competing order was introduced
- `skills/engineering-review/SKILL.md` and the review phase require a
  post-implementation architectural validation that contrasts the result
  with the proposed design and the project's conventions, in addition to
  functional tests
- the rules are framework- and language-neutral: no universal folder
  structure, fixed file count, or preferred pattern count is imposed
- a styling-system decision that significantly affects architecture,
  dependencies, or the visual experience triggers a human decision gate
- routine technical details remain autonomous and are not turned into
  approval gates
- the architecture evaluation remains in exactly one phase, and the
  post-implementation validation is distinct from it
- `skills/project-analysis/SKILL.md` detects module and import conventions
  without introducing new ones during analysis

## V2.7 Regression — Architecture Decision Gate

Verify that the architecture diagnosis is evidence-based and that the
mandatory decision gate exists and is not contradictory across
`skills/architecture-advisor/SKILL.md`, `skills/project-analysis/SKILL.md`,
`workflows/task-to-code.md`, `commands/task.md`, `agents/orchestrator.md`,
and `references/system-workflow-regression-matrix.md`:

- the architecture knowledge remains defined once in
  `skills/architecture-advisor/SKILL.md`; no technology-specific Skill and no
  new document directory was created
- the canonical Skill requires inspecting real structure, entry points,
  imports, boundaries, stack-relevant elements, configuration, tests, and
  existing documentation before proposing a solution
- the diagnosis explicitly distinguishes detected with evidence, inferred
  with uncertainty, and no identifiable architecture, and states that
  finding `components`, `services`, or `lib` directories does not by itself
  establish an architecture
- the Skill separates architectural style, organization strategy, and
  language/framework conventions, and does not impose hexagonal, Clean,
  feature, microservice, or any other option universally
- the Skill defines the default rule to separate reusable types,
  interfaces, and contracts from their consumers, organized by
  feature/domain/module, with justified exceptions and respect for language
  conventions
- the Skill covers responsibility separation per stack without requiring
  empty folders or a fixed directory set
- the Skill and `workflows/task-to-code.md` define the mandatory decision
  gate: continue a coherent existing architecture, or present the diagnosis
  with a recommendation, concrete alternatives, and trade-offs, and wait for
  the user's decision before implementing; a recommendation is never
  silently treated as approved
- `commands/task.md` and `agents/orchestrator.md` delegate to that gate
  instead of defining a competing order
- the post-implementation validation requires concrete evidence and
  distinguishes documentation consistency of `ai-engineering`, compliance
  with the flow before implementing, and architectural quality of the
  resulting code; it does not claim the agent's behavior is proven by
  verifying instructions alone
- the Skill carries compact references to recognized architectural sources
  without creating a Skill per technology
- the V2.7 checks do not weaken the existing branch, dependency,
  validation, provider, delivery, review, or session-resilience gates
- `skills/project-analysis/SKILL.md` reports the confidence of the
  architecture classification explicitly

## V2.8 Regression — Architecture Catalog and Evidence Boundaries

Verify that the architecture catalog offers alternatives for the ecosystems
actually supported by `skills/stack-detection/SKILL.md`, without turning
them into a universal folder structure or architecture requirement:

- the canonical Skill distinguishes application architectural style, code
  organization, and language/framework conventions
- alternatives are documented for supported languages and frameworks and
  are selected only after identifying the actual stack
- architecture is detected from implementation evidence; when no coherent
  architecture is identified or evidence conflicts, the flow presents
  appropriate alternatives and trade-offs and waits for approval on an
  important unresolved decision
- guidance integrates reusable types, interfaces, contracts, components,
  services, hooks/composables, contexts/providers, and adapters where
  relevant, without requiring universal directories or empty folders
- the catalog remains canonical in the existing architecture-advisor Skill;
  no duplicate architecture document or per-framework architecture Skill is
  introduced
- `/system-check` validates consistency and completeness of the written
  catalog and references only; it does not claim to execute `/task`, prove
  agent behavior, inspect an arbitrary application's architecture, or
  validate application code behavior
- documentation consistency, flow compliance, implemented architecture
  quality, and functional behavior remain separate findings

## V2.12 Regression — Web Quality Skills

Verify that the imported web quality Skills remain faithfully vendored,
honest about their evidence, and correctly gated:

- `skills/accessibility/`, `skills/seo/`, `skills/performance/`,
  `skills/core-web-vitals/`, and `skills/best-practices/` exist and each
  contains SKILL.md, its referenced supporting files, and LICENSE.txt
- every imported Skill is recorded in
  `integrations/external-skill-registry.md` with the pinned upstream commit
  `afa8da942115f2961fdbfa80807ea0b232ff6c00`, an MIT license, and documented
  local modifications, and each is listed as INSTALLED in
  `integrations/skill-catalog.md`
- the upstream `web-quality-audit` orchestration Skill and its
  `scripts/analyze.sh` are neither present nor referenced by any imported
  Skill; no second orchestrator was introduced
- field, lab, and hypothesis evidence remain distinct: a Lighthouse or lab
  score is never reported as WCAG conformance or as a prediction of
  rankings, and static findings are labeled as hypotheses
- SEO and Core Web Vitals checks are gated by application type and are
  reported as NOT_APPLICABLE for non-web deliverables rather than scored
- the absence of a heap-profiling workflow is documented and memory leaks
  are not claimed to be detected automatically
- Chrome DevTools MCP integration is described as optional, with documented
  non-MCP fallbacks; no imported Skill requires an MCP connection or an
  automatic installation
- the imported Skills do not override AGENTS.md or the branch, dependency,
  architecture, validation, provider, delivery, review, or
  session-resilience policies

## V2.13 Regression — Proportional Planning and SDD

Verify that planning is proportional, evidence-based, and integrated without
duplicating rules across files:

- `workflows/task-planning.md` exists at its canonical path, appears in the
  required-file inventory, and is invoked from Phase 9 of
  `workflows/task-to-code.md` without introducing a competing lifecycle
  order
- `skills/complexity-advisor/SKILL.md` classifies complexity and recommends
  a rigor level; it does not implement the planning procedure
- the planning procedure — levels and content per level — is defined once in
  `workflows/task-planning.md`, and the retained rigor level and artifact
  decision belong to `skills/task-context/SKILL.md`; no rule is duplicated
  across these files
- `agents/orchestrator.md` and `commands/task.md` delegate to the planning
  workflow instead of restating the levels or the approval gates
- a preliminary complexity estimate may be recorded as tentative during
  alignment; it does not fix the rigor level
- the definitive complexity classification (Phase 8) and the derived rigor
  level (Phase 9) are not fixed before sufficient evidence is available
- trivial and low tasks are not forced to produce a heavy specification, and
  a Markdown specification is not required by default
- medium and higher tasks define verifiable acceptance criteria, and complex
  or high-risk tasks define their expected behavior before implementing
- approval is triggered by the Human Decision Gates (risk, irreversible or
  high-impact operations, security or data, cost, scope, shared or
  production resources, project rules) and not by the classification label;
  a Critical task activates reinforced controls and requires approval before
  high-impact operations
- a durable planning artifact is created only for justified traceability,
  approval, or an explicit request; the default is session state, and no
  second specification of the code is produced
- `workflows/project-validation.md` and `skills/discovery/SKILL.md` remain
  unchanged and their existing behavior is preserved
- no duplicate canonical workflow, competing order, or second orchestrator
  was introduced

## V2.14 Regression — Formatter and Linter Setup and Execution

Verify that formatter and linter configuration and execution are concrete,
conflict-free, scoped, and reported honestly, with the procedure defined
once:

- `workflows/project-validation.md` defines the mandatory setup procedure
  once, in Phase 5, and remains the canonical procedure; no separate
  `workflows/tooling-setup.md` or second setup flow was created
- Phase 2 defines the tool lifecycle states — detected, configured,
  installed, executed, approved — and states explicitly that they are
  independent of the check-result statuses (`PASS`, `FAIL`,
  `SETUP_REQUIRED`, `BLOCKED`)
- Phase 5 detects before configuring, selects tools per the AGENTS.md
  defaults, configures for the detected framework and versions, prevents
  formatter/linter conflicts, scopes monorepo changes to the affected
  packages, and produces an explicit proposal when tooling is missing
- existing working configuration is preserved and a redundant second tool is
  not added
- the standard mandatory setup is authorized by policy without a redundant
  approval, while a paid tool, a non-standard dependency, or replacing a
  working configuration still requires a decision, and a real restriction is
  reported as BLOCKED rather than success
- `skills/stack-detection/SKILL.md` adds only detection and selection rules
  and delegates the procedure to `workflows/project-validation.md` instead of
  restating it
- `commands/validate.md` reports the per-tool lifecycle states separately
  from check statuses and reports proposals, blocks, and formatter or linter
  errors
- `AGENTS.md` remains the single source of the default tools per ecosystem
  and was not duplicated by the procedure
- a documentation-only repository such as this one is not given an
  application formatter, linter, `package.json`, or dependencies

## V2.15 Regression — Contracts and Implementation Recipes

Verify that implementation guidance and interface contracts are concrete,
non-duplicative, and proportional:

- `skills/implementation-recipes/SKILL.md` and
  `skills/api-contract-design/SKILL.md` exist at their canonical paths and
  are listed in the Skill inventory of this workflow
- `agents/orchestrator.md`, `workflows/task-to-code.md`, and `README.md`
  reference both Skills consistently, and none restates a competing
  lifecycle order or a duplicate decision policy
- both Skills defer architecture decisions to
  `skills/architecture-advisor/SKILL.md` instead of duplicating them
- `skills/implementation-recipes/SKILL.md` states the non-negotiable
  quality baseline: quality is never proportional to task size, and only
  planning rigor and test depth are proportional
- `AGENTS.md` carries the quality-not-proportional principle, and
  `skills/complexity-advisor/SKILL.md` clarifies that complexity and rigor
  govern planning and testing depth only, never code quality
- `skills/api-contract-design/SKILL.md` requires defining the contract
  before implementation when a boundary, interface, or integration changes,
  reuses the existing contract style, treats breaking changes as approval
  gates, and never exposes secrets in the contract
- no separate architecture-decisions Skill exists; the Decision Records
  guidance lives in `skills/architecture-advisor/SKILL.md` and records only
  significant, hard-to-reverse decisions without fabricated rationale
- `workflows/task-to-code.md` loads the contract Skill in the architecture
  phase when boundaries change and loads the implementation recipes in the
  implement phase
- `references/system-workflow-regression-matrix.md` includes the
  contract-before-implementation and quality-not-proportional scenarios
  (R-33, R-34)
- `integrations/skill-catalog.md` lists the internal authored Skills and
  does not present them as external import candidates
- no new command, package.json, dependency, or competing orchestrator was
  introduced

## V2.16 Regression — Database Engineering

Verify that database work is gated, safe, and non-duplicative:

- `skills/database-engineering/SKILL.md` exists at its canonical path and is
  listed in the Skill inventory of this workflow
- `agents/orchestrator.md`, `workflows/task-to-code.md`, and `README.md`
  reference the Skill consistently and state that it loads only when a task
  touches the schema, migrations, queries, indexes, or transactions
- `skills/database-engineering/SKILL.md` requires backward-compatible
  expand/contract migrations applied through the project's tooling, forbids
  editing applied migrations, and requires parameterized queries and
  N+1 awareness
- it requires explicit human approval for destructive or production data
  operations and never permits bypassing the migration tooling
- `skills/implementation-recipes/SKILL.md` defers schema, migration, query,
  and transaction changes to the database Skill instead of duplicating them
- the Skill does not duplicate the architecture decision policy owned by
  `skills/architecture-advisor/SKILL.md` or the contract policy owned by
  `skills/api-contract-design/SKILL.md`
- `references/system-workflow-regression-matrix.md` includes the database
  safety scenario (R-35)
- no new command, package.json, dependency, or competing orchestrator was
  introduced

## V2.17 Regression — Delivery Pipeline Authoring

Verify that pipeline authoring is an authorized mode, environment-aware, and
gate-enforced, without weakening the read-only assessment:

- `workflows/secure-delivery.md` describes an authoring mode distinct from
  the read-only assessment, and `commands/delivery-check.md` remains
  read-only and states that authoring happens inside a task
- the authoring mode builds one artifact and promotes it across environments
  with environment differences kept out of the artifact and no secret values
  in source
- it requires the same validation gates before every deployment and states
  that gates are never weakened to obtain a green pipeline
- production deployment requires an explicit approval gate, and each
  deployable environment has a stated rollback or recovery path
- a failing pipeline is met with a proposed fix or rollback, never a silent
  bypass, and no remote write, push, merge, or deploy occurs without explicit
  authorization
- `workflows/task-to-code.md`, `agents/orchestrator.md`, and `README.md`
  reference the authoring mode consistently and do not restate a competing
  delivery policy
- `workflows/secure-delivery.md` still does not recommend or introduce a CI
  platform, container base, or deployment dependency without justification
  and authorization
- `references/system-workflow-regression-matrix.md` includes the pipeline
  authoring scenario (R-36)
- no new command, skill, package.json, dependency, or competing orchestrator
  was introduced

## V2.18 Regression — Neutral State Root and Single Orchestrator

Verify the properties that are not covered by a later, more specific
check:

- the learning store uses the provider-neutral state root
  (`${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}`) with no
  client-specific path, and never writes inside a target project's source
  tree or the canonical repository's tracked files
- learnings never store credentials, tokens, secrets, or personal data, and
  identifiers that become path components are normalized against traversal
- promotion to the canonical layer is always a proposal that requires human
  approval; no instruction allows the system to edit the canonical
  repository automatically
- `workflows/session-resilience.md` writes checkpoints to the same
  provider-neutral state root and defines the handoff path in exactly one
  place; `AGENTS.md` and the commands reference it without duplicating the
  literal path
- no new command, skill, package.json, dependency, or competing orchestrator
  was introduced

The learning execution, the maturity ladder, and the context-efficiency
policy are verified in V2.38 and V2.30.

## V2.19 Regression — Standalone Estimation

Verify that estimation is standalone, honest, and non-invasive:

- `workflows/estimation.md` and `commands/estimate.md` exist at their
  canonical paths and are listed in the required-file inventory of this
  workflow
- estimation is described as a standalone activity that does not start the
  task lifecycle and does not modify application source, configuration,
  branches, or remote state
- estimation is scoped to a complete project from start to finish and states
  that individual tickets, stories, and tasks are out of scope; `AGENTS.md`,
  `agents/orchestrator.md`, `commands/estimate.md`, and `README.md` reflect
  that project scope consistently
- `commands/estimate.md` is a thin adapter that delegates to
  `workflows/estimation.md`; the accepted artifacts, the procedure, and the
  report are defined only in the workflow and are not restated in the command
- the document is written to the current working directory as
  `<slug>-estimation.<ext>` with a relative, cross-platform-safe path, and the
  command reports the saved location (relative and absolute)
- before writing, the workflow asks the user for the document language and
  format, offering a short list of suggested options, and uses exactly what
  the user states; the language is not inferred from the request; a choice
  the current client or model cannot produce is flagged and a supported
  alternative is offered instead of being produced silently (Markdown
  recommended, HTML dependency-free, PDF/DOCX only with an existing converter
  and no dependency added); the chosen language and format are reported
- the slug normalization rejects path separators, `.`/`..`, characters
  illegal on Windows, reserved device names, and case-collision risk, and
  never uses an absolute or machine-specific path
- effort is reported as a low/likely/high range per discipline with stated
  assumptions and a confidence level; monetary cost appears only when a rate
  is supplied, and is otherwise reported as not determined
- the workflow states that it never fabricates effort, duration, cost, or
  rates and never presents an unread artifact as a low estimate
- Phase 1 accepts an image supplied as a local path or as an attachment to the
  session, interpreted with `skills/design-analysis/SKILL.md`; when the client
  or model cannot read the image, it is reported as not estimated, never
  fabricated
- `AGENTS.md`, `agents/orchestrator.md`, `workflows/task-to-code.md`, and
  `README.md` reference the estimation workflow consistently and keep
  estimation-only requests free of branch preparation
- `references/system-workflow-regression-matrix.md` includes the estimation
  scenario (R-39)
- no package.json, dependency, or competing orchestrator was introduced

## V2.20 Regression — Internationalization and Mobile UI

Verify that localization is decided early and implemented correctly, and that
mobile interfaces are analyzed and built for mobile:

- `skills/internationalization/SKILL.md` exists at its canonical path and is
  listed in the required-file inventory of this workflow
- the Skill is scoped to user-facing work and is explicitly not loaded for
  work without user-facing text
- localization is settled early (required locales, RTL, and locale-aware
  formatting) in `workflows/task-to-code.md` (Phase 5), `agents/orchestrator.md`
  (§11), and `commands/task.md`, without duplicating a competing sequence
- the Skill requires user-facing strings to be externalized, forbids
  concatenating translated fragments, requires pluralization and locale-aware
  formatting, and covers direction/RTL, translation workflow, testing, and
  accessibility/SEO (`lang`, `dir`, `hreflang`)
- a new i18n library is subject to `workflows/dependency-assessment.md` and
  human authorization
- `skills/design-analysis/SKILL.md` provides a Mobile Analysis section and
  `skills/implementation-recipes/SKILL.md` provides concrete mobile
  implementation guidance, without introducing a separate mobile Skill
- `agents/orchestrator.md`, `integrations/skill-catalog.md`, `README.md`, and
  `references/official-documentation.md` reference internationalization and
  mobile consistently
- `references/system-workflow-regression-matrix.md` includes the
  internationalization and mobile scenarios (R-40, R-41)
- no new command, package.json, dependency, or competing orchestrator was
  introduced

## V2.21 Regression — Refactoring and Abstraction Justification

Verify that unsolicited refactoring and unnecessary fragmentation are prohibited:

- `skills/complexity-advisor/SKILL.md` defines the Refactoring and Abstraction
  Justification gate with thresholds (reuse, project size/complexity, required
  change, testability) and negative triggers (single use, no threshold,
  aesthetic, barrel without utility)
- `skills/architecture-advisor/SKILL.md` defines the Fragmentation Gate
  requiring the same justification for component/module extraction, barrels,
  and reorganization
- `AGENTS.md` scopes localized refactoring to what the task requires; unsolicited
  restructuring is out of scope and becomes an Engineering Insight
- `workflows/task-to-code.md` Phase 9 references the justification gate for
  proposed new components; Phase 10 enforces it and requires resolution of
  over-engineering findings (revert or justify with evidence), not waiver
- `skills/engineering-review/SKILL.md` classifies unnecessary abstraction/
  fragmentation as a finding that must be resolved, not a conditional PASS
- `references/system-workflow-regression-matrix.md` includes the scenario
  (R-42)
- no new command, package.json, dependency, or competing orchestrator was
  introduced

## V2.22 Regression — Code Audit

Verify that the code-creation audit is a standalone, read-only, non-duplicative workflow:

- `commands/code-audit.md` exists and delegates to `workflows/code-audit.md`
  without restating the procedure; it specifies the agent (`reviewer`) and
  read-only behavior
- `workflows/code-audit.md` defines the 14 applicability-gated dimensions,
  the senior/staff bar, the dimension matrix (PASS/WARN/FAIL/N/A), findings
  format (P0–P3, evidence, severity, confidence), and the verdict
- the workflow loads only applicable Skills and does not duplicate their
  checklists; it references `architecture-advisor`, `complexity-advisor`,
  `implementation-recipes`, `engineering-review`, `api-contract-design`,
  `database-engineering`, `best-practices`, `testing-strategy`,
  `test-driven-development`, `performance`, `core-web-vitals`,
  `accessibility`, `seo`, `internationalization`, `frontend-design`,
  `design-analysis`, `repository-strategy`
- it audits code only (no process compliance: branch policy, dependency
  gate, evidence honesty, git hygiene are out of scope)
- it is read-only: no edits, commits, pushes, or remote writes
- `agents/reviewer.md` and `agents/orchestrator.md` reference the code-audit
  workflow and intent
- `README.md` distinguishes `/code-audit` from `/system-check`, `/validate`,
  and `/review-pr`
- `references/system-workflow-regression-matrix.md` includes the scenario
  (R-43)
- no new Skill, agent, package.json, dependency, or competing orchestrator
  was introduced
- **exactly two modes**: the last commit (`HEAD`) by default and the whole
  project with `--all`; no `--commit`, `--range`, paths, or fallback between
  modes
- the report is **advisory**: improvement suggestions, never applied changes
  and never a commit gate
- `workflows/code-audit.md` Phase 2 resolves the commit scope with
  `git diff`/`git show`, handles a root commit through `--root`, and never
  checks out a commit or modifies the working tree; declaration sites outside
  the scope are read-only context

```sh
failures=0
grep -q '\-\-all' commands/code-audit.md || { echo "FAIL V2.22: no --all"; failures=1; }
grep -q 'last commit' commands/code-audit.md || { echo "FAIL V2.22: no default commit mode"; failures=1; }
grep -q 'last commit' workflows/code-audit.md || { echo "FAIL V2.22: workflow has no default commit mode"; failures=1; }
grep -q 'root commit' workflows/code-audit.md || { echo "FAIL V2.22: root commit not handled"; failures=1; }
if grep -qE '\-\-commit|\-\-range' commands/code-audit.md workflows/code-audit.md; then
  echo "FAIL V2.22: a third scope mode was added"
  failures=1
fi
exit "$failures"
```

## V2.23 Regression — Folder Architecture, Dependency Direction, and Type/Contract Ownership

Verify that folder architecture and type/contract ownership are defined once, enforced in generation/review/audit, and reconciled with R-42:

- `skills/architecture-advisor/SKILL.md` defines the **Folder Architecture Map** (layers/modules, responsibility→folder, allowed dependency directions, contract/type location per module, co-location rules) and the **ownership principle** (type lives in the folder of the owning layer); includes the per-ecosystem matrix (TS/JS, Java/Kotlin, Python, C#/.NET, Go, Rust, PHP, Ruby, SQL) with typical folders, contract location, and import direction; explicitly states "not mandatory; preserve detected conventions."
- `skills/complexity-advisor/SKILL.md` reconciles R-42: shared contract extraction meets reuse threshold; single-consumer props/local types do not; language idiom overrides file-count aesthetics.
- `skills/implementation-recipes/SKILL.md` includes the **Folder Architecture and Contract Placement** section with the cross-ecosystem table and per-stack examples (React/Next.js feature `types/`, Java `domain`, Python `domain/contracts.py`, Go consumer-side interfaces).
- `skills/engineering-review/SKILL.md` checks folder conformance, dependency direction, and contract ownership in Architecture.
- `workflows/code-audit.md` Dimension 1 renamed to "Architecture & folder structure"; adds explicit **Folder Architecture check** (build map, map declarations, verify imports, flag layer-skipping/mixed-responsibility/global buckets) and **Type/Contract Ownership check** (language-aware declaration patterns, consumer mapping, follow declaration sites, flag shared-in-impl/import-from-impl/duplicates/cycles/idiom violations; exempt single-consumer props, Go consumer interfaces, private types, RBS); severity P1/P2/P3 with tags `TYPE_OWNERSHIP`/`FOLDER_ARCHITECTURE`; pre-existing labeling for declarations outside scope; no universal template imposed.
- `commands/code-audit.md` notes scope expansion for architecture/type ownership.
- `references/system-workflow-regression-matrix.md` includes R-44 with documented cases:
  - TS: shared type in `lib/payment-service.ts` imported by UI/hook → P2; `MethodOptionProps`/`Product` inline → OK.
  - Java: contract in `service/impl` imported by controller → defect; DTO in `domain` → OK.
  - Python: `TypedDict` in `services/stripe.py` imported by router → defect; `Protocol` in `domain/contracts.py` → OK.
  - Go: interface in consumer package → OK; forced in provider → defect anti-idiomatic.
  - Folder: UI → concrete repo; cross-feature private import; global `types.ts` dump.
- No new Skill, agent, command, package.json, dependency, or competing orchestrator introduced.
- No blanket "all inline types are defects" rule; no universal folder template imposed.
- **default scope is the uncommitted change set; checks operate only on modified files; declaration sites outside the diff are context only; pre-existing issues outside the diff are not reported; clean tree = NOTHING TO AUDIT**

## V2.24 Regression — Commit Generation with Project-Native Gates

Verify that commit generation is a standalone workflow with project-native
quality gates and atomic commits:

- `commands/commit.md` exists and delegates to `workflows/commit.md`
  without restating the procedure; it specifies the agent (`orchestrator`)
  and allows `--dry-run`, `--staged`, `--full`, and path arguments.
- `workflows/commit.md` defines the gate sequence: secrets scan → format
  check → lint → typecheck → unit tests → integration/e2e (`--full`) →
  build → security audit → AI sanity review.
- The workflow detects actual tooling from the project (package manager,
  scripts, lockfiles); it does not invent commands or install tools.
- Missing mandatory tooling (formatter, linter for software projects)
  reports `SETUP_REQUIRED` and blocks commits; no silent omission.
- Secrets scan blocks on common patterns; no external dependency required.
- AI sanity review is lightweight: obvious correctness/security defects
  only; no architecture, patterns, or over-engineering evaluation.
- Commits are atomic, English, Conventional Commits unless project
  convention differs; no AI mentions; no unverified test claims.
- Same gates for AI and human changes; no authorship-based bypass.
- Never pushes, deploys, or bypasses gates (`--no-verify` not supported).
- Clean tree = `NOTHING TO COMMIT`.
- Documentation-only repositories skip application gates.
- `skills/git-workflow/SKILL.md` references the workflow without
  duplicating rules.
- `agents/orchestrator.md` references the workflow in the Delivery phase.
- `README.md` distinguishes `/commit` from `/validate`, `/code-audit`,
  and `/generate-pr`.
- `references/system-workflow-regression-matrix.md` includes R-45.
- No new Skill, agent, package.json, dependency, or competing orchestrator
  introduced.

- No new Skill, agent, package.json, dependency, or competing orchestrator
  introduced.

## V2.26 Regression — Folder Architecture by Complexity, Dumping Ground Prohibition, and Reuse-Based Type Ownership

Verify that folder architecture is decided by complexity, `lib/`/`utils/` dumping grounds are prohibited, and type extraction is reuse-based:

- `skills/architecture-advisor/SKILL.md` replaces "detected and preserved" with **detect first, decide by complexity if absent**: small→layer-first (`src/api/<domain>`, `src/services/<domain>`, `src/server/<domain>`, `src/shared/`, `src/components/`, `src/hooks/`); medium→feature-first (`src/features/<feature>/{api,domain,server,components,hooks,contracts}` + `src/shared/`); large→feature modules with explicit public APIs and boundaries. **Universal responsibility rules**: `lib/`/`utils/` are generic utilities only; API has its own folder (`contracts.ts` + `client.ts`); services in `services/` or `domain/`; storage in `server/`/`infrastructure/`; i18n in `shared/` or top-level. A mixed `lib/` is an anti-pattern, not a convention to preserve. New code uses correct folders; existing moves only when touched (R-42).
- `skills/implementation-recipes/SKILL.md` adds the **complexity-based structure decision table** and an **API module recipe** (`contracts.ts` for schemas + inferred types, `client.ts` for validators/converters/fetch, `service.ts` for domain logic, `storage.ts` for persistence); i18n example moved to `src/shared/i18n/`.
- `skills/project-analysis/SKILL.md` adds **Dumping Ground Detection**: classifies `lib/` files; flags domain-specific code in `lib/` as "legacy; do not replicate".
- `skills/engineering-review/SKILL.md` adds dumping ground check, reuse-based type extraction (only when imported from another file), schema + inferred type pair stays in `contracts.ts`/`schemas.ts`, API file cohesion (contracts separate from client).
- `workflows/code-audit.md` Dimension 1: **Folder Architecture check** adds dumping ground detection; **Type/Contract Ownership check** is reuse-based (P2 when imported from another file, WARN when only exported); **File Cohesion check** adds API-specific pattern (schemas + validators + client in one file → P2).
- `references/system-workflow-regression-matrix.md` includes R-47 with documented cases:
  - Counterexample: a `lib/` containing i18n, payments service, storage, API contracts, and API client mixed → P2 `FOLDER_ARCHITECTURE` + `TYPE_OWNERSHIP`.
  - Correct for a small app: layer-first `src/api/payments/{contracts,client}`, `src/services/payments`, `src/server/payments/storage`, `src/shared/i18n/`.
- No new Skill, agent, package.json, dependency, or competing orchestrator introduced.
- `/commit` does **not** block on architecture/type placement; behaves like Husky (objective gates only). Architecture is in code-audit/review/task-to-code.

## V2.27 Regression — Commit Approval Gate

Verify that no commit is created without explicit human approval that follows
a proposal the user was able to review, including when the prompt itself asks
for the commit:

- `AGENTS.md` includes "Creating a commit" in Human Decision Gates and a **Commit Approval Policy** with a fixed sequence — inspect, run the gates, present the proposal with the change set, ask a direct question, create only after an affirmative answer — and states that: an instruction to commit in the prompt, task, ticket, command arguments, earlier turn, or automation is not commit approval; silence, timeout, resumed checkpoint, and automated retry are not approval; a proposal without the change set is not approvable; approval covers exactly the presented groups, files, and messages.
- `workflows/commit.md`: the purpose states that the request that started the work is not commit approval; Phase 13 is a **Proposal and Approval Gate** that presents the diff of each group, rejects a commit instruction in the request as approval, requires the answer to follow the proposal, and lists the not-approval cases; Phase 14 creates only commits approved by that answer; Phase 15 reports approval status; a dry run stops at the proposal.
- `commands/commit.md`: states that the command prepares and validates a proposal and does not create commits by itself, includes the approval gate, and rejects `$ARGUMENTS` or the triggering prompt as approval.
- `skills/git-workflow/SKILL.md`: "Before commit" shows the change set, asks for explicit approval, and creates only after the affirmative answer.
- `agents/orchestrator.md` and `workflows/task-to-code.md`: delivery requires approval after the user reviewed the change set and rejects command invocation, an implementation request, or a commit instruction in the original task as approval.
- `commands/task.md`: the task command presents the diff and the proposal and never treats the task request as commit approval.
- `references/system-workflow-regression-matrix.md` includes R-48.
- No commit is created during validation; the validation itself is read-only.

```sh
for f in AGENTS.md workflows/commit.md commands/commit.md \
         skills/git-workflow/SKILL.md workflows/task-to-code.md \
         commands/task.md agents/orchestrator.md; do
  if ! tr '\n' ' ' < "$f" | tr -s ' ' ' ' | grep -Fqi 'not commit approval'; then
    printf 'FAIL M-05: %s does not reject the commit instruction as approval\n' "$f"
  fi
done
```

## V2.28 Regression — Primary Unit per File, Unit Folders, and Co-located Tests

Verify that file/unit organization is applied according to ecosystem support, not as a universal rule:

- `skills/architecture-advisor/SKILL.md`: **Primary Unit per File and Stack Applicability** defines the rule, the dumping-ground anti-pattern, the Angular exception, and optional barrels; **Unit Folders and Co-located Tests** defines the sibling anti-pattern, ecosystem-appropriate layouts, R-42 scope, and the official documentation reference.
- `skills/implementation-recipes/SKILL.md`: the canonical applicability matrix covers React/Next.js, TypeScript/JavaScript backend (Node.js, Express, NestJS, Fastify), Angular, Vue, Svelte, Solid, React Native, Ionic, Java, Kotlin, Python, Go, Rust, C#/.NET, PHP, and Ruby, with the primary-unit rule and test location per ecosystem; the per-language recipe sections cover each backend and mobile stack. Two rules apply in every ecosystem: a test next to its unit or in its mirrored tree, never at the source root or a layer root; and a support location (`testing/`, `test/support/`, `spec/support/`, `conftest.py`, `src/testFixtures`) that never contains test cases. Barrels are optional and not part of the default layout.
- `skills/architecture-advisor/SKILL.md`: **Unit Folders and Co-located Tests** states that a test case never sits at the source root or a layer root, that an unowned test belongs to `tests/integration/` or next to the module it exercises, and that a support location never contains test cases.
- `workflows/output-compliance.md`: `FILE-001` detects a test at the source root, a test case inside a support location, a `src/test` folder that holds only support files, and a test inside `src/` with no sibling unit and no `.integration.test.` mark, with `file:line` evidence, and its per-ecosystem table covers Python, Go, Rust, Java, Kotlin, C#/.NET, PHP, and Ruby mirrored trees.
- `AGENTS.md` states the test placement and support-location rule, and lists it in the structural compliance check.
- `skills/project-analysis/SKILL.md`: Folder Architecture Map and Internal Module Layout Detection report primary-unit violations, unit grouping, test co-location, and barrel justification where supported.
- `skills/engineering-review/SKILL.md`: review flags multi-unit files and `X`, `X.test`, and `X/` siblings only where the ecosystem supports the rule; it does not impose React/Angular conventions on Solid, Kotlin, Python, Go, Rust, C#/.NET, or Ruby.
- `workflows/code-audit.md`: Dimension 1 includes the **Primary unit per file check**, **Unit folder and test co-location check**, and **Barrel check**, with `FILE_COHESION`/`FOLDER_ARCHITECTURE` tagging and P2/P3 severity.
- `workflows/task-to-code.md` Phase 8: the proposal must include a File Responsibility Map, respect the applicability matrix, and use a unit folder with a co-located test where the ecosystem supports it.
- `references/official-documentation.md`: **Project Structure and Test Co-location** records what each source supports: React/Next.js do not mandate one component per file; Airbnb/ESLint do; Angular supports “one concept per file” with an exception; Vue/Svelte are inherently one component per file; Solid and Kotlin allow multiple related declarations; Java public API types are conventionally one per file, and PHP autoloaded classes are one per file; the remaining ecosystems have no one-unit-per-file mandate.
- `references/system-workflow-regression-matrix.md` includes R-46 and R-49.
- No demo-specific assertion is part of this validation; the demo and other reference paths are read-only.

## V2.29 Regression — Target Project and Evidence-Only Paths

Verify that scope is explicit and reference paths are never modified:

- `AGENTS.md`: **Scope and Evidence Boundaries** states that the target project is the only writable scope, evidence-only paths are read-only without explicit authorization, reference examples are not implicit tasks, and out-of-target changes require a stop-and-ask decision.
- `workflows/task-to-code.md`: Phase 1 records the target project root and evidence-only paths; Phase 2 preserves the boundary; Phase 3 treats design artifacts as evidence; Phase 10 forbids modifying evidence-only paths, reference projects, design artifacts, or external repositories.
- `agents/orchestrator.md`: **Scope and Evidence Boundaries** requires identifying the target and evidence-only paths before delegating and passing the boundary to every delegated agent.
- `skills/discovery/SKILL.md`: discovery identifies the target project and evidence-only paths and reports them in its output.
- `references/system-workflow-regression-matrix.md` includes R-50.
- A path supplied only as evidence, comparison, or context is never modified without explicit authorization, even when it demonstrates a real defect.

## V2.30 Regression — Minimum Sufficient Context and Token Efficiency

Verify that context efficiency is bounded, measured, and subordinate to
correctness:

- `AGENTS.md`: **Minimum Sufficient Context** requires a Context Manifest,
  soft exploration budgets, stop rules, measurement, and no correctness
  tradeoff.
- `workflows/session-resilience.md`: **Token and Context Efficiency** defines
  the Context Manifest, soft exploration budgets for R1–R4, retrieval rules,
  stop rules, and measurement; budgets exclude implementation and validation.
- `workflows/task-to-code.md`: Phase 1 records the manifest and provisional
  rigor; Phase 2 refines it without unrelated paths; Phase 9 includes the
  final budget; Phase 10 enforces the stop rules and records expansions.
- `skills/task-context/SKILL.md`: the working interpretation contains the
  Context Manifest and the rule not to broaden it without evidence.
- `references/token-efficiency.md`: defines metrics, baseline and
  calibration, optional read-only queries, and the no-hard-cap principle.
- `references/system-workflow-regression-matrix.md` includes R-51.
- The policy applies to exploration only; implementation, tests, formatting,
  build, validation, and security checks remain mandatory.
- No hard token cap, compaction plugin, or reduction of required evidence is
  introduced.

## V2.31 Regression — Structural Compliance Verification

Verify that the structural rules are enforced by this system and are not
installed in the target project:

- `workflows/output-compliance.md` exists at its canonical path and is listed
  in the required-file inventory of this workflow.
- It defines deterministic checks with executable detection patterns and
  `file:line` evidence: `FUNC-001`, `TYPE-001`, `TYPE-002`, `TYPE-003`,
  `STRUCT-001` to `STRUCT-007`, and `CONFIG-001`.
- It is read-only and never modifies code.
- `AGENTS.md` **Implementation and Validation** makes the check mandatory
  before a task is reported as implemented and states that a `FAIL` is a
  defect of the task, not a finding to defer.
- `workflows/task-to-code.md` Phase 10 runs it and forbids reporting
  completion with an unresolved `FAIL`. Phase 4 keeps the project's lint
  configuration standard and never adds a custom rule file.
- `workflows/project-validation.md` includes it as a mandatory check, and
  `commands/validate.md` runs it.
- `workflows/code-audit.md` requires the executed detection as evidence for
  `TS_FUNCTION_STYLE` and `TYPE_OWNERSHIP` findings.
- `references/linter-rules.md` exists at its canonical path and is listed in
  the required-file inventory of this workflow. It defines the standard
  ESLint flat configuration, the Prettier configuration, and the coverage
  matrix, and states that **no custom rule file**, plugin, or checker script
  is created in a target project.
- `AGENTS.md` **Formatting and Linting Policy** requires only the standard
  configuration, expressed through the project's own `eslint.config.*` and
  Prettier files, and states the rule in the same terms. A linter without a
  custom artifact is a complete setup.
- `workflows/project-validation.md` Phase 5 keeps the configuration standard
  and Phase 7 verifies that the configuration exists and that the project
  contains no bespoke enforcement artifact.
- `CONFIG-001` is exercised both ways: a project that adds `eslint-rules/` or
  `scripts/check-contract-consumers.mjs` is `FAIL`, and the same project
  without those files is `PASS`.
- `references/system-workflow-regression-matrix.md` includes R-52.
- The checks cover TypeScript/JavaScript, Python, Java/Kotlin, Go, Rust,
  C#/.NET, PHP, and Ruby.

```sh
failures=0
for f in AGENTS.md skills/implementation-recipes/SKILL.md \
         workflows/project-validation.md workflows/task-to-code.md \
         workflows/output-compliance.md references/linter-rules.md; do
  if ! tr '\n' ' ' < "$f" | tr -s ' ' ' ' | grep -Fqi 'no custom rule file'; then
    printf 'FAIL V2.31: %s does not state that no custom rule file is installed\n' "$f"
    failures=1
  fi
done
exit "$failures"
```

## V2.32 Regression — Mandatory Rule Reachability

Verify that every mandatory rule is reachable from every surface an agent
may read, and that the canonical wording lives in exactly one place.

`ai-engineering` is the single source of truth. A rule that exists only in
an optional Skill is invisible to an agent that never loads it. A rule
restated in many files drifts apart. The registry in
`references/mandatory-rules.md` records the canonical wording, the anchor,
and the required surfaces of every mandatory rule.

### Reachability check

For each row of the registry, verify the anchor is present on every required
surface. The comparison is normalized — lowercase, `-` and space treated as
equivalent, and whitespace collapsed — so that a line wrap or a hyphen does
not produce a false failure.

The check uses only `grep`, `sed`, and `tr`, so it runs on any client and in
any environment without an interpreter dependency:

```sh
normalize() {
  tr '\n' ' ' | tr 'A-Z' 'a-z' | tr '-' ' ' | tr -s ' ' ' '
}

trim() {
  sed 's/`//g; s/^[[:space:]]*//; s/[[:space:]]*$//'
}

failures="$(
  grep -E '^\| M-[0-9]+ \|' references/mandatory-rules.md |
  while IFS='|' read -r _ id _ _ anchor surfaces; do
    id="$(printf '%s' "$id" | trim)"
    anchor="$(printf '%s' "$anchor" | trim)"
    normalized_anchor="$(printf '%s' "$anchor" | normalize)"
    printf '%s\n' "${surfaces%%|*}" | tr ',' '\n' | while read -r surface; do
      surface="$(printf '%s' "$surface" | trim)"
      [ -n "$surface" ] || continue
      if [ ! -f "$surface" ]; then
        printf 'FAIL %s: missing surface %s\n' "$id" "$surface"
        continue
      fi
      normalized_file="$(normalize < "$surface")"
      if ! printf '%s' "$normalized_file" | grep -Fq -- "$normalized_anchor"; then
        printf 'FAIL %s: anchor "%s" missing from %s\n' "$id" "$anchor" "$surface"
      fi
    done
  done
)"

if [ -n "$failures" ]; then
  printf '%s\n' "$failures"
  exit 1
fi
```

A rule whose canonical wording is missing from `AGENTS.md` is a `FAIL`: it
would not reach every client.

### Single-source check

Verify the canonical wording is not copied into other files. The anchor of a
rule whose canonical wording is in `AGENTS.md` must appear in `AGENTS.md`,
and the surfaces listed in the registry must reference the rule rather than
restate its normative wording:

```sh
# The canonical wording of an always-active rule lives in AGENTS.md.
# Count the files that carry the anchor: more than the canonical file plus
# the registry means a copy that can drift.
grep -rlF "arrow functions" --include="*.md" . | grep -v node_modules
```

Review the result. A surface that restates the normative wording instead of
referencing `AGENTS.md` is a drift risk: convert it into a reference.

### Trigger check

Verify that every Skill description names the concrete triggers that make an
agent load it. A description that only states an abstract purpose does not
match a concrete task and the Skill is never loaded:

```sh
for skill in skills/*/SKILL.md; do
  printf '%s -> ' "$skill"
  sed -n '3p' "$skill" | cut -c14-
done
```

A description that omits the nouns of the task it supports is a `FAIL`.

### Consistency checks

- `references/mandatory-rules.md` exists at its canonical path and is listed
  in the required-file inventory of this workflow.
- Every registry row has an ID, a canonical wording, an anchor, and at least
  one required surface.
- Every required surface exists in the repository.
- The canonical wording of every always-active rule is in `AGENTS.md`.
- `references/system-workflow-regression-matrix.md` includes R-53.

## V2.33 Regression — New Project Initialization

Verify that a new project follows one path and asks for the architectural
style before any structure is created:

- `workflows/project-initialization.md` exists at its canonical path and is
  listed in the required-file inventory of this workflow.
- It recovers the source analysis the task points to and reuses it instead
  of re-deriving requirements; a requirement the analysis does not cover is
  treated as unknown, not implied.
- It confirms the detected stack against `references/official-documentation.md`.
- Phase 3 presents the applicable architectural styles with their trade-offs
  and asks the user which one to use. A recommendation is not a decision, and
  the repository structure is not created before the answer.
- Phase 5 installs the standard formatter and linter configuration from
  `references/linter-rules.md`, with reproducible commands, plus a test runner
  and a first test. No custom rule file, plugin, or checker script is created
  in the project.
- `AGENTS.md` **Human Decision Gates** lists the architectural style of a new
  project as a decision that requires the user.
- `workflows/task-to-code.md` Phase 4 routes a new project to the
  initialization workflow instead of creating it inline, and its Phase 8 gate
  covers the new-project case.
- `skills/architecture-advisor/SKILL.md` covers the new-project case in its
  decision gate.
- `agents/orchestrator.md` and `commands/task.md` route a new project to the
  initialization workflow.
- `workflows/estimation.md` defines how the estimation document is reused as
  the input of implementation.
- `references/project-startup-decisions.md` exists at its canonical path and
  is listed in the required-file inventory of this workflow. It covers the
  universal decisions and the decisions of a backend, a frontend, a library
  or CLI, and a mobile application, each with a recommended default, the
  condition that requires the user, and where the answer comes from.
- Each decision is resolved in order: answered by the source analysis,
  governed by a detected convention or policy, has a defensible default, or
  materially changes the outcome with no default. The user is asked once,
  grouped by theme, only for the last category, with its recommendation and
  trade-off. Nothing that the source analysis already answers is re-asked,
  and no decision is left implicit in a default that was never stated.
- `references/project-creation-order.md` exists at its canonical path and is
  listed in the required-file inventory of this workflow. It defines the
  universal creation steps, the backend creation steps, the frontend,
  library, and mobile creation steps, and the complete creation checklist.
- `workflows/project-initialization.md` Phase 4 creates the project step by
  step, verifies each step before the next one starts, applies the growth
  rule, and copies the creation checklist into the report.
- Each creation step states what it produces, how to verify it, and where
  the project grows next, so the structure has an extension point instead of
  a parallel structure.
- `references/system-workflow-regression-matrix.md` includes R-54.

## V2.34 Regression — End-to-End Audit of an Adjustment

Verify that an adjustment can be applied and proved correct in one command,
driven only by its text:

- `commands/audit-e2e.md` exists at its canonical path and is listed in the
  required-file inventory of this workflow. It delegates to
  `workflows/system-audit.md` and copies no block from it.
- Every command in `commands/` has its own description in the README, and
  the structural audit verifies it. A command without a README description
  is a `FAIL`: the catalog a reader consults is incomplete.
- `workflows/system-audit.md` exists at its canonical path and is listed in
  the required-file inventory of this workflow.
- The command accepts the adjustment text as its only input. It defines no
  mode, no flag, and no implicit scope, and it never derives the scope from
  the Git change set.
- The adjustment text determines the area, the rules in scope, the required
  checks, and the artifacts to build, through the area mapping in the
  workflow.
- The command implements the adjustment following `workflows/task-to-code.md`,
  then audits it in the same run.
- The structural audit always runs: reference integrity, index and
  inventory, numbering, rule reachability, cross-client reach in
  `AGENTS.md`, duplication and contradiction, and coverage.
- The behavioral audit builds a real artifact per affected ecosystem or
  project type in a temporary directory outside the repository, installs the
  canonical artifacts, verifies the compliant artifact passes, injects the
  violation the adjustment was meant to prevent, verifies it is detected
  with `file:line` evidence, fixes it, re-verifies, and cleans the
  directory.
- A check that could not be executed is reported as `NOT VERIFIED` with the
  reason, never as passed.
- A defect found in the system's own artifact is reported as a defect with
  the evidence, and the cycle is repeated after the correction.
- The command presents the commit text and does not create the commit.
- `references/system-workflow-regression-matrix.md` includes R-55.

## V2.35 Regression — Progressive Project Creation

Verify that a new project is created progressively, within an agreed scope,
with user-controlled stopping points:

- `commands/create-project.md` exists at its canonical path and is listed in
  the required-file inventory of this workflow.
- `AGENTS.md` states that a new project is created progressively, that the
  scope catalog is agreed first and is a ceiling, that each step ends in a
  checkpoint that reports the next step and waits, and that stopping is a
  normal outcome.
- `AGENTS.md` states that creating a project from zero requires the
  explicit `create-project` command, and that without it the agent asks
  instead of creating one.
- `references/project-creation-order.md` defines the scope catalog, the
  dependency budget, the one-slice-at-a-time rule, the checkpoint
  semantics, and the progress tracker with its states.
- `workflows/project-initialization.md` Phase 3 agrees the scope catalog,
  and Phase 4 executes one step at a time, verifies it, updates the
  tracker, reports the next suggested step, and waits.
- The remaining steps are never generated to finish the project, and the
  scope is never exceeded without asking.
- `references/system-workflow-regression-matrix.md` includes R-56.

## V2.36 Regression — Target Version and Official Documentation

Verify that the target version is settled before the structure is created,
and that the official documentation mandate is satisfiable:

- `AGENTS.md` states that a new project settles its target version before its
  structure is created, that the source of truth is the source analysis,
  then the official documentation, then a stated default, that a version is
  never chosen because it is installed, and that the official documentation
  is consulted for that version with the consultation recorded.
- `workflows/project-initialization.md` Phase 2 confirms the target version,
  consults the official documentation for it, compares it with the
  environment, and reports a mismatch instead of silently resolving it.
- `references/project-startup-decisions.md` lists the target version as a
  universal decision, and states the version and documentation rules.
- `references/project-creation-order.md` states that the project is created
  for the confirmed target version, and every skeleton step of every project
  type produces an artifact pinned to it and verifies the reported version.
- `skills/stack-detection/SKILL.md` separates the version found from the
  target version, and records the source of truth for it.
- `references/official-documentation.md` contains a section for every
  supported ecosystem, so the consultation mandate is satisfiable for all of
  them.
- `workflows/system-audit.md` requires the behavioral artifact to be built
  for the confirmed target version, and to report as `NOT VERIFIED` any
  check that could not run against it.
- `references/system-workflow-regression-matrix.md` includes R-57.

## V2.37 Regression — Documentation Freshness

Verify that consulted documentation is current and that a stale source is
visible:

- `AGENTS.md` states that the version guide is consulted first, that a
  version-agnostic source is preferred, that a pinned source is registered
  with its version and a review rule, and that the consulted version and the
  consultation date are recorded.
- `references/official-documentation.md` contains a **Freshness** policy that
  classifies each source as version-agnostic or version-pinned, and a
  **Version-Pinned Sources** table with the version and the review rule of
  every pinned source.
- Every URL in the official documentation index that contains a version
  number is registered in the pinned table. An unregistered pinned source is
  a `FAIL`, because it can age without anyone noticing.
- `workflows/system-audit.md` runs that check, and resolves every external
  URL.
- `skills/stack-detection/SKILL.md` consults the version guide before the
  ecosystem pages, prefers a version-agnostic source, and records the
  version and the date.
- `workflows/project-initialization.md` Phase 2 reports a source that does
  not cover the target version instead of applying it silently.
- `references/system-workflow-regression-matrix.md` includes R-58.

## V2.38 Regression — Self-Learning Execution

Verify that the system learns from its mistakes and does not repeat them:

- `workflows/self-learning.md` Phase 1 is an **Apply** phase: it reads the
  store for the project scope and the global scope before acting, applies
  every relevant entry, and reports the applied ones.
- Repeating a lesson the store already records is defined as a defect of
  the task, not bad luck, and the task is not complete until it is
  corrected.
- An entry the evidence contradicts is not obeyed: the evidence wins and the
  entry is reported as superseded.
- An empty or missing store is reported and never blocks a task.
- `AGENTS.md` states the consume rule, and `workflows/task-to-code.md`
  Phase 1 reads the learnings and Phase 12 captures a lesson when there is
  a reusable signal.
- `commands/task.md` requires reading the learnings before acting.
- The maturity ladder is Junior, Semi Senior, Senior, Staff, is derived from
  evidence, is stored with its date, and is reported in the task report.
- The execution floor is Staff for every task; the level changes the
  direction and review the work needs, never the quality it gets.
- The store holds no credentials, tokens, secrets, or personal data, and
  never writes inside a target project or the canonical repository.
- `references/system-workflow-regression-matrix.md` includes R-59 and R-60.

### Store structure check

```sh
root="${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}"
[ -d "$root" ] && echo "store present: $root" || echo "store absent (allowed, never blocks)"
[ -d "$root/learnings" ] && echo "learnings scope present" || echo "no learnings yet (allowed)"
[ -d "$root/maturity" ] && echo "maturity scope present" || echo "no maturity record yet (allowed)"
```

A missing store or scope is reported, never treated as a failure. A learning
entry that contains a credential, a token, or personal data is a `FAIL`.

## V2.39 Regression — Portability

Verify that the system works on every client and operating system:

- `references/portability.md` exists at its canonical path and is listed
  in the required-file inventory of this workflow. It defines the contract:
  the repository is the source of truth, the knowledge is client-neutral, the
  procedures assume a POSIX shell, and no operating-system assumption is
  baked into the rules.
- `AGENTS.md` states the portability rule: a client without Skills still
  receives every mandatory rule, the procedures degrade to `NOT VERIFIED`
  with the reason when a tool is missing, and no operating-system path or
  client directory is hardcoded.
- Every workflow is reachable **without** the `commands/` adapters, so a
  client without slash commands is never blocked.
- No shared procedure hardcodes an operating-system path or a client
  configuration directory, and none depends on a macOS-only utility.
- A check that could not run because a tool is missing is reported as
  `NOT VERIFIED` with the tool named, never as passed.
- The setup instructions cover macOS, Linux, and Windows, and state that
  linking is a convenience rather than a requirement.
- `references/system-workflow-regression-matrix.md` includes R-61.

## V2.40 Regression — Feature-Driven Structure

Verify that a feature-driven project is organized by capability, with one
folder per unit and a public API per feature:

- `AGENTS.md` states the rule: the code is grouped by business capability,
  each unit has its own folder with its implementation, its test, and its
  types, each feature exposes a public API, a feature never imports the
  internals of another feature, and a unit with a second consumer is
  promoted to `shared/` rather than copied.
- `skills/architecture-advisor/SKILL.md` defines **Feature-Driven Folder
  Architecture**: the target tree, the folder per unit, the public API of a
  feature, the golden dependency rule, the promotion rule, and the decision
  that the owner, the folder, and the public surface are recorded before
  the files are created.
- The convenience barrel and the public API of a feature are distinguished,
  so the optionality of one does not weaken the requirement of the other.
- `skills/implementation-recipes/SKILL.md` gives the feature-first structure
  with one folder per unit and a public API per feature.
- `references/project-creation-order.md` requires the folder decision before
  the files are created, and states that a flat feature folder is the
  symptom of skipping it.
- `workflows/output-compliance.md` checks STRUCT-001 (one folder per unit),
  STRUCT-002 (no import that reaches past a feature's public API), and
  STRUCT-003 (every feature exposes a public API).
- `workflows/code-audit.md` includes the feature-driven structure check with
  the golden dependency rule.
- `references/system-workflow-regression-matrix.md` includes R-62.

## V2.41 Regression — Folder Architecture as a Decision

Verify that the folder structure is decided, and that every style has a
concrete tree for the detected ecosystem:

- `references/folder-architectures.md` exists at its canonical path and is
  listed in the required-file inventory of this workflow. It defines the
  concrete tree of every style, the rules that hold in every tree, the
  per-ecosystem translation of the unit of organization, and the concrete
  variants of the most distinct combinations.
- Every style listed in `skills/architecture-advisor/SKILL.md` has a tree in
  that reference, and the reference is where the trees live, so they do not
  diverge.
- `AGENTS.md` states that the folder structure is a decision, asked with the
  architectural style and settled before any file is created, and that a
  style presented only as a one-line hint is not a choice the user can make.
- `references/project-startup-decisions.md` lists the folder architecture as
  a decision of the startup decision set.
- `workflows/project-initialization.md` presents the concrete trees with the
  question, asks about the architecture **and** the folders together, and
  never filters Ports and Adapters out of the options.
- `references/project-creation-order.md` takes the tree from the same
  reference.
- `workflows/output-compliance.md` checks STRUCT-004: the folders that exist
  are the folders that were agreed.
- The per-ecosystem translation covers every ecosystem of the applicability
  matrix: TypeScript and JavaScript, React and Next.js, Angular, Vue, Svelte,
   Solid, Java and Kotlin, Python, Go, Rust, C# and .NET, PHP, and Ruby.
- `references/system-workflow-regression-matrix.md` includes R-63.

## V2.42 Regression — Topology Decision and the Monorepo Counter-Case

Verify that the repository topology is decided with a reason, and that an
unjustified monorepo is declined:

- `AGENTS.md` states that a monorepo is justified by a concrete benefit —
  shared code or types, atomic changes across projects, common tooling, or
  consistent validation — and that one team and a shared release train are
  not that benefit.
- `skills/repository-strategy/SKILL.md` states the counter-case: when the
  applications share no code and no change spans them, the benefit a
  monorepo exists for is absent, a coordinated release is a pipeline
  concern, and the simpler topology is separate repositories.
- `workflows/project-initialization.md` records the topology decision and
  its reason as part of the startup decision set.
- `workflows/output-compliance.md` checks STRUCT-005: a repository with
  several packages and no recorded concrete benefit is a `FAIL`.
- `references/system-workflow-regression-matrix.md` includes R-64.

## V2.43 Regression — Source Grounding

Verify that every structural rule and folder tree names its source:

- `AGENTS.md` requires every structural rule and every folder tree to name
  its source and the kind of source, and forbids presenting a community
  methodology as a framework standard or extrapolating one ecosystem's rule
  to another.
- `references/folder-architectures.md` states the authority of each tree:
  Fowler's Presentation-Domain-Data Layering, Feature-Sliced Design,
  Cockburn's original ports and adapters description, the bounded context for
  DDD, and Spring Modulith for a feature-driven Java module.
- `references/folder-architectures.md` records the source of the rule for
  every ecosystem in the per-ecosystem table.
- `references/official-documentation.md` distinguishes an ecosystem official
  rule from a recognized community methodology, and records the verified
  source for each: the Angular Style Guide for "one concept per file", the
  Vue single-file component specification, Solid for multiple components per
  file, the Go specification for a package per directory, PSR-4 for one
  class per file, Spring Modulith for the provided interface, and the
  Feature-Sliced Design layers and public API references.
- Every URL in that section resolves.
- `references/system-workflow-regression-matrix.md` includes R-65.

## V2.44 Regression — Architecture Matrix

Verify that every architectural style works in every supported ecosystem:

- `references/folder-architectures.md` contains the **Canonical Matrix** with
  the top-level folders, the public API, and the core dependency direction of
  each style, and states that the matrix is machine-checkable and how it is
  verified.
- The matrix covers the six styles — layered, feature-driven, hexagonal, DDD,
  modular monolith, and event-driven — and the ecosystems of the
  per-ecosystem translation table.
- A checker derives the expected layout from the matrix, so the trees and the
  checks cannot drift apart.
- For every combination it verifies: the folders of the style exist, a unit
  owns its implementation and keeps its test where the ecosystem places it,
  the module exposes its public API, and the dependency direction of the style
  holds.
- Where the ecosystem has a toolchain available, the generated project is
  compiled, built, imported, or linted. An ecosystem without a local toolchain
  is verified structurally and reported as not verified for execution, never
  as executed.
- `references/system-workflow-regression-matrix.md` includes R-66.

## V2.45 Regression — Installation Gate

Verify that nothing is installed without the user's approval:

- `AGENTS.md` states that the unit of approval is the **installation
  package**, assembled once per task or phase: a single dependency is the
  minimal adjustment and is installed directly, and a package with several
  dependencies is presented once and installed after the user's evaluation.
- `AGENTS.md` states that the gate is never applied once per library, that a
  package is never split to fit the single-dependency case, and that a
  general approval to implement or continue authorizes no installation.
- `AGENTS.md` requires the **installation list** for a multi-dependency
  package: exact package name, version, runtime or development scope, why it
  is needed, what it brings, and the alternatives including doing nothing.
- `workflows/dependency-assessment.md` Phase 4 presents the installation
  list, has no exception, and Phase 5 installs only the approved packages at
  the approved versions, returning to the gate for anything else.
- `workflows/project-validation.md` Phase 5 proposes the mandatory setup
  through the same gate instead of installing it directly.
- `workflows/project-initialization.md` has a phase that applies the gate to
  any step that installs, and `references/project-creation-order.md` marks
  the tooling step as approved before it runs.
- No workflow or Skill contains an instruction that installs a package
  outside an installation package evaluation.
- `references/system-workflow-regression-matrix.md` includes R-67.

## V2.46 Regression — Phase Checklist

Verify that the user can see what the system is doing, step by step:

- `AGENTS.md` requires every status update raised during a phase to carry a
  checklist of the concrete steps of that phase, with the done, pending, and
  blocked states, and states that a checklist of phase names is a defect.
- `workflows/session-resilience.md` defines the **Phase Checklist**: the
  format, the derivation from the work of the phase, the rule that each step
  names the artifact it produces, the emission in every status update, the
  refresh as steps complete, and the fact that it is session state and needs
  no commit.
- `workflows/task-to-code.md` Phase 10 detects the steps from the plan and
  the File Responsibility Map and carries the checklist in every update.
- `references/project-creation-order.md` uses the same format for the
  creation tracker, so there is one format everywhere.
- The same format is used in the creation tracker, an implementation phase, a
  review, and a validation run.
- `workflows/session-resilience.md` lists the scenarios the derivation must
  cover: a new project, a bug fix, a new endpoint, a new feature, a
  refactor, performance work, a migration, a dependency upgrade, a
  documentation change, an accessibility fix, localization, added tests, a
  CI/CD change, and a cross-cutting configuration change. Each yields a
  different checklist, and a checklist of phase names is rejected.
- `references/system-workflow-regression-matrix.md` includes R-68.

## V2.47 Regression — Phase Plan, Checklist, and Phase Gate

Verify that every task shows its phases and stops for approval at each
boundary:

- `AGENTS.md` states that the phase plan is shown before the work starts in
  every task, with at least one phase and no maximum, that a single-phase task
  shows a one-phase plan, and that a plan with zero phases is a defect.
- `AGENTS.md` states that every phase boundary reports what the phase
  produced, what remains, and what the next phase adds, and that it is an
  approval gate: the next phase starts only after an affirmative answer in the
  current conversation. The task request, an earlier "continue", the plan
  approval, silence, a timeout, a resumed checkpoint, or an automated retry is
  not approval. The last phase has no transition gate.
- `AGENTS.md` **Human Decision Gates** lists continuing from one phase to the
  next.
- `workflows/session-resilience.md` defines the **Phase Plan** format, the
  derivation of the sequence from the task, the **Phase boundary report and
  phase gate**, and states that the artifacts are session state requiring no
  commit.
- `workflows/task-to-code.md` Phase 9 publishes the plan in every task and
  references the phase gate.
- `workflows/output-compliance.md` `CHECK-002` requires the boundary report to
  state what the next phase adds and, when another phase remains, to ask for
  approval to start it; a report for the last phase is exempt from the
  question.
- `commands/task.md` and `agents/orchestrator.md` state the phase gate and
  reject the task request or an earlier instruction to continue as approval.
- The phase plan and the task plan artifact are not the same thing: the
  plan states what the task does, the phase plan states the order it
  happens in.
- The phase plan and the phase checklist are not the same thing: the
  checklist derives from the task plan and the file responsibility map, and
  `checklist_not_a_plan` rejects a phase plan passed as a checklist.
- The phase names rejected as checklist steps are read from
  `workflows/task-to-code.md`, so the check never drifts from its source of
  truth.
- `references/system-workflow-regression-matrix.md` includes R-69.

```sh
failures=0
for f in AGENTS.md workflows/session-resilience.md workflows/task-to-code.md \
         commands/task.md agents/orchestrator.md; do
  if ! tr '\n' ' ' < "$f" | tr -s ' ' ' ' | grep -Fqi 'phase gate'; then
    printf 'FAIL V2.47: %s does not define the phase gate\n' "$f"
    failures=1
  fi
done
exit "$failures"
```

## V2.48 Regression — Feature Responsibility Folders

Verify that a feature is a boundary with explicit folders, not a pile of
files:

- `AGENTS.md` states that the feature root holds only its responsibility
  folders and its public API, that a component, a hook, a service, or a
  contract never sits at the feature root, and that a component and a
  contract never share a folder.
- `references/folder-architectures.md` defines the feature tree with
  `components/`, `hooks/`, `services/`, `types/`, `context/`, and the public
  API (`index.ts` or the ecosystem equivalent), the table that maps each
  artifact to its responsibility folder, the
  consumer-group rule for contracts, and a concrete tree for every supported
  ecosystem: TypeScript, JavaScript, React, Next.js, Angular, Vue, Svelte,
  Solid, Java, Kotlin, Python, Go, Rust, C#/.NET, PHP, and Ruby.
- `skills/architecture-advisor/SKILL.md` references the canonical reference
  instead of restating the per-ecosystem table.
- `skills/implementation-recipes/SKILL.md` places feature-level contracts in
  `types/` and unit-local contracts in the unit folder, and gives the
  responsibility folders for React, Angular, and Vue.
- `references/project-creation-order.md` records the responsibility folder
  with the owner, and states that a feature root never holds a source file.
- `workflows/output-compliance.md` checks STRUCT-006 (the feature root holds
  only responsibility folders and its public API) and STRUCT-007 (a contract
  is not mixed with the units of a responsibility folder), and STRUCT-001
  checks the unit folder inside a responsibility folder.
- `workflows/code-audit.md` includes the responsibility-folder check and the
  mixed component/contract finding.
- `references/mandatory-rules.md` includes M-30.
- `references/system-workflow-regression-matrix.md` includes R-70.

## V2.49 Regression — Design-Pattern Selection

Verify that pattern selection is signal-driven, language-neutral, and
count-agnostic:

- `AGENTS.md` contains M-31 and the anchor `design forces`, and states that
  a task may need 0, 2, 32, or 100+ patterns with no maximum.
- `references/design-patterns.md` exists and contains: the design-force
  model; a language-neutral signal taxonomy; a catalog of at least 100
  patterns across the GoF, enterprise, integration, architectural,
  concurrency, functional, UI, reliability, data, and security families;
  the selection procedure with rejection and composition; the count
  semantics; the stack-neutrality rules and translation examples; the eight
  worked flows with counts 2, 4, 5, 6, 7, 5, 5, and 9; a 32-pattern scale
  example; the validation checklist; and the sources.
- `skills/architecture-advisor/SKILL.md` delegates to the reference, requires
  the **design forces** and evidence, and rejects a pattern without a
  signal.
- `skills/complexity-advisor/SKILL.md` scales the depth of pattern analysis
  by rigor without capping or padding the count, and states that the task's
  **design forces** are the only input.
- `workflows/task-to-code.md` Phase 8 requires the pattern selection record
  (forces, signals, selected, rejected, stack translation, count) before the
  plan.
- `workflows/task-planning.md` includes the selection in the plan.
- `workflows/code-audit.md` checks `PATTERN_UNJUSTIFIED`, `PATTERN_UNUSED`,
  `PATTERN_DUPLICATE`, composition, stack translation, and the derived
  count.
- `skills/engineering-review/SKILL.md` checks the pattern selection against
  the code.
- `references/official-documentation.md` contains the GoF, Fowler EAA, and
  Enterprise Integration Patterns sources.
- `references/mandatory-rules.md` includes M-31.
- `references/system-workflow-regression-matrix.md` includes R-71.
- `commands/task.md` and `agents/orchestrator.md` reference the selection and
  contain the anchor `design forces`.
- The count is never presented as a target: the same mechanism yields 2 for
  a small flow and 32+ for a program, and the catalog has at least 100
  patterns.

## V2.50 Regression — Independent Mode of Our Own Deliverable

Verify that the independent audit of a deliverable we produced is a mode of
the single code-audit flow, not a second flow:

- `workflows/code-audit.md` defines the **Independent Mode** with the
  independence contract (I-1 to I-7), the allowed and denied input lists,
  Phases I0 to I3, and the independent verdict rule
- the Independent Mode states that it is not a third scope mode, and the
  scope is still chosen in its Phase 2
- Phase I0 does not stop the audit when the context already contains
  implementation history: it marks independence `NOT ESTABLISHED`, withholds the
  implementation material, records it in the Input Manifest, and caps the verdict
  at `NOT VERIFIED` with the reason `independence not established`
- Phase I1 rebuilds the acceptance criteria from the original requirement and
  forbids deriving them from the implementation
- Phase I2 re-executes the gates through `workflows/project-validation.md` and
  the structural checks through `workflows/output-compliance.md`, and a skipped
  check is `NOT VERIFIED`, never a pass
- Phase I3 falsifies every delivery claim with `CONFIRMED`, `FALSIFIED`, or
  `NOT VERIFIED`; a falsified claim is a P1 finding
- the verdict taxonomy is the single one of `workflows/code-audit.md`:
  `PASS`, `PASS WITH FINDINGS`, `FAIL`, or `NOT VERIFIED`
- `workflows/external-audit.md`, `commands/external-audit.md`, and
  `skills/external-audit/SKILL.md` do not exist: the flow is not duplicated
- `agents/external-auditor.md` exists with `mode: all`, follows the Independent
  Mode of `workflows/code-audit.md` without restating it, states that a fresh
  subagent or a new session is the path to independence, states that switching
  mode inside the implementing session does not create independence, and does
  not reference a command that no longer exists
- `commands/code-audit.md` states that auditing our own deliverable is not a
  scope and names the `external-auditor` agent
- the Independent Mode is distinct from `pull-request-review` (a colleague's
  PR), `project-validation` (gate execution), and `system-audit` (the system
  itself)
- `agents/reviewer.md` is unchanged in behavior and `code-audit` keeps its
  scope and agent
- `README.md` describes the Independent Mode of `/code-audit` and does not list
  `/external-audit`
- `references/system-workflow-regression-matrix.md` includes R-72
- the behavioral pilot is reproducible: the `external-auditor` agent started as
  a fresh subagent without any command passes Phase I0 and reads the Independent
  Mode of `workflows/code-audit.md`
- the `external-auditor` agent has no `edit` capability (`edit` denied, `subagent`
  denied), has shell, webfetch, external directories, and browser or Code Mode
  execution allowed, and denies the shell commands that write, publish, or
  install packages
- a contaminated context does not stop the audit: the agent completes the audit,
  records the contamination in the Input Manifest, and caps the verdict
- no new package.json, dependency, or competing orchestrator was introduced

## V2.51 Regression — Opt-In Engineering Baseline

Verify that the engineering baseline exists, is opt-in, and makes onboarding
work from the repository itself instead of depending on an existing team
member explaining the project from memory, without overwriting existing
conventions:

- `workflows/engineering-baseline.md` exists and its Purpose describes the
  covered failure in plain language: onboarding that only works when an
  existing team member explains the project from memory, and the reduction of
  repeated initial configuration
- the workflow defines the adoption contract B-1 to B-9: opt-in, asked once,
  additive and incremental, one module at a time, existing convention wins,
  conflict reported, verified not declared, no package without the gate, and
  no remote write or commit
- the **single question** is defined once, with the four answers, and states
  the conditions under which it is never asked (documentation-only project,
  read-only intent, existing record, already answered in the session); the
  "choose modules" path asks which modules without preselecting
- the answer mapping is explicit: acceptance and "never" write
  `docs/engineering-baseline.md`; "not now" writes nothing and asks again on
  a later task
- the record path, frontmatter (`status`, `version`, `date`, `modules`), the
  status values, and the module values are defined; a module is `adopted`
  only with verification evidence
- the six modules exist with their artifacts and verification: `onboarding`,
  `docs-gate`, `decisions`, `bootstrap`, `devops`, `release`; the recommended
  order is `onboarding`, `docs-gate`, `decisions`, `bootstrap`, `devops`,
  `release`
- new-project scheduling is separate from the recommended adoption order and
  places each module at the earliest creation step where its verification is
  possible: `decisions` after `A2`, `onboarding`, `bootstrap`, and `docs-gate`
  after `A8`, `devops` after `A9`, and `release` after `A9` and `A10`
- the diagnosis is read-only and classifies every module as `Adopt`,
  `Adapt`, `Conflict`, or `Skip`, with evidence
- the workflow forbids restructuring the repository, overwriting a working
  file without an approved diff, shadowing existing scripts, and duplicating
  `workflows/secure-delivery.md`
- the `docs-gate` module requires an executable `scripts/check-docs`, wired
  into CI and `/validate`, that fails on a deliberate break and passes on the
  restore; prose is not accepted as the gate
- the required set of `scripts/check-docs` is derived from the record, so a
  module that is `pending`, `declined`, or `n/a` is never checked and a
  partial adoption is not failed for documents it never selected or has not
  applied yet
- the newcomer test is defined as a clean copy plus a fresh session that
  follows only `docs/onboarding.md`; pass requires zero questions and a green
  suite, and a static reading is not a newcomer test
- `commands/adopt-baseline.md` exists, uses `agent: orchestrator`, delegates
  to the workflow without restating it, and never creates an agent
- `skills/engineering-baseline/SKILL.md` exists and makes the flow
  discoverable in clients that load Skills natively
- `workflows/task-to-code.md` Phase 1 contains the single gate with the four
  conditions and treats an accepted adoption as a separate change set
- `workflows/project-initialization.md` Phase 3 asks the question in the
  startup decision set, records the answer, and Phase 4 instantiates the
  selected modules at the earliest creation step where each can be verified,
  following the workflow's new-project scheduling
- `AGENTS.md` contains M-32 and the anchor `opt-in engineering baseline`, and
  the Human Decision Gates list includes the opt-in baseline with the same
  opt-in, asked-once, separate-change-set, and "never" rules
- `references/mandatory-rules.md` includes M-32 with its canonical anchor and
  required surfaces
- the baseline is distinct from `project-validation`, `secure-delivery`,
  the Independent Mode of `code-audit`, and `system-audit`
- `README.md` lists `/adopt-baseline` and distinguishes it from
  `/create-project`, `/validate`, and the Independent Mode of `/code-audit`
- `commands/task.md` and `agents/orchestrator.md` reference the **opt-in
  engineering baseline** gate at the entry point, with the four conditions and
  the four answers; a task surface that never mentions it is a `FAIL`
- `references/system-workflow-regression-matrix.md` includes R-73
- the out-of-scope declaration names incident response, infrastructure
  provisioning, capacity and cost, privacy and compliance, and product
  analytics, instead of implying coverage
- no package.json, dependency, or `.env.example` was added to this
  documentation-only repository


```sh
failures=0
for f in commands/task.md agents/orchestrator.md; do
  grep -qi "opt-in engineering baseline" "$f" ||
    { echo "FAIL V2.51: $f does not surface the baseline gate"; failures=1; }
done
grep -q "The Single Question" workflows/engineering-baseline.md ||
  { echo "FAIL V2.51: the canonical question is missing"; failures=1; }
exit "$failures"
```

## V2.52 Regression — Design Quality and Dependency Direction

Verify that design quality is specified and that dependency direction is
verified mechanically:

- `AGENTS.md` states the Staff bar for design: SOLID, cohesion and coupling,
  explicit contracts, composition over inheritance, the simplest design, and a
  recorded pattern selection.
- `skills/engineering-review/SKILL.md` defines the five SOLID criteria with the
  tags `SOLID_SRP`, `SOLID_OCP`, `SOLID_LSP`, `SOLID_ISP`, and `SOLID_DIP`,
  plus language notes.
- `workflows/code-audit.md` Dimension 3 applies the SOLID check with
  `file:line` evidence, and Dimension 1 requires `LAYER-001`.
- `workflows/output-compliance.md` defines `LAYER-001` with the forbidden
  dependency edges per layer and executable detection for TypeScript and
  JavaScript, Java and Kotlin, Python, C#/.NET, PHP, and Ruby.
- `references/system-workflow-regression-matrix.md` includes R-74.
- Proof: a fixture whose `domain/` imports `infrastructure/` is detected, and
  the fixed fixture is clean.

```sh
failures=0
grep -qi "Staff bar" AGENTS.md || { echo "FAIL V2.52: AGENTS.md has no design bar"; failures=1; }
grep -qi "SOLID" skills/engineering-review/SKILL.md || { echo "FAIL V2.52: no SOLID criteria"; failures=1; }
grep -qi "LAYER-001" workflows/code-audit.md workflows/output-compliance.md || { echo "FAIL V2.52: no LAYER-001"; failures=1; }
exit "$failures"
```

## V2.53 Regression — Clean Architecture Mapping and Unit Style

Verify that Clean and Hexagonal are documented as one dependency rule and
that functions versus classes is an ecosystem decision, not an architecture
requirement:

- `references/folder-architectures.md` contains the **Clean Architecture
  mapping** table, the TypeScript/Node tree, and the per-ecosystem
  Hexagonal/Clean table.
- `skills/implementation-recipes/SKILL.md` contains **Unit style: functions
  or classes** with the per-ecosystem table, the port/adapter rules, and the
  prohibition on mixing styles inside one layer.
- `skills/architecture-advisor/SKILL.md` states that Clean and Hexagonal are
  the same dependency rule and that the unit style is the ecosystem's idiom.
- `references/system-workflow-regression-matrix.md` includes R-75.
- Proof: a Clean fixture with functions and one with classes both pass; a
  `domain/` importing `infrastructure/` is detected.

```sh
failures=0
grep -q "Clean Architecture mapping" references/folder-architectures.md || { echo "FAIL V2.53: no Clean mapping"; failures=1; }
grep -q "Hexagonal / Clean, per ecosystem" references/folder-architectures.md || { echo "FAIL V2.53: no per-ecosystem table"; failures=1; }
grep -q "Unit style: functions or classes" skills/implementation-recipes/SKILL.md || { echo "FAIL V2.53: no unit-style table"; failures=1; }
grep -q "same dependency rule" skills/architecture-advisor/SKILL.md || { echo "FAIL V2.53: no equivalence statement"; failures=1; }
exit "$failures"
```

## V2.54 Regression — Live Phase Checklist and Native Task Surface

Verify that the checklist is a live view, emitted at every step transition,
and that its client mirror degrades honestly:

- `AGENTS.md` states that the checklist is live, leads every update, and is
  re-emitted at every step transition, and that a client-rendered panel such
  as the MCP connection list is not a writable surface.
- `workflows/session-resilience.md` contains **Emission and cadence** with the
  transition table, and **Native task surfaces** with the conversation copy as
  the authority, the failure of the mirror reported, and the plugin path for a
  surface only a plugin can extend.
- `workflows/task-to-code.md` and `agents/orchestrator.md` carry the
  operational instruction, including the mirror and the prohibition on
  holding the checklist back until the phase ends.
- `references/mandatory-rules.md` includes M-35 and
  `references/system-workflow-regression-matrix.md` includes R-76.
- Proof: a session with the native surface available shows the checklist in
  the conversation and in the panel on the same transitions; a session without
  it reports the tool as absent instead of claiming the panel was updated.

```sh
failures=0
grep -q "every step transition" AGENTS.md || { echo "FAIL V2.54: no cadence in AGENTS.md"; failures=1; }
grep -q "not a writable surface" AGENTS.md || { echo "FAIL V2.54: no non-writable panel rule in AGENTS.md"; failures=1; }
grep -q "Emission and cadence" workflows/session-resilience.md || { echo "FAIL V2.54: no cadence section"; failures=1; }
grep -q "Native task surfaces" workflows/session-resilience.md || { echo "FAIL V2.54: no native surface section"; failures=1; }
grep -q "conversation copy is the authority" workflows/session-resilience.md || { echo "FAIL V2.54: no authority statement"; failures=1; }
grep -q "every step transition" workflows/task-to-code.md || { echo "FAIL V2.54: no cadence in task-to-code.md"; failures=1; }
grep -q "every step transition" agents/orchestrator.md || { echo "FAIL V2.54: no cadence in orchestrator.md"; failures=1; }
grep -q "M-35" references/mandatory-rules.md || { echo "FAIL V2.54: no M-35"; failures=1; }
grep -q "R-76" references/system-workflow-regression-matrix.md || { echo "FAIL V2.54: no R-76"; failures=1; }
grep -q "Encoding in the flat list" workflows/session-resilience.md || { echo "FAIL V2.54: no flat-list encoding rule"; failures=1; }
grep -q "step never uses a phase label" workflows/session-resilience.md || { echo "FAIL V2.54: no ban on phase labels as step prefixes"; failures=1; }
grep -q "Cadence of the mirror" workflows/session-resilience.md || { echo "FAIL V2.54: no per-step mirror cadence"; failures=1; }
exit "$failures"
```

## V2.55 Regression — Entry-Point Independence and Operating Contract

Verify that the plan and the checklist do not depend on a command, that no
tool call precedes them, and that they survive a degraded session:

- `AGENTS.md` states that the obligation is **independent of the entry point**
  and that no tool call is made before the phase plan and the initial
  checklist.
- `workflows/session-resilience.md` carries the same obligation, the checkpoint
  fields for the plan, the checklist, and the pending gate, the re-emission on
  recovery, and the compaction, restart, and model-switch contract.
- `commands/task.md` declares itself an adapter and restates the obligation
  without becoming its condition.
- `references/mandatory-rules.md` includes M-36 and
  `references/system-workflow-regression-matrix.md` includes R-77.
- Proof: a session started by a plain message shows the plan before its first
  tool call; a compacted or restarted session re-emits the checklist before
  continuing.

```sh
failures=0
grep -q "independent of the entry point" AGENTS.md || { echo "FAIL V2.55: no entry-point rule in AGENTS.md"; failures=1; }
grep -q "No tool call is made before the phase plan" AGENTS.md || { echo "FAIL V2.55: no ordering rule in AGENTS.md"; failures=1; }
grep -q "independent of the entry point" workflows/session-resilience.md || { echo "FAIL V2.55: no entry-point rule in session-resilience.md"; failures=1; }
grep -q "survives compaction" workflows/session-resilience.md || { echo "FAIL V2.55: no compaction contract"; failures=1; }
grep -q "The current checklist, with the state of every step" workflows/session-resilience.md || { echo "FAIL V2.55: checkpoint misses the checklist"; failures=1; }
grep -q "Re-emit the phase plan and the current checklist" workflows/session-resilience.md || { echo "FAIL V2.55: no re-emission on recovery"; failures=1; }
grep -q "independent of the entry point" commands/task.md || { echo "FAIL V2.55: task adapter does not restate the rule"; failures=1; }
grep -q "M-36" references/mandatory-rules.md || { echo "FAIL V2.55: no M-36"; failures=1; }
grep -q "R-77" references/system-workflow-regression-matrix.md || { echo "FAIL V2.55: no R-77"; failures=1; }
exit "$failures"
```

## V2.56 Regression — Arrow Function Scope and Legacy Code

Verify that the arrow functions convention applies to the code a task creates
and never rewrites code that already exists:

- `AGENTS.md` scopes the rule to the functions a task creates, states that a
  pre-existing declaration is never converted, and treats a violation that
  pre-existed the change set as a finding rather than a blocker.
- `skills/implementation-recipes/SKILL.md` carries the same scope.
- `references/linter-rules.md` documents the two patterns for an existing
  project with legacy declarations, so the setup does not turn working code
  red.
- `workflows/output-compliance.md` FUNC-001 reads the added lines of the
  change set and classifies a declaration present in the base revision as
  pre-existing.
- `workflows/project-validation.md` keeps a pre-existing violation out of the
  gate.
- `references/mandatory-rules.md` includes M-01 with the scope, and the
  regression matrix includes R-78.
- Proof: a diff that edits a `function` body without converting it passes; a
  new `function` declaration without a legitimate reason fails; a legacy repo
  keeps a green lint through a documented freeze.

```sh
failures=0
grep -q "A pre-existing declaration is never converted" AGENTS.md || { echo "FAIL V2.56: no scope in AGENTS.md"; failures=1; }
grep -q "pre-existed the change set" AGENTS.md || { echo "FAIL V2.56: no pre-existing rule in AGENTS.md"; failures=1; }
grep -q "governs the functions the task" skills/implementation-recipes/SKILL.md || { echo "FAIL V2.56: no scope in the recipes"; failures=1; }
grep -q "Existing projects with legacy declarations" references/linter-rules.md || { echo "FAIL V2.56: no legacy pattern"; failures=1; }
grep -q "added lines of the change set only" workflows/output-compliance.md || { echo "FAIL V2.56: FUNC-001 not scoped"; failures=1; }
grep -q "finding, not a gate failure" workflows/project-validation.md || { echo "FAIL V2.56: gate does not exempt pre-existing"; failures=1; }
grep -q "M-01" references/mandatory-rules.md || { echo "FAIL V2.56: no M-01"; failures=1; }
grep -q "R-78" references/system-workflow-regression-matrix.md || { echo "FAIL V2.56: no R-78"; failures=1; }
exit "$failures"
```

## V2.57 Regression — Delegated Work Mirrored Into the Delegating Panel

Verify that a delegated phase or step is visible in the panel of the agent the
user is talking to, not only in the subagent's own session:

- `AGENTS.md` states that delegated work is mirrored into the delegating
  agent's panel and that a delegated result without a checklist is incomplete.
- `workflows/session-resilience.md` carries **Delegated work** with the five
  steps: the mirror before the delegation, the prompt that requires the
  checklist, the rewrite on every return, the no-native-surface path, and the
  incomplete result.
- `workflows/task-to-code.md` and `agents/orchestrator.md` carry the operational
  instruction, referencing the session-resilience section instead of restating
  it.
- `references/mandatory-rules.md` includes M-38 and
  `references/system-workflow-regression-matrix.md` includes R-79.
- Proof: a delegation to a subagent produces a returned checklist, the
  delegating panel is rewritten from it, and a return without a checklist is
  rejected. The check verifies the wording; it cannot observe the user's panel,
  which is reported as a limit.

```sh
failures=0
grep -qi "the delegating agent mirrors" AGENTS.md || { echo "FAIL V2.57: no M-38 wording in AGENTS.md"; failures=1; }
grep -q "is incomplete" AGENTS.md || { echo "FAIL V2.57: no incomplete-result rule in AGENTS.md"; failures=1; }
grep -q "Delegated work" workflows/session-resilience.md || { echo "FAIL V2.57: no delegated work section"; failures=1; }
grep -q "is incomplete" workflows/session-resilience.md || { echo "FAIL V2.57: no incomplete rule in session-resilience.md"; failures=1; }
grep -qi "the delegating agent mirrors" workflows/task-to-code.md || { echo "FAIL V2.57: no delegation instruction in task-to-code.md"; failures=1; }
grep -qi "the delegating agent mirrors" agents/orchestrator.md || { echo "FAIL V2.57: no delegation instruction in orchestrator.md"; failures=1; }
grep -q "M-38" references/mandatory-rules.md || { echo "FAIL V2.57: no M-38"; failures=1; }
grep -q "R-79" references/system-workflow-regression-matrix.md || { echo "FAIL V2.57: no R-79"; failures=1; }
exit "$failures"
```

## V2.58 Regression — New Task Starts a New List

Verify that the checklist and its panel belong to the task in progress and
never accumulate the previous task's items:

- `AGENTS.md` states that a new task rewrites the checklist and the mirror from
  its own phase plan, that no item of a previous task remains even when it is
  unfinished, and that the reset is a task boundary and not a mid-task one.
- `workflows/session-resilience.md` carries **Task boundary and reset** with the
  rewrite before the first tool call, the unfinished-task path through the
  report and the checkpoint, the current-state rule, and the mid-task
  prohibition.
- `workflows/task-to-code.md` and `agents/orchestrator.md` carry the operational
  instruction and reference the session-resilience section.
- `references/mandatory-rules.md` includes M-40 and
  `references/system-workflow-regression-matrix.md` includes R-80.
- Proof: a second task started while the first is unfinished shows only the
  second task's phases and steps, and the first task's state is reported and
  preserved by its checkpoint. The check verifies the wording; the user's panel
  is reported as a limit.

```sh
failures=0
grep -q "A new task starts a new list" AGENTS.md || { echo "FAIL V2.58: no reset rule in AGENTS.md"; failures=1; }
grep -q "no item of a previous task remains" AGENTS.md || { echo "FAIL V2.58: no removal rule in AGENTS.md"; failures=1; }
grep -q "Task boundary and reset" workflows/session-resilience.md || { echo "FAIL V2.58: no reset section"; failures=1; }
grep -q "An unfinished previous task is not carried over" workflows/session-resilience.md || { echo "FAIL V2.58: no unfinished-task path"; failures=1; }
grep -q "Task boundary and reset" workflows/task-to-code.md || { echo "FAIL V2.58: no reset instruction in task-to-code.md"; failures=1; }
grep -q "A new task starts a new list" agents/orchestrator.md || { echo "FAIL V2.58: no reset instruction in orchestrator.md"; failures=1; }
grep -q "M-40" references/mandatory-rules.md || { echo "FAIL V2.58: no M-40"; failures=1; }
grep -q "R-80" references/system-workflow-regression-matrix.md || { echo "FAIL V2.58: no R-80"; failures=1; }
exit "$failures"
```

## V2.59 Regression — Discipline Reinforcement Requires a Behavioral Scenario

Verify that the disciplines adopted from an external methodology are present in
their canonical Skills and workflows, that the rule requiring a behavioral
scenario before a reinforcement is accepted is stated and registered, and that
the audit can prove the scenario by removing a required step:

- `AGENTS.md` states the rule (M-42) with the phrase `behavioral scenario`.
- `skills/test-driven-development/SKILL.md` carries the Iron Law, the Root
  Cause Before Fix procedure, and Verification Before Completion.
- `skills/pull-request/SKILL.md` carries Responding to Review Feedback.
- `workflows/task-to-code.md` Phase 5 asks one question at a time with a
  recommendation.
- `references/mandatory-rules.md` includes M-42 and
  `references/system-workflow-regression-matrix.md` includes R-81.
- Proof: a copy of the repository in which the Iron Law section is removed is
  detected by this check with `file:line` evidence, and the unmodified
  repository passes. The behavioral audit runs both halves in a temporary
  directory.

The check takes the repository root as its argument and defaults to the current
directory, so the removal probe can run against a copy.

```sh
root="${1:-.}"
failures=0
grep -q "behavioral scenario" "$root/AGENTS.md" || { echo "FAIL V2.59: M-42 is not stated in AGENTS.md"; failures=1; }
tdd="skills/test-driven-development/SKILL.md"
grep -q "^### The Iron Law" "$root/$tdd" || { echo "FAIL V2.59: $tdd is missing '### The Iron Law' section (expected after the Red-Green-Refactor overview)"; failures=1; }
grep -q "^### Root Cause Before Fix" "$root/$tdd" || { echo "FAIL V2.59: $tdd is missing '### Root Cause Before Fix' section"; failures=1; }
grep -q "^## Verification Before Completion" "$root/$tdd" || { echo "FAIL V2.59: $tdd is missing '## Verification Before Completion' section"; failures=1; }
grep -q "^## Responding to Review Feedback" "$root/skills/pull-request/SKILL.md" || { echo "FAIL V2.59: skills/pull-request/SKILL.md is missing '## Responding to Review Feedback' section"; failures=1; }
grep -q "one question at a time" "$root/workflows/task-to-code.md" || { echo "FAIL V2.59: no one-question rule in task-to-code.md"; failures=1; }
grep -q "^| M-42 |" "$root/references/mandatory-rules.md" || { echo "FAIL V2.59: no M-42 row"; failures=1; }
grep -q "^| R-81 |" "$root/references/system-workflow-regression-matrix.md" || { echo "FAIL V2.59: no R-81 row"; failures=1; }
exit "$failures"
```

## V2.60 Regression — Audit Criteria, Implementation Rules, and Decision Record Agree

Verify that the rules the independent audit judges are the same rules the
implementer is told to meet, that the pattern selection reaches the auditor as
an artifact, and that a dirty session can start a clean audit:

- `references/audit-pass-criteria.md` states the secondary-dependency rule, the
  module error model rule, and the documentation-claim rule.
- `skills/implementation-recipes/SKILL.md` states the secondary-dependency
  rule, the bound-at-cap rule, and the module error model rule.
- `workflows/code-audit.md` defines the Allowed artifact (decision note) and
  reads the selection from the change set in Dimension 2.
- `workflows/task-to-code.md` requires the decision note in the change set.
- `agents/external-auditor.md` denies the state directory and `reports/` as
  input.
- `references/mandatory-rules.md` and the regression matrix include R-82.
- Proof: a copy of the repository in which the secondary-dependency rule is
  removed from `references/audit-pass-criteria.md` is detected by this check
  with file:line evidence, and the unmodified repository passes.

The check takes the repository root as its argument and defaults to the current
directory.

```sh
root="${1:-.}"
failures=0
ap="references/audit-pass-criteria.md"
ir="skills/implementation-recipes/SKILL.md"
ca="workflows/code-audit.md"
tc="workflows/task-to-code.md"
ea="agents/external-auditor.md"
grep -q "secondary (optional) dependency that fails" "$root/$ap" || { echo "FAIL V2.60: $ap is missing the secondary-dependency rule"; failures=1; }
grep -q "unless the project has no error model" "$root/$ap" || { echo "FAIL V2.60: $ap is missing the project error model rule (M-81)"; failures=1; }
grep -q "Every guarantee stated in the documentation" "$root/$ap" || { echo "FAIL V2.60: $ap is missing the documentation-claim rule"; failures=1; }
grep -q "Secondary dependencies degrade" "$root/$ir" || { echo "FAIL V2.60: $ir is missing the secondary-dependency recipe"; failures=1; }
grep -q "Bounds tested at the limit" "$root/$ir" || { echo "FAIL V2.60: $ir is missing the bound-at-cap recipe"; failures=1; }
grep -q "Allowed artifact (decision record)" "$root/$ca" || { echo "FAIL V2.60: $ca is missing the Allowed artifact rule"; failures=1; }
grep -q "decision note in the change set" "$root/$tc" || { echo "FAIL V2.60: $tc does not require the decision note"; failures=1; }
grep -q "AI_ENGINEERING_STATE_HOME" "$root/$ea" || { echo "FAIL V2.60: $ea does not deny the state directory"; failures=1; }
grep -q "^| R-82 |" "$root/references/system-workflow-regression-matrix.md" || { echo "FAIL V2.60: no R-82 row"; failures=1; }
exit "$failures"
```

## V2.61 Regression — Selected Pattern Presentation and SOLID Coverage

Verify that the proposal shown before implementation names the design patterns
by name and states the five SOLID principles, not a count alone, for every
language and framework:

- `AGENTS.md` states the presentation record rule (M-45), with the anchor
  `presentation record`, and names SRP, OCP, LSP, ISP, and DIP.
- `references/design-patterns.md` contains the `## Presentation Record` section,
  the selected-patterns table with the `Count:` line, the rejected-candidates
  rule, the five SOLID rows with `APPLIES`, `PRESERVED`, and `NOT APPLICABLE`,
  and the `### SOLID per stack` table covering every supported stack family.
- `workflows/task-to-code.md`, `workflows/task-planning.md`,
  `commands/task.md`, `agents/orchestrator.md`, and
  `skills/engineering-review/SKILL.md` each reference the presentation record.
- `references/mandatory-rules.md` contains the M-45 row, and the regression
  matrix contains the R-84 row.
- Proof: a copy of the repository in which the SOLID table is removed from
  `references/design-patterns.md` is detected by this check with file:line
  evidence, and the unmodified repository passes.

The check takes the repository root as its argument and defaults to the current
directory.

```sh
root="${1:-.}"
failures=0
dp="references/design-patterns.md"
grep -q "presentation record" "$root/AGENTS.md" || { echo "FAIL V2.61: AGENTS.md does not state the presentation record rule"; failures=1; }
for p in SRP OCP LSP ISP DIP; do
  grep -q "$p" "$root/AGENTS.md" || { echo "FAIL V2.61: AGENTS.md does not name $p"; failures=1; }
done
grep -n "^## Presentation Record" "$root/$dp" >/dev/null || { echo "FAIL V2.61: $dp has no '## Presentation Record' section"; failures=1; }
grep -n "^### SOLID per stack" "$root/$dp" >/dev/null || { echo "FAIL V2.61: $dp has no '### SOLID per stack' table"; failures=1; }
grep -q "Count: N selected" "$root/$dp" || { echo "FAIL V2.61: $dp has no Count line"; failures=1; }
for status in APPLIES PRESERVED "NOT APPLICABLE"; do
  grep -q "$status" "$root/$dp" || { echo "FAIL V2.61: $dp does not define status $status"; failures=1; }
done
for p in SRP OCP LSP ISP DIP; do
  grep -q "^| $p | " "$root/$dp" || { echo "FAIL V2.61: $dp has no SOLID row for $p"; failures=1; }
done
for f in workflows/task-to-code.md workflows/task-planning.md commands/task.md agents/orchestrator.md skills/engineering-review/SKILL.md; do
  grep -qi "presentation record" "$root/$f" || { echo "FAIL V2.61: $f does not reference the presentation record"; failures=1; }
done
grep -q "^| M-45 |" "$root/references/mandatory-rules.md" || { echo "FAIL V2.61: no M-45 row"; failures=1; }
grep -q "^| R-84 |" "$root/references/system-workflow-regression-matrix.md" || { echo "FAIL V2.61: no R-84 row"; failures=1; }
exit "$failures"
```

## V2.62 Regression — Stage Is One Concern

Verify that the stage decomposition is decided by concern, not by a file count,
and that the concern measurement works:

- `AGENTS.md` states that a stage is one concern and that the concern decides the
  split (anchor `the concern decides the split`).
- `workflows/session-resilience.md` defines the concern rule and the
  `### Concern measurement` block, and the block is executable: it is extracted
  from the document and run against two synthetic repositories created in a
  temporary directory, with no folder convention (no `src/`). One directory of
  one unit must print one line; several directories of one unit must print at
  least two; two units must print two distinct units. A block that cannot tell
  these apart is a `FAIL`.
- `workflows/task-to-code.md` and `commands/task.md` state the concern rule
  (anchor `A stage is one concern`).
- `references/mandatory-rules.md` M-41 and `references/system-workflow-regression-matrix.md`
  R-83 carry the concern rule and reference V2.62.
- Proof: a copy of `AGENTS.md` without the concern rule is detected by the anchor
  check with file:line evidence, and the unmodified repository passes.

The check takes the repository root as its argument and defaults to the current
directory.

```sh
root="${1:-.}"
failures=0
sr="workflows/session-resilience.md"
grep -q "the concern decides the split" "$root/AGENTS.md" || { echo "FAIL V2.62: AGENTS.md does not state that the concern decides the split"; failures=1; }
for f in workflows/task-to-code.md commands/task.md; do
  tr '\n' ' ' < "$root/$f" | tr -s ' ' | grep -Fq "A stage is one concern" || { echo "FAIL V2.62: $f does not state the concern rule"; failures=1; }
done
grep -q "^### Concern measurement" "$root/$sr" || { echo "FAIL V2.62: $sr has no Concern measurement section"; failures=1; }
grep -q "^| M-41 |.*a stage is one concern" "$root/references/mandatory-rules.md" || { echo "FAIL V2.62: M-41 does not carry the concern rule"; failures=1; }
grep -q "^| R-83 |.*V2.62" "$root/references/system-workflow-regression-matrix.md" || { echo "FAIL V2.62: R-83 does not reference V2.62"; failures=1; }
snippet=$(sed -n '/^### Concern measurement/,/^## Phase Checklist/p' "$root/$sr" |
  awk '/^```sh$/{f=1; next} /^```$/{if (f) exit} f')
if [ -z "$snippet" ]; then
  echo "FAIL V2.62: $sr has no executable concern measurement block"
  failures=1
else
  # Fixtures carry a committed baseline, so only the increment's change set is
  # measured. They use no folder convention: a flat package, a package with
  # several directories, and two units (services) in one repository.
  fx=$(mktemp -d "${TMPDIR:-/tmp}/v262.XXXXXX")
  git_fx() { git -c user.name=v262 -c user.email=v262@local "$@"; }
  base() { (cd "$1" && git init -q && git add -A && git_fx commit -qm base); }
  one="$fx/one-dir"; multi="$fx/multi-dir"; units="$fx/two-units"
  mkdir -p "$one/lib" "$multi/lib" "$multi/features/x" "$units/services/a" "$units/services/b"
  printf '{}\n' > "$one/package.json"; printf 'x\n' > "$one/lib/a.ts"; printf 'x\n' > "$one/lib/b.ts"
  printf '{}\n' > "$multi/package.json"; printf 'x\n' > "$multi/lib/a.ts"; printf 'x\n' > "$multi/features/x/y.ts"
  printf 'module a\n' > "$units/services/a/go.mod"; printf 'module b\n' > "$units/services/b/go.mod"
  printf 'x\n' > "$units/services/a/main.go"; printf 'x\n' > "$units/services/b/main.go"
  base "$one"; base "$multi"; base "$units"
  printf 'y\n' >> "$one/lib/a.ts"; printf 'y\n' >> "$one/lib/b.ts"
  printf 'y\n' >> "$multi/lib/a.ts"; printf 'y\n' >> "$multi/features/x/y.ts"
  printf 'y\n' >> "$units/services/a/main.go"; printf 'y\n' >> "$units/services/b/main.go"
  s=$(sh -c "$snippet" sh "$one" | wc -l | tr -d ' ')
  m=$(sh -c "$snippet" sh "$multi" | wc -l | tr -d ' ')
  u=$(sh -c "$snippet" sh "$units" | awk '{print $2}' | sort -u | wc -l | tr -d ' ')
  rm -rf "$fx"
  [ "$s" -eq 1 ] || { echo "FAIL V2.62: one directory of one unit is not reported as one line (lines=$s)"; failures=1; }
  [ "$m" -ge 2 ] || { echo "FAIL V2.62: several directories of one unit are not detected (lines=$m)"; failures=1; }
  [ "$u" -eq 2 ] || { echo "FAIL V2.62: two units are not reported as two units (units=$u)"; failures=1; }
fi
exit "$failures"
```

## V2.63 Regression — Styling Approach by Framework and Timing

Verify that the styling approach of a new frontend or mobile project is the
first question of its creation, asked from the options of the detected
framework, that no option is offered for a framework that does not support it,
and that an existing project is detected and continued without a question:

- `AGENTS.md` states the rule (anchor `styling approach`, with the phrase
  `first question`) under Human Decision Gates, with the framework exclusion and
  the existing-project rule.
- `references/styling-options.md` defines the detection signals in order, the
  options of each framework with their verification status, and the excluded
  options with their reason. Its Angular section offers no Styled Components,
  Emotion, or NativeWind; its React Native section offers no CSS Modules or
  Sass; its Excluded table names Styled Components for Angular and CSS files for
  React Native.
- The timing is enforced: `references/project-creation-order.md` has a step C0
  (frontend) and E0 (mobile) before the scaffold step C1 and E1, and
  `workflows/project-initialization.md` has a Phase 2b before Phase 3 and before
  Phase 4.
- The superseded wordings in `references/mandatory-rules.md` (the retired
  frontend default, the mid-creation question, and the existing-project conflict
  question) are absent from the canonical content, checked by V2.63 and 6.6.
- The surfaces of M-46 carry the anchor `styling approach`.
- `references/system-workflow-regression-matrix.md` includes R-85.
- Proof: the block below runs over the canonical files and over temporary copies.
  A copy in which an incompatible option is injected into the Angular section is
  detected with `file:line`; a copy in which C0 is moved after C1 is detected as a
  timing defect; a copy without the Phase 2b heading is detected. The repaired
  copies pass. The temporary directory is removed.

The check takes the repository root as its argument and defaults to the current
directory.

```sh
root="${1:-.}"
failures=0
so="$root/references/styling-options.md"
pc="$root/references/project-creation-order.md"
pi="$root/workflows/project-initialization.md"

[ -f "$so" ] || { echo "FAIL V2.63: references/styling-options.md is missing"; failures=1; }
grep -q "^| M-46 |.*styling approach" "$root/references/mandatory-rules.md" || { echo "FAIL V2.63: M-46 is not in the registry"; failures=1; }
grep -q "^| R-85 |" "$root/references/system-workflow-regression-matrix.md" || { echo "FAIL V2.63: R-85 is not in the matrix"; failures=1; }
grep -qi "styling approach" "$root/AGENTS.md" || { echo "FAIL V2.63: AGENTS.md does not state the styling approach rule"; failures=1; }
tr '\n' ' ' < "$root/AGENTS.md" | tr -s ' ' | grep -qi "project is the first question of its creation" || { echo "FAIL V2.63: AGENTS.md does not state that the styling approach is the first question"; failures=1; }

for surface in references/styling-options.md references/project-startup-decisions.md references/project-creation-order.md workflows/project-initialization.md workflows/task-to-code.md skills/architecture-advisor/SKILL.md skills/stack-detection/SKILL.md commands/create-project.md agents/orchestrator.md; do
  grep -qi "styling approach" "$root/$surface" || { echo "FAIL V2.63: $surface does not carry the styling approach rule"; failures=1; }
done

# The registry and this check record the superseded wordings on purpose; they are excluded.
retired=$(grep -rlF -e "recommended approach with CSS custom" -e "conflict between two approaches is reported" -e "asked, once, in the startup" "$root" --include="*.md" | grep -v "references/mandatory-rules.md" | grep -v "workflows/system-validation.md" | grep -v "/node_modules/" || true)
[ -z "$retired" ] || { echo "FAIL V2.63: superseded styling wording still present in: $retired"; failures=1; }

grep -q "^| Angular | Styled Components" "$so" || { echo "FAIL V2.63: the Excluded table does not exclude Styled Components for Angular"; failures=1; }
grep -q "^| React Native | CSS files" "$so" || { echo "FAIL V2.63: the Excluded table does not exclude CSS files for React Native"; failures=1; }

prev=0
for fw in "Ionic" "Angular" "React Native" "Next.js" "Vue" "React (web)"; do
  n=$(grep -n "^| [0-9] | $fw |" "$so" | head -1 | cut -d: -f1)
  if [ -z "$n" ] || [ "$n" -le "$prev" ]; then
    echo "FAIL V2.63: detection order is missing or broken at $fw in references/styling-options.md"
    failures=1
  else
    prev=$n
  fi
done

# timing FILE: the styling step must exist and precede the scaffold step of each
# UI project type, and the Phase 2b heading must precede Phase 3 and Phase 4.
timing() {
  c0=$(grep -n "^| C0 | Styling approach" "$1" | head -1 | cut -d: -f1)
  c1=$(grep -n "^| C1 | " "$1" | head -1 | cut -d: -f1)
  e0=$(grep -n "^| E0 | Styling approach" "$1" | head -1 | cut -d: -f1)
  e1=$(grep -n "^| E1 | " "$1" | head -1 | cut -d: -f1)
  [ -n "$c0" ] && [ -n "$c1" ] && [ "$c0" -lt "$c1" ] || { echo "FAIL V2.63: $1 has no styling step C0 before the scaffold step C1"; return 1; }
  [ -n "$e0" ] && [ -n "$e1" ] && [ "$e0" -lt "$e1" ] || { echo "FAIL V2.63: $1 has no styling step E0 before the scaffold step E1"; return 1; }
  return 0
}
phases() {
  p2b=$(grep -n "^## Phase 2b " "$1" | head -1 | cut -d: -f1)
  p3=$(grep -n "^## Phase 3 " "$1" | head -1 | cut -d: -f1)
  p4=$(grep -n "^## Phase 4 " "$1" | head -1 | cut -d: -f1)
  [ -n "$p2b" ] && [ -n "$p3" ] && [ -n "$p4" ] && [ "$p2b" -lt "$p3" ] && [ "$p3" -lt "$p4" ] || { echo "FAIL V2.63: $1 does not place Phase 2b between Phase 2 and Phase 3"; return 1; }
  return 0
}
timing "$pc" || failures=1
phases "$pi" || failures=1

# forbid FILE START STOP PATTERN LABEL: reports every line of the section that
# offers an option matching PATTERN, with its file and line.
forbid() {
  awk -v start="$2" -v stop="$3" -v pat="$4" -v label="$5" '
    $0 == start { on = 1; next }
    on && index($0, stop) == 1 { on = 0 }
    on && $0 ~ pat { printf "FAIL V2.63: %s:%d: %s is offered for %s\n", FILENAME, NR, $0, label; bad = 1 }
    END { exit bad }
  ' "$1"
}

forbid "$so" "### Angular" "### React (web)" "Styled Components|Emotion|NativeWind" "Angular" || failures=1
forbid "$so" "### React Native (Expo or bare)" "### Ionic" "CSS Modules|Sass|Stylus" "React Native" || failures=1

# Behavioral proof on temporary copies.
fx=$(mktemp -d "${TMPDIR:-/tmp}/v263.XXXXXX")
cp "$so" "$fx/styling-options.md"
awk '{ print } index($0, "| Tailwind CSS | Utility classes in templates") == 1 { print "| Styled Components | Styles written in JavaScript | injected | NOT VERIFIED |" }' "$so" > "$fx/injected.md"
forbid "$fx/styling-options.md" "### Angular" "### React (web)" "Styled Components|Emotion|NativeWind" "Angular" >/dev/null || { echo "FAIL V2.63: the compliant copy is rejected"; failures=1; }
out=$(forbid "$fx/injected.md" "### Angular" "### React (web)" "Styled Components|Emotion|NativeWind" "Angular" 2>&1 || true)
printf '%s\n' "$out" | grep -q "^FAIL V2.63: $fx/injected.md:[0-9]*: | Styled Components" || { echo "FAIL V2.63: the injected Styled Components option was not detected with file:line"; failures=1; }
grep -v "| injected |" "$fx/injected.md" > "$fx/repaired.md"
forbid "$fx/repaired.md" "### Angular" "### React (web)" "Styled Components|Emotion|NativeWind" "Angular" >/dev/null || { echo "FAIL V2.63: the repaired copy is rejected"; failures=1; }

# Timing: the scaffold step moved before the styling step is a defect; the repaired copy passes.
awk '/^\| C0 \| Styling approach/{next} {print}' "$pc" > "$fx/creation-no-c0.md"
timing "$fx/creation-no-c0.md" >/dev/null 2>&1 && { echo "FAIL V2.63: a creation order without C0 is accepted"; failures=1; }
awk '/^\| C0 \| Styling approach/{held=$0; next} /^\| C1 \| /{print; print held; next} {print}' "$pc" > "$fx/creation-late.md"
timing "$fx/creation-late.md" >/dev/null 2>&1 && { echo "FAIL V2.63: a styling step placed after the scaffold C1 is accepted"; failures=1; }
timing "$pc" >/dev/null || { echo "FAIL V2.63: the canonical order is rejected"; failures=1; }
awk '/^## Phase 2b /{skip=1} /^## Phase 3 /{skip=0} !skip' "$pi" > "$fx/init-no-2b.md"
phases "$fx/init-no-2b.md" >/dev/null 2>&1 && { echo "FAIL V2.63: an initialization without Phase 2b is accepted"; failures=1; }
phases "$pi" >/dev/null || { echo "FAIL V2.63: the canonical initialization is rejected"; failures=1; }
rm -rf "$fx"

exit "$failures"
```

## V2.64 Regression — Audit Criteria Read Gate and Platform Criteria

Verify that the code audit reads the pass criteria before it judges any
dimension, that an unsettled criterion is `AMBIGUOUS` and not judged by the
auditor, and that the criteria cover the platform behavior a mobile audit
otherwise misses: assertions on platform-dependent branches, error text
associated with its field, and the keyboard dismissal path.

- `references/audit-pass-criteria.md` states the read gate, the `AMBIGUOUS`
  rule, the platform assertion criterion (Dimension 7), the error association
  criterion (Dimension 9), and the keyboard criterion (Dimension 13).
- `workflows/code-audit.md` requires the read before judging, in Phase 5.
- `references/system-workflow-regression-matrix.md` contains the R-86 row.
- Proof: a copy in which the error association criterion is removed is detected
  with the criteria section location, and the unmodified repository passes.

The check takes the repository root as its argument and defaults to the current
directory.

```sh
root="${1:-.}"
failures=0
ap="references/audit-pass-criteria.md"
ca="workflows/code-audit.md"
rmx="references/system-workflow-regression-matrix.md"

criteria_gate() {
  r="$1"
  flat_ap=$(tr '\n' ' ' < "$r/$ap" | tr -s ' ')
  flat_ca=$(tr '\n' ' ' < "$r/$ca" | tr -s ' ')
  miss=0
  printf '%s' "$flat_ap" | grep -q "is resolved by the user, not by the auditor" || { echo "FAIL V2.64: $ap is missing the AMBIGUOUS rule"; miss=1; }
  printf '%s' "$flat_ap" | grep -q "A dimension is judged only after this file has been read in full" || { echo "FAIL V2.64: $ap is missing the read gate"; miss=1; }
  printf '%s' "$flat_ap" | grep -q "Platform-dependent behavior has an assertion on its rendered result" || { echo "FAIL V2.64: $ap is missing the platform assertion criterion"; miss=1; }
  printf '%s' "$flat_ap" | grep -q "never used to carry error text" || { echo "FAIL V2.64: $ap is missing the error association criterion"; miss=1; }
  printf '%s' "$flat_ap" | grep -q "A keyboard type without a return key" || { echo "FAIL V2.64: $ap is missing the keyboard dismissal criterion"; miss=1; }
  printf '%s' "$flat_ca" | grep -q "Before judging any dimension" || { echo "FAIL V2.64: $ca does not require the read before judging"; miss=1; }
  grep -q "^| R-86 |" "$r/$rmx" || { echo "FAIL V2.64: no R-86 row"; miss=1; }
  return "$miss"
}

criteria_gate "$root" || failures=1

# Behavioral proof: a copy with the error association criterion removed must be
# detected, and the unmodified copy must pass.
fx=$(mktemp -d "${TMPDIR:-/tmp}/v264.XXXXXX")
mkdir -p "$fx/references" "$fx/workflows"
cp "$root/$ap" "$fx/$ap"
cp "$root/$ca" "$fx/$ca"
cp "$root/$rmx" "$fx/$rmx"
criteria_gate "$fx" >/dev/null || { echo "FAIL V2.64: the unmodified copy is rejected"; failures=1; }
sed -i.bak '/never used to carry error text/d; /description is never used/d; /error role, a label, or the platform/d' "$fx/$ap"
out=$(criteria_gate "$fx" 2>&1) && { echo "FAIL V2.64: a copy without the error association criterion is accepted"; failures=1; }
printf '%s\n' "$out" | grep -q "FAIL V2.64: $ap is missing the error association criterion" || { echo "FAIL V2.64: the removed criterion was not detected"; failures=1; }
section=$(grep -n "^### 9. Accessibility" "$fx/$ap" | cut -d: -f1)
echo "V2.64 probe detected: $fx/$ap section 9 starts at line ${section:-unknown}"
rm -rf "$fx"

exit "$failures"
```

## V2.65 Regression — Test API Imports and the Test Case Form

Verify that the test API rule is canonical in `AGENTS.md`, registered as M-47
with the anchor "imported explicitly from the module of the framework", and
that the detection in `workflows/output-compliance.md` (TEST-001 and TEST-002)
finds a test API used without an import and a case declared with `it`, with
`file:line` evidence, and accepts the compliant form.

- `AGENTS.md` states the canonical wording under Implementation and Validation.
- `references/mandatory-rules.md` contains the M-47 row, and the regression
  matrix contains the R-87 row.
- The implementation recipe and the TDD skill carry the import rule, and every
  TypeScript example in the TDD skill that calls `test(` imports it.
- Proof: the detection function is extracted from `workflows/output-compliance.md`
  itself, not copied, and run on temporary fixtures: a compliant file passes, a
  file with no import and an `it` case is detected with `file:line`, and the
  repaired file passes.

The behavioral proof needs `perl`, which the detection uses. Where it is
absent the check is `NOT VERIFIED`. The TypeScript compiler result (TS2593) is
not part of this check; it is established by the behavioral audit of
`workflows/system-audit.md` on a real Vitest project.

The check takes the repository root as its argument and defaults to the current
directory.

```sh
root="${1:-.}"
failures=0
ag="AGENTS.md"
ir="skills/implementation-recipes/SKILL.md"
tdd="skills/test-driven-development/SKILL.md"
oc="workflows/output-compliance.md"
rmx="references/system-workflow-regression-matrix.md"
mr="references/mandatory-rules.md"

grep -q "imported explicitly from the module of the framework" "$root/$ag" || { echo "FAIL V2.65: $ag is missing the M-47 canonical wording"; failures=1; }
grep -q "^| M-47 |" "$root/$mr" || { echo "FAIL V2.65: no M-47 row"; failures=1; }
grep -q "^| R-87 |" "$root/$rmx" || { echo "FAIL V2.65: no R-87 row"; failures=1; }
grep -q "Import every identifier the file uses" "$root/$ir" || { echo "FAIL V2.65: $ir is missing the import recipe"; failures=1; }
grep -q "Import every test API explicitly" "$root/$tdd" || { echo "FAIL V2.65: $tdd is missing the import bullet"; failures=1; }
grep -q "TEST-001" "$root/$oc" && grep -q "TEST-002" "$root/$oc" || { echo "FAIL V2.65: $oc does not define TEST-001 and TEST-002"; failures=1; }

# Every TypeScript example in the TDD skill that calls test( must import it.
awk '
  /^```typescript/ { on = 1; blk = ""; start = NR; next }
  /^```/ && on {
    if (blk ~ /test\(/ && blk !~ /import \{/) {
      printf "FAIL V2.65: %s:%d: a TypeScript example calls test without an import\n", FILENAME, start
      bad = 1
    }
    on = 0; next
  }
  on { blk = blk $0 "\n" }
  END { exit bad }
' "$root/$tdd" || failures=1

command -v perl >/dev/null 2>&1 || { echo "NOT VERIFIED V2.65: perl is not available, the behavioral proof was not run"; exit 2; }

fx=$(mktemp -d "${TMPDIR:-/tmp}/v265.XXXXXX")
awk '/^test_api_check\(\) \{/{on=1} on{print} on && /^\}$/{exit}' "$root/$oc" > "$fx/check.sh"
grep -q "^test_api_check() {" "$fx/check.sh" || { echo "FAIL V2.65: test_api_check is not defined in $oc"; failures=1; }

(
  . "$fx/check.sh" || exit 1
  printf "import { describe, expect, test } from 'vitest';\ndescribe('a', () => {\n  test('b', () => {\n    expect(1).toBe(1);\n  });\n});\n" > "$fx/compliant.test.ts"
  printf "describe('a', () => {\n  it('b', () => {\n    expect(1).toBe(1);\n  });\n});\n" > "$fx/violation.test.ts"
  test_api_check "$fx/compliant.test.ts" || { echo "FAIL V2.65: the compliant test is rejected"; exit 1; }
  out=$(test_api_check "$fx/violation.test.ts" 2>&1) && { echo "FAIL V2.65: the violating test is accepted"; exit 1; }
  printf '%s\n' "$out" | grep -q "FAIL TEST-001: $fx/violation.test.ts:1: describe is used without an explicit import" || { echo "FAIL V2.65: describe without an import is not detected with file:line"; exit 1; }
  printf '%s\n' "$out" | grep -q "FAIL TEST-002: $fx/violation.test.ts:2: the case is declared with it" || { echo "FAIL V2.65: an it case is not detected with file:line"; exit 1; }
  { printf "import { describe, expect, test } from 'vitest';\n"; sed 's/it(/test(/' "$fx/violation.test.ts"; } > "$fx/repaired.test.ts"
  test_api_check "$fx/repaired.test.ts" || { echo "FAIL V2.65: the repaired test is rejected"; exit 1; }
) || failures=1

echo "V2.65 probe: fixtures in $fx were checked and are removed"
rm -rf "$fx"

exit "$failures"
```

## V2.66 Regression — Unwired Controls, Dead Exports, Decision Records, and Modified-Test Scope

Verify that the four rules added after the external audit are canonical in
`AGENTS.md`, registered (M-48, M-49, M-50 and R-88, R-89, R-90), reflected in
the criteria and the security procedure, and enforced by checks that run on
fixtures:

- M-48: a control that is exported or documented but not called on the path
  from the trust boundary to the sink is detected by the security procedure
  ("Control wiring" in `skills/security-review/SKILL.md`) and by Dimensions 6
  and 7 of `references/audit-pass-criteria.md`.
- M-49: `DEAD-EXPORT-001` in `workflows/output-compliance.md` reports an export
  that a barrel re-exports and no module imports, with file:line, and accepts the
  same export once a module imports it.
- M-50: the absence of a decision record is a Dimension 2 `FAIL`.
- M-47 scope: `TEST-001` and `TEST-002` on a modified test file report only the
  lines the change set adds, so a pre-existing violation is a finding.

The behavioral proof needs `perl`; where it is absent the check is
`NOT VERIFIED`. The check takes the repository root as its argument.

```sh
root="${1:-.}"
failures=0
ag="AGENTS.md"
ap="references/audit-pass-criteria.md"
sr="skills/security-review/SKILL.md"
oc="workflows/output-compliance.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"

flat_ap=$(tr '\n' ' ' < "$root/$ap" | tr -s ' ')

flat_ag=$(tr '\n' ' ' < "$root/$ag" | tr -s ' ')
grep -qF "is a control only when every external input" <<< "$flat_ag" || { echo "FAIL V2.66: $ag is missing the M-48 wording"; failures=1; }
grep -qF "is imported by another module" <<< "$flat_ag" || { echo "FAIL V2.66: $ag is missing the M-49 wording"; failures=1; }
grep -qF "its record is in the change set" <<< "$flat_ag" || { echo "FAIL V2.66: $ag is missing the M-50 wording"; failures=1; }
for id in M-48 M-49 M-50; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.66: no $id row"; failures=1; }
done
for id in R-88 R-89 R-90; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.66: no $id row"; failures=1; }
done
grep -qF "Control wiring" "$root/$sr" || { echo "FAIL V2.66: $sr has no control-wiring step"; failures=1; }
grep -qF "reaches it on the path from the trust boundary to the sink (M-48)" <<< "$flat_ap" || { echo "FAIL V2.66: $ap Dimension 6 has no wiring criterion"; failures=1; }
grep -qF "is a \`FAIL\` of SOLID SRP" <<< "$flat_ap" || { echo "FAIL V2.66: $ap Dimension 3 has no SRP extraction criterion"; failures=1; }
grep -qF "never \`NOT VERIFIED\`" <<< "$flat_ap" || { echo "FAIL V2.66: $ap Dimension 2 does not make a missing record a FAIL"; failures=1; }
grep -q "DEAD-EXPORT-001" "$root/$oc" || { echo "FAIL V2.66: $oc does not define DEAD-EXPORT-001"; failures=1; }
grep -qF "on the lines the change set adds" "$root/$oc" || { echo "FAIL V2.66: $oc does not scope modified test files to added lines"; failures=1; }

command -v perl >/dev/null 2>&1 || { echo "NOT VERIFIED V2.66: perl is not available, the behavioral proof was not run"; exit 2; }

fx=$(mktemp -d "${TMPDIR:-/tmp}/v266.XXXXXX")
awk '/^imports_name\(\) \{/{on=1} on{print} on && /^\}$/{exit}' "$root/$oc" > "$fx/check.sh"
awk '/^dead_export_check\(\) \{/{on=1} on{print} on && /^\}$/{exit}' "$root/$oc" >> "$fx/check.sh"
awk '/^test_api_check\(\) \{/{on=1} on{print} on && /^\}$/{exit}' "$root/$oc" >> "$fx/check.sh"
grep -q "^dead_export_check() {" "$fx/check.sh" || { echo "FAIL V2.66: dead_export_check is not defined in $oc"; failures=1; }

(
  mkdir -p "$fx/project/src"
  cd "$fx/project" || exit 1
  . "$fx/check.sh" || exit 1
  printf "export const parse = (raw: string) => raw;\n" > src/parser.ts
  printf "export { parse } from './parser';\n" > src/index.ts
  printf "const run = () => 1;\nexport default run;\n" > src/app.ts
  out=$(dead_export_check . src/parser.ts)
  printf '%s\n' "$out" | grep -q "FAIL DEAD-EXPORT-001: src/parser.ts:1: parse is exported with no consumer outside a barrel" \
    || { echo "FAIL V2.66: an export reached only through a barrel is not detected with file:line"; exit 1; }
  printf "import { parse } from './parser';\nexport const run = () => parse('x');\n" > src/app.ts
  out=$(dead_export_check . src/parser.ts)
  [ -z "$out" ] || { echo "FAIL V2.66: an export imported by a module is still reported: $out"; exit 1; }
) || failures=1

(
  . "$fx/check.sh" || exit 1
  printf "describe('a', () => {\n  test('b', () => {\n    expect(1).toBe(1);\n  });\n  it('c', () => {});\n});\n" > "$fx/mod.test.ts"
  printf "5\n" > "$fx/added.txt"
  out=$(test_api_check "$fx/mod.test.ts" "$fx/added.txt" 2>&1)
  printf '%s\n' "$out" | grep -q "FAIL TEST-002: $fx/mod.test.ts:5: the case is declared with it" \
    || { echo "FAIL V2.66: an it case on an added line is not detected"; exit 1; }
  printf '%s\n' "$out" | grep -q "^FAIL TEST-001: $fx/mod.test.ts:1:" \
    && { echo "FAIL V2.66: a pre-existing violation on an unchanged line is reported as a FAIL"; exit 1; }
  printf '%s\n' "$out" | grep -q "^FINDING TEST-001: $fx/mod.test.ts:1:" \
    || { echo "FAIL V2.66: a pre-existing violation on an unchanged line is not recorded as a FINDING (M-47, R-145)"; exit 1; }
  exit 0
) || failures=1

echo "V2.66 probe: fixtures in $fx were checked and are removed"
rm -rf "$fx"

exit "$failures"
```

## V2.67 Regression — Contract Duplication, Scan Coverage, Probe Isolation, and Comment Accuracy

Verify that the four changes made after the second external audit are canonical
and reachable, and that the contract-duplication detection runs on fixtures:

- M-51: a scan that excludes inputs reports them as `NOT VERIFIED`. The wording is
  in `AGENTS.md`, registered in `references/mandatory-rules.md`, and present in the
  security procedure, the secure-delivery Secrets section, and Dimension 6.
- TYPE-004: a contract name declared in more than one file is a `FAIL` with
  `file:line` for each copy. The detection is run on fixtures: a TypeScript
  interface and a JSDoc `@typedef` declared in two files are detected, a name
  declared once is not, and a same-file declaration is not counted.
- Probe isolation: the test reference states that a negative probe is evidence
  only when its own rule rejects it, and it lists the warning sign.
- Comment accuracy: the criteria and the engineering review compare comments with
  the code they describe.

The fixture part needs only `awk` and `sh`, so it does not depend on a runtime.
The check takes the repository root as its argument.

```sh
root="${1:-.}"
failures=0
ag="AGENTS.md"
ap="references/audit-pass-criteria.md"
sr="skills/security-review/SKILL.md"
sd="workflows/secure-delivery.md"
oc="workflows/output-compliance.md"
er="skills/engineering-review/SKILL.md"
lr="references/linter-rules.md"
wgt="skills/test-driven-development/references/writing-good-tests.md"
co="workflows/code-audit.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' '; }

flat_ag=$(flat "$ag")
flat_ap=$(flat "$ap")
flat_er=$(flat "$er")
flat_sr=$(flat "$sr")
flat_sd=$(flat "$sd")

grep -qF "excludes inputs by extension, path, or size" <<< "$flat_ag" || { echo "FAIL V2.67: $ag is missing the M-51 wording"; failures=1; }
grep -qF "never as clean" <<< "$flat_ag" || { echo "FAIL V2.67: $ag is missing the M-51 never-clean wording"; failures=1; }
for id in M-51; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.67: no $id row"; failures=1; }
done
for id in R-91 R-92 R-93 R-94; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.67: no $id row"; failures=1; }
done
grep -qF "**Scan coverage.**" <<< "$flat_sr" || { echo "FAIL V2.67: $sr has no scan-coverage step"; failures=1; }
grep -qF "Scan coverage:" <<< "$flat_sd" || { echo "FAIL V2.67: $sd has no scan-coverage bullet"; failures=1; }
grep -qF "excludes inputs by extension, path, or size" <<< "$flat_ap" || { echo "FAIL V2.67: $ap Dimension 6 has no scan-coverage criterion"; failures=1; }
grep -qF "A comment, JSDoc, or document that states a behavior" <<< "$flat_ap" || { echo "FAIL V2.67: $ap has no comment-accuracy criterion"; failures=1; }
grep -qF "Comment and document accuracy" <<< "$flat_er" || { echo "FAIL V2.67: $er has no comment-accuracy bullet"; failures=1; }
grep -qF "Duplicate contract declarations" <<< "$flat_er" || { echo "FAIL V2.67: $er has no duplicate-contract bullet"; failures=1; }
grep -q "^### TYPE-004 " "$root/$oc" || { echo "FAIL V2.67: $oc does not define TYPE-004"; failures=1; }
grep -q "TYPE-004" "$root/$co" || { echo "FAIL V2.67: $co does not run TYPE-004 as evidence"; failures=1; }
grep -qF "| A contract name declared in one file only | No | Verified by \`TYPE-004\` |" "$root/$lr" || { echo "FAIL V2.67: $lr has no TYPE-004 row"; failures=1; }
grep -q "^## Isolate Each Negative Probe" "$root/$wgt" || { echo "FAIL V2.67: $wgt has no probe-isolation section"; failures=1; }
grep -qF "A negative probe passes only because another rule" "$root/$wgt" || { echo "FAIL V2.67: $wgt has no probe-isolation warning sign"; failures=1; }

command -v awk >/dev/null 2>&1 || { echo "NOT VERIFIED V2.67: awk is not available, the TYPE-004 fixture was not run"; exit 2; }

fx=$(mktemp -d "${TMPDIR:-/tmp}/v267.XXXXXX")
awk '/^duplicate_contract_check\(\) \{/{on=1} on{print} on && /^\}$/{exit}' "$root/$oc" > "$fx/check.sh"
grep -q "^duplicate_contract_check() {" "$fx/check.sh" || { echo "FAIL V2.67: duplicate_contract_check is not defined in $oc"; failures=1; }

(
  cd "$fx" || exit 1
  mkdir -p src
  . "$fx/check.sh" || exit 1
  printf '/**\n * @typedef {Object} TextEntry\n */\nexport const a = 1;\n' > src/workspace.mjs
  printf '/**\n * @typedef {Object} TextEntry\n */\nexport const b = 2;\n' > src/secrets.mjs
  printf 'export interface Shared { x: number }\n' > src/a.ts
  printf 'export type Shared = { y: string };\n' > src/b.ts
  printf 'export interface Once { x: number }\nexport interface Once2 { x: number }\n' > src/c.ts
  printf 'export interface Twice { x: number }\nexport type Twice = { y: number };\n' > src/d.ts
  out=$(duplicate_contract_check src/workspace.mjs src/secrets.mjs src/a.ts src/b.ts src/c.ts src/d.ts)
  printf '%s\n' "$out" | grep -q "FAIL TYPE-004: src/workspace.mjs:2: TextEntry is also declared in another file" \
    || { echo "FAIL V2.67: the JSDoc typedef in workspace.mjs is not detected with file:line"; exit 1; }
  printf '%s\n' "$out" | grep -q "FAIL TYPE-004: src/secrets.mjs:2: TextEntry is also declared in another file" \
    || { echo "FAIL V2.67: the JSDoc typedef in secrets.mjs is not detected with file:line"; exit 1; }
  printf '%s\n' "$out" | grep -q "FAIL TYPE-004: src/a.ts:1: Shared is also declared in another file" \
    || { echo "FAIL V2.67: the TypeScript interface is not detected with file:line"; exit 1; }
  printf '%s\n' "$out" | grep -q "FAIL TYPE-004: src/b.ts:1: Shared is also declared in another file" \
    || { echo "FAIL V2.67: the TypeScript type alias is not detected with file:line"; exit 1; }
  [ "$(printf '%s\n' "$out" | grep -c '^FAIL TYPE-004')" = 4 ] \
    || { echo "FAIL V2.67: a name declared once or in one file is reported: $out"; exit 1; }
  printf 'export const c = 3;\n' > src/secrets.mjs
  out=$(duplicate_contract_check src/workspace.mjs src/secrets.mjs src/c.ts)
  [ -z "$out" ] || { echo "FAIL V2.67: a name declared in one file is still reported: $out"; exit 1; }
  exit 0
) || failures=1

echo "V2.67 probe: fixtures in $fx were checked and are removed"
rm -rf "$fx"

exit "$failures"
```

## V2.68 Regression — Identity, Health Probe, and Audit Signals

Verify that the rules and checks added after the third external audit are
canonical, reachable, and effective:

- M-52: an identity read from a client-controlled value is not authentication.
  The wording is in `AGENTS.md`, registered, and present in the catalog, the
  security procedure, and Dimension 6. The entry-point sweep applies only when
  the project has an identity mechanism.
- The catalog names the concrete signals of the audit: identity from a
  client-controlled header or query, an ownership check that passes on a missing
  owner, a non-atomic check and write, a CORS wildcard with credentials, a
  non-production debug tool, owner-only file modes and stripped upload metadata,
  unreconciled payment amounts, and unlimited costly endpoints.
- M-53: a project entered by the system gets a read-only health probe. Its
  tracked-artifact command is run on a temporary git repository: a `.tsbuildinfo`
  and a `dist/` file are detected, and a source file is not.
- A typecheck whose errors are all under `node_modules` is a configuration
  failure in both the validation and the commit workflows.
- Open threat-model items and duplicated source trees are reported without
  blocking, and the duplicate-tree section exists.

The check takes the repository root as its argument. It needs `git` for the
fixture; without it the fixture is `NOT VERIFIED`, and the text checks still run.

```sh
root="${1:-.}"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
vc="skills/vulnerability-catalog/SKILL.md"
sr="skills/security-review/SKILL.md"
sd="workflows/secure-delivery.md"
pv="workflows/project-validation.md"
rs="skills/repository-strategy/SKILL.md"
ap="references/audit-pass-criteria.md"
co="workflows/code-audit.md"
cm="workflows/commit.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' '; }

need() {
  grep -qF -- "$2" <<< "$(flat "$1")" || { echo "FAIL V2.68: $1 is missing $3"; failures=1; }
}

need "$ag" "identifies its caller only from a credential the client cannot forge" "the M-52 wording"
need "$ag" "read-only health probe of the project" "the M-53 wording"
need "$ag" "never reported as passing" "the M-53 never-passing wording"
for id in M-52 M-53; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.68: no $id row"; failures=1; }
done
need "$vc" "x-user-id" "the identity-from-header signal"
need "$vc" "CWE-290" "the CWE-290 source of the identity signal"
need "$vc" "CWE-367" "the CWE-367 source of the check-and-write signal"
need "$vc" "CWE-942" "the CWE-942 source of the CORS signal"
need "$vc" "CWE-732" "the CWE-732 source of the file-mode signal"
need "$vc" "stripped on upload" "the metadata minimal fix"
need "$vc" "order reference is accepted without reconciling" "the payment reconciliation signal"
need "$vc" "no rate limit on authentication, payment, charge" "the rate-limit signal"
need "$vc" "**Identity source:**" "the identity-source cross-cutting check"
need "$sr" "**Entry-point sweep.**" "the entry-point sweep step"
need "$sr" "## Open threat-model items" "the open threat-model section"
need "$sd" "Report every critical and high finding of an executed audit" "the severity reporting bullet"
need "$sd" "Open threat-model items whose surface the change touches" "the threat-model reporting bullet"
need "$pv" "## Phase 8a — Project Health Probe" "the Phase 8a health probe"
need "$pv" "configuration failure, not a dependency defect" "the node_modules typecheck classification"
need "$pv" "DUPLICATE-TREE" "the duplicate-tree finding"
need "$pv" "A failing check is never reported as passing" "the probe never-passing rule"
need "$rs" "## Duplicated Source Across Repositories" "the duplicate-source section"
need "$ap" "identifies its caller only from a credential the client cannot forge (M-52)" "the Dimension 6 identity criterion"
need "$co" "identity comes from a credential the client cannot forge (M-52)" "the code-audit security row"
need "$cm" "configuration failure under \`workflows/project-validation.md\` (Phase 8)" "the commit typecheck classification"
for id in R-95 R-96 R-97 R-98 R-99; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.68: no $id row"; failures=1; }
done

command -v git >/dev/null 2>&1 || { echo "NOT VERIFIED V2.68: git is not available, the artifact fixture was not run"; exit 2; }

cmd="$(grep -m1 'git ls-files | grep -E' "$root/$pv" | sed 's/^[[:space:]]*//')"
[ -n "$cmd" ] || { echo "FAIL V2.68: $pv has no tracked-artifact command"; exit 1; }

fx="$(mktemp -d "${TMPDIR:-/tmp}/v268.XXXXXX")"
(
  cd "$fx" || exit 1
  git init -q . || exit 1
  mkdir -p src dist
  printf 'x\n' > tsconfig.tsbuildinfo
  printf 'x\n' > dist/a.js
  printf 'export const a = 1;\n' > src/a.ts
  git add . || exit 1
  out="$(sh -c "$cmd")"
  printf '%s\n' "$out" | grep -qx 'tsconfig.tsbuildinfo' \
    || { echo "FAIL V2.68: the documented command does not detect a tracked .tsbuildinfo: $out"; exit 1; }
  printf '%s\n' "$out" | grep -qx 'dist/a.js' \
    || { echo "FAIL V2.68: the documented command does not detect a tracked dist file: $out"; exit 1; }
  printf '%s\n' "$out" | grep -q 'src/a.ts' \
    && { echo "FAIL V2.68: the documented command reports a source file: $out"; exit 1; }
  exit 0
) || failures=1

echo "V2.68 probe: fixture in $fx was checked and is removed"
rm -rf "$fx"

exit "$failures"
```

## V2.69 Regression — Delivery Safety: Timeouts, Cleartext, Client Cache, Downgrade, Hardcoded Endpoints

Verify that the rules added after the external audit of the mobile and the
counter-case projects (M-54 to M-58) are canonical, reachable, and effective:

- M-54: every external call has an explicit timeout, for every project type.
- M-55: `http://` is rejected for every non-local build of a credential endpoint.
- M-56: client-side cached data is partitioned by the identity that owns it and
  cleared on sign-out.
- M-57: a dependency remediation never downgrades.
- M-58: no hardcoded environment endpoint and no fixed shared temporary path.
- The health probe enumerates every lockfile root and reports dead exports
  without blocking an unrelated task (Phase 8a, items 3, 6, and 7).

The text checks run on the repository. The behavioral part extracts the
`SAFE-001` to `SAFE-003` and `DEAD-EXPORT-001` functions from
`workflows/output-compliance.md`, runs them on a temporary fixture outside the
repository, verifies each violation is detected with `file:line`, repairs the
fixture, and verifies it passes. The fixture needs `awk`, `grep`, `sed`, and
`perl`; without them the behavioral part is `NOT VERIFIED`, and the text checks
still run.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
pv="workflows/project-validation.md"
dp="workflows/dependency-assessment.md"
ap="references/audit-pass-criteria.md"
ir="skills/implementation-recipes/SKILL.md"
sr="skills/security-review/SKILL.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.69: $1 is missing $3"; failures=1; }
}

need "$ag" "every external call has an explicit timeout" "the M-54 wording"
need "$ag" "is rejected for every non-local build" "the M-55 wording"
need "$ag" "partitioned by the identity that owns it" "the M-56 wording"
need "$ag" "a dependency remediation never downgrades" "the M-57 wording"
need "$ag" "no hardcoded environment endpoint" "the M-58 wording"
need "$ir" "every external call has an explicit timeout" "the M-54 recipe"
need "$ir" "AbortSignal.timeout" "the timeout mechanism in the recipe"
need "$ap" "This bullet is never \`N/A\`" "the M-54 applicability in Dimension 8"
need "$ap" "the waits and growth bullets apply to hot paths" "the narrowed hot-path scope"
need "$sr" "partitioned by the identity that owns it (M-56)" "the M-56 review step"
need "$dp" "## Remediation of Audit Findings" "the remediation section"
need "$dp" "a dependency remediation never downgrades" "the M-57 workflow rule"
need "$pv" "Enumerate every lockfile and every workspace root" "the per-root audit enumeration"
need "$pv" "findings from one root are never attributed to another" "the per-root attribution rule"
need "$pv" "DEAD-EXPORT-001" "the probe dead-export item"
need "$oc" "### SAFE-001, SAFE-002, SAFE-003" "the SAFE section"
for id in M-54 M-55 M-56 M-57 M-58; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.69: no $id row"; failures=1; }
done
for id in R-100 R-101 R-102 R-103 R-104; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.69: no $id row"; failures=1; }
done

command -v awk >/dev/null 2>&1 && command -v perl >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.69: awk or perl is not available, the behavioral fixture was not run"; exit 2; }

# Extracts the first sh block of a section of workflows/output-compliance.md.
extract() {
  awk -v start="$1" '
    $0 ~ "^### " start { on = 1; next }
    on && /^### / { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; exit }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v269.XXXXXX")"
(
  cd "$fx" || exit 1
  mkdir -p src scripts
  extract "SAFE-001, SAFE-002, SAFE-003" > safe.sh
  extract "DEAD-EXPORT-001" > dead.sh
  [ -s safe.sh ] && [ -s dead.sh ] || { echo "FAIL V2.69: the SAFE or DEAD-EXPORT block was not extracted"; exit 1; }
  . ./safe.sh
  . ./dead.sh

  cat > src/ok.ts <<'TS'
export const load = async (url: string) => {
  const response = await fetch(url, { signal: AbortSignal.timeout(5000) });
  return response.json();
};
export const base = process.env.EXPO_PUBLIC_API_BASE_URL ?? "https://api.example.com";
const emulator = "http://10.0.2.2:8080";
TS
  cat > src/bad.ts <<'TS'
export const load = async (url: string) => {
  const response = await fetch(url);
  return response.json();
};
export const base = "http://shop.example.net";
export const dev = "http://localhost:8080";
TS
  printf '%s\n' '#!/bin/sh' "LOG=/$(printf tmp)/shoplite-run.log" 'OK="${TMPDIR:-/tmp}/run.$$"' > scripts/run.sh
  printf 'export const clearCache = () => undefined;\n' > src/cache.ts
  printf "export * from './cache';\n" > src/index.ts
  printf 'import { load } from "./ok";\nconsole.log(load);\n' > src/consumer.ts

  # 1. The compliant artifacts pass.
  out="$(safe_timeout_check . src/ok.ts)$(safe_cleartext_check . src/ok.ts)$(safe_endpoint_check . src/ok.ts)"
  [ -z "$out" ] || { echo "FAIL V2.69: the compliant file is reported: $out"; exit 1; }

  # 2. The violations are detected with file:line.
  t="$(safe_timeout_check . src/bad.ts)"
  printf '%s\n' "$t" | grep -qx 'FAIL SAFE-001: src/bad.ts:2: external call with no timeout' \
    || { echo "FAIL V2.69: SAFE-001 did not detect src/bad.ts:2: $t"; exit 1; }
  c="$(safe_cleartext_check . src/bad.ts src/ok.ts)"
  printf '%s\n' "$c" | grep -q '^FAIL SAFE-002: src/bad.ts:5:' \
    || { echo "FAIL V2.69: SAFE-002 did not detect src/bad.ts:5: $c"; exit 1; }
  printf '%s\n' "$c" | grep -q '10.0.2.2' \
    && { echo "FAIL V2.69: SAFE-002 reports the emulator loopback: $c"; exit 1; }
  e="$(safe_endpoint_check . src/bad.ts scripts/run.sh)"
  printf '%s\n' "$e" | grep -q '^FAIL SAFE-003: src/bad.ts:6:' \
    || { echo "FAIL V2.69: SAFE-003 did not detect src/bad.ts:6: $e"; exit 1; }
  printf '%s\n' "$e" | grep -q '^FAIL SAFE-003: scripts/run.sh:2:' \
    || { echo "FAIL V2.69: SAFE-003 did not detect the fixed /tmp path: $e"; exit 1; }
  printf '%s\n' "$e" | grep -q 'run.sh:3:' \
    && { echo "FAIL V2.69: SAFE-003 reports the TMPDIR fallback: $e"; exit 1; }
  d="$(dead_export_check . src/cache.ts)"
  printf '%s\n' "$d" | grep -q 'FAIL DEAD-EXPORT-001: src/cache.ts:1: clearCache' \
    || { echo "FAIL V2.69: DEAD-EXPORT-001 did not detect clearCache: $d"; exit 1; }
  d="$(dead_export_check . src/ok.ts)"
  printf '%s\n' "$d" | grep -q 'load is exported' \
    && { echo "FAIL V2.69: DEAD-EXPORT-001 reports load, which consumer.ts imports"; exit 1; }
  printf '%s\n' "$d" | grep -q 'FAIL DEAD-EXPORT-001: src/ok.ts:5: base' \
    || { echo "FAIL V2.69: DEAD-EXPORT-001 did not detect the unused base export: $d"; exit 1; }

  # 3. The repaired artifact passes.
  printf '%s\n' \
    'export const load = async (url: string) => {' \
    '  const response = await fetch(url, { signal: AbortSignal.timeout(5000) });' \
    '  return response.json();' \
    '};' \
    'export const base = process.env.EXPO_PUBLIC_API_BASE_URL ?? "https://api.example.com";' > src/bad.ts
  out="$(safe_timeout_check . src/bad.ts)$(safe_cleartext_check . src/bad.ts)$(safe_endpoint_check . src/bad.ts)"
  [ -z "$out" ] || { echo "FAIL V2.69: the repaired file is still reported: $out"; exit 1; }
  exit 0
) || failures=1

echo "V2.69 probe: fixture in $fx was checked and is removed"
rm -rf "$fx"

exit "$failures"
```

## V2.70 Regression — Path Encoding, Rejected Credentials, Backup, and the Whole-Project Sweep

Verify that the rules added after the second external audit of the mobile and
demo projects are canonical, reachable, and effective, and that the delivery-safety
detectors cover the whole project and not only the change set:

- M-59: a value interpolated into a URL path is encoded with the encoder of its context.
- M-60: a rejected credential ends the session, and cached data is not served to it.
- M-61: an app that holds personal or payment data does not allow backup of personal data.
- The detectors `SAFE-002` (scheme form), `SAFE-004` to `SAFE-007`, `AUTH-001`, and
  `DEP-001` exist in `workflows/output-compliance.md`, and the health probe runs them
  over the whole project through `project_safety_sweep` (Phase 8a, item 6).

The text checks run on the repository. The behavioral part extracts the detector
and sweep blocks from `workflows/output-compliance.md`, builds a fixture git
repository outside the repository, verifies each violation is detected with
`file:line` and each compliant twin is not, runs the sweep, repairs the fixture,
verifies the repair passes, and removes the fixture. It needs `awk`, `perl`, `git`,
`grep`, and `sed`; without them the behavioral part is `NOT VERIFIED`, and the text
checks still run.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
pv="workflows/project-validation.md"
ir="skills/implementation-recipes/SKILL.md"
sr="skills/security-review/SKILL.md"
ap="references/audit-pass-criteria.md"
sa="workflows/system-audit.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.70: $1 is missing $3"; failures=1; }
}

need "$ag" "is encoded with the encoder of its context" "the M-59 wording"
need "$ag" "a rejected credential ends the session" "the M-60 wording"
need "$ag" "does not allow backup of personal" "the M-61 wording"
need "$ir" "is encoded with the encoder of its context" "the M-59 recipe"
need "$ir" "a rejected credential ends the session" "the M-60 recipe"
need "$ir" "does not allow backup of personal" "the M-61 recipe"
need "$sr" "a rejected credential ends the session" "the M-60 review step"
need "$sr" "does not allow backup of personal" "the M-61 review step"
need "$ap" "a rejected credential ends the session" "the M-60 Dimension 6 criterion"
need "$ap" "does not allow backup of personal" "the M-61 Dimension 6 criterion"
need "$oc" "### SAFE-004 to SAFE-007, AUTH-001, DEP-001" "the new detector section"
need "$oc" "### Whole-project sweep" "the sweep section"
need "$oc" "project_safety_sweep()" "the sweep function"
need "$oc" "url_scheme_check()" "the SAFE-002 scheme detector"
need "$oc" "id_encoding_check()" "the SAFE-004 detector"
need "$oc" "cache_partition_check()" "the SAFE-005 detector"
need "$oc" "signout_clear_check()" "the SAFE-005 sign-out detector"
need "$oc" "rejected_credential_check()" "the SAFE-006 detector"
need "$oc" "identity_source_check()" "the AUTH-001 detector"
need "$oc" "backup_check()" "the SAFE-007 detector"
need "$oc" "dependency_downgrade_check()" "the DEP-001 detector"
need "$oc" "no build manifest or app configuration is tracked" "the M-61 NOT VERIFIED rule"
need "$pv" "project_safety_sweep" "the probe running the sweep"
need "$pv" "not only the change set" "the whole-project scope of the probe"
need "$pv" "DEP-001" "the manifest downgrade check in the probe"
need "$sa" "A delivery-safety detector, or the whole-project sweep" "the audit area row"
for id in M-59 M-60 M-61; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.70: no $id row"; failures=1; }
done
for id in R-105 R-106 R-107; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.70: no $id row"; failures=1; }
done

command -v awk >/dev/null 2>&1 && command -v perl >/dev/null 2>&1 && command -v git >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.70: awk, perl, or git is not available, the behavioral fixture was not run"; exit 2; }

# Extracts the first sh block of a section of workflows/output-compliance.md.
extract() {
  awk -v start="$1" '
    $0 ~ "^### " start { on = 1; next }
    on && /^### / { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; exit }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v270.XXXXXX")"
(
  cd "$fx" || exit 1
  git init -q .
  git config user.email "fixture@example.invalid"
  git config user.name "fixture"
  mkdir -p src android/app/src/main
  extract "SAFE-001, SAFE-002, SAFE-003" > safe.sh
  extract "DEAD-EXPORT-001" > dead.sh
  { extract "SAFE-004 to SAFE-007"; extract "SAFE-008, SAFE-009"; } > new.sh
  # The sweep calls the V2.73 detectors too, so they are loaded with it.
  { extract "Whole-project sweep"
    awk '/^### (I18N-00[1-4]|DEC-001|CLAIM-001|A11Y-001|A11Y-002|DIP-002) / { on = 1; next }
      /^### / { on = 0 }
      on && /^```sh$/ { inb = 1; next }
      inb && /^```$/ { inb = 0; next }
      inb { print }' "$root/$oc"
  } > sweep.sh
  [ -s safe.sh ] && [ -s dead.sh ] && [ -s new.sh ] && [ -s sweep.sh ] || {
    echo "FAIL V2.70: a detector or sweep block was not extracted"; exit 1; }
  . ./safe.sh
  . ./dead.sh
  . ./new.sh
  . ./sweep.sh

  # Compliant twins.
  printf '%s\n' 'export const productPath = (productId: string) => `/api/v1/products/${encodeURIComponent(productId)}`;' > src/ok-path.ts
  printf '%s\n' 'import { z } from "zod";' 'export const apiBaseUrlSchema = z.string().url().refine((value) => value.startsWith("https://"), "https only");' > src/config-ok.ts
  printf '%s\n' 'export const load = (cache: any, userId: string) => {' '  cache.writeCache(`${userId}:cart`, {});' '};' > src/cache-ok.ts
  printf '%s\n' 'export const signOut = async (userId: string) => {' '  await deleteToken();' '  await clearCacheForIdentity(userId);' '};' > src/session-ok.ts
  printf '%s\n' 'export const check = async (response: Response, signOut: () => Promise<void>) => {' '  if (response.status === 401) {' '    await signOut();' '    return null;' '  }' '  return response.json();' '};' > src/auth-ok.ts
  printf '%s\n' '<manifest xmlns:android="http://schemas.android.com/apk/res/android">' '  <application android:allowBackup="false" android:label="app">' '  </application>' '</manifest>' > android/app/src/main/AndroidManifest.xml
  printf '%s\n' '{' '  "dependencies": {' '    "expo": "^54.0.0"' '  }' '}' > package.json

  # Violations.
  printf '%s\n' 'export const productPath = (productId: string) => `/api/v1/products/${productId}`;' > src/bad-path.ts
  printf '%s\n' 'import { z } from "zod";' 'export const apiBaseUrlSchema = z.string().url();' > src/config-bad.ts
  printf '%s\n' 'export const load = (cache: any) => {' '  cache.writeCache("cart", {});' '  cache.writeCache("payment-methods", {});' '};' > src/cache-bad.ts
  printf '%s\n' 'export const signOut = async () => {' '  await deleteToken();' '};' > src/session-bad.ts
  printf '%s\n' 'export const check = async (response: Response) => {' '  if (response.status === 401) {' '    return null;' '  }' '  return response.json();' '};' > src/auth-bad.ts
  printf '%s\n' 'export const userOf = (request: Request) => request.headers.get("x-user-id");' > src/identity-bad.ts
  printf '%s\n' '{ "expo": { "android": { "allowBackup": true } } }' > app.json
  git add -A && git commit -q -m base

  # 1. The compliant twins pass the detectors.
  out="$(url_scheme_check . src/config-ok.ts)$(id_encoding_check . src/ok-path.ts)$(cache_partition_check . src/cache-ok.ts)$(signout_clear_check . src/session-ok.ts)$(rejected_credential_check . src/auth-ok.ts)$(identity_source_check . src/ok-path.ts)"
  [ -z "$out" ] || { echo "FAIL V2.70: a compliant twin is reported: $out"; exit 1; }
  out="$(backup_check . android/app/src/main/AndroidManifest.xml)"
  [ -z "$out" ] || { echo "FAIL V2.70: the compliant manifest is reported: $out"; exit 1; }

  # 2. Each violation is detected with file:line.
  i="$(id_encoding_check . src/bad-path.ts)"
  printf '%s\n' "$i" | grep -q '^FAIL SAFE-004: src/bad-path.ts:1:' \
    || { echo "FAIL V2.70: SAFE-004 did not detect src/bad-path.ts:1: $i"; exit 1; }
  u="$(url_scheme_check . src/config-bad.ts)"
  printf '%s\n' "$u" | grep -q '^FAIL SAFE-002: src/config-bad.ts:2:' \
    || { echo "FAIL V2.70: SAFE-002 did not detect the scheme-free URL setting: $u"; exit 1; }
  c="$(cache_partition_check . src/cache-bad.ts)"
  printf '%s\n' "$c" | grep -q '^FAIL SAFE-005: src/cache-bad.ts:2:' \
    || { echo "FAIL V2.70: SAFE-005 did not detect the cache key without identity: $c"; exit 1; }
  printf '%s\n' "$c" | grep -q '^FAIL SAFE-005: src/cache-bad.ts:3:.*payment data' \
    || { echo "FAIL V2.70: SAFE-005 did not detect payment data in plain storage: $c"; exit 1; }
  s="$(signout_clear_check . src/session-bad.ts)"
  printf '%s\n' "$s" | grep -q '^FAIL SAFE-005: src/session-bad.ts:1:' \
    || { echo "FAIL V2.70: SAFE-005 did not detect the sign-out with no cache clear: $s"; exit 1; }
  a="$(rejected_credential_check . src/auth-bad.ts)"
  printf '%s\n' "$a" | grep -q '^FAIL SAFE-006: src/auth-bad.ts:2:' \
    || { echo "FAIL V2.70: SAFE-006 did not detect the 401 branch that keeps the session: $a"; exit 1; }
  d="$(identity_source_check . src/identity-bad.ts)"
  printf '%s\n' "$d" | grep -q '^FAIL AUTH-001: src/identity-bad.ts:1:' \
    || { echo "FAIL V2.70: AUTH-001 did not detect the identity from a header: $d"; exit 1; }
  b="$(backup_check . app.json)"
  printf '%s\n' "$b" | grep -q '^FAIL SAFE-007: app.json:1:' \
    || { echo "FAIL V2.70: SAFE-007 did not detect allowBackup true: $b"; exit 1; }
  printf '%s\n' '<manifest>' '  <application android:label="app">' '  </application>' '</manifest>' > android/app/src/main/LegacyManifest.xml
  b2="$(backup_check . android/app/src/main/LegacyManifest.xml)"
  printf '%s\n' "$b2" | grep -q '^FAIL SAFE-007: android/app/src/main/LegacyManifest.xml:1: allowBackup is not set' \
    || { echo "FAIL V2.70: SAFE-007 did not detect an unset allowBackup, which the Android default allows: $b2"; exit 1; }
  rm -f android/app/src/main/LegacyManifest.xml

  # 3. The manifest downgrade is detected against the base revision.
  printf '%s\n' '{' '  "dependencies": {' '    "expo": "^52.0.49"' '  }' '}' > package.json
  g="$(dependency_downgrade_check . HEAD package.json)"
  printf '%s\n' "$g" | grep -q '^FAIL DEP-001: package.json:3: expo moves from \^54.0.0 to \^52.0.49' \
    || { echo "FAIL V2.70: DEP-001 did not detect the downgrade: $g"; exit 1; }

  # 4. The whole-project sweep reports every violation, and no compliant twin.
  sw="$(project_safety_sweep)"
  printf '%s\n' "$sw" | grep -q '^SWEEP scanned: ' \
    || { echo "FAIL V2.70: the sweep did not state what it scanned"; exit 1; }
  for expect in \
    'FAIL SAFE-004: src/bad-path.ts:1:' \
    'FAIL SAFE-002: src/config-bad.ts:2:' \
    'FAIL SAFE-005: src/cache-bad.ts:2:' \
    'FAIL SAFE-005: src/session-bad.ts:1:' \
    'FAIL SAFE-006: src/auth-bad.ts:2:' \
    'FAIL AUTH-001: src/identity-bad.ts:1:' \
    'FAIL SAFE-007: app.json:1:'; do
    printf '%s\n' "$sw" | grep -qF "$expect" \
      || { echo "FAIL V2.70: the sweep did not report: $expect"; exit 1; }
  done
  for ok in src/ok-path.ts src/config-ok.ts src/cache-ok.ts src/session-ok.ts src/auth-ok.ts; do
    printf '%s\n' "$sw" | grep -E '^FAIL (SAFE-00[2456]|AUTH-001):' | grep -qF "$ok:" \
      && { echo "FAIL V2.70: the sweep reports the compliant twin $ok"; exit 1; }
  done
  printf '%s\n' "$sw" | grep -q 'NOT VERIFIED M-57' \
    || { echo "FAIL V2.70: the sweep does not mark the downgrade check as NOT VERIFIED without a base"; exit 1; }

  # 5. The repaired fixture passes.
  printf '%s\n' 'export const productPath = (productId: string) => `/api/v1/products/${encodeURIComponent(productId)}`;' > src/bad-path.ts
  printf '%s\n' 'import { z } from "zod";' 'export const apiBaseUrlSchema = z.string().url().refine((value) => value.startsWith("https://"), "https only");' > src/config-bad.ts
  printf '%s\n' 'export const load = (cache: any, userId: string) => {' '  cache.writeCache(`${userId}:cart`, {});' '};' > src/cache-bad.ts
  printf '%s\n' 'export const signOut = async (userId: string) => {' '  await deleteToken();' '  await clearCacheForIdentity(userId);' '};' > src/session-bad.ts
  printf '%s\n' 'export const check = async (response: Response, signOut: () => Promise<void>) => {' '  if (response.status === 401) {' '    await signOut();' '    return null;' '  }' '  return response.json();' '};' > src/auth-bad.ts
  printf '%s\n' 'export const userOf = (session: { userId: string }) => session.userId;' > src/identity-bad.ts
  printf '%s\n' '{ "expo": { "android": { "allowBackup": false } } }' > app.json
  printf '%s\n' '{' '  "dependencies": {' '    "expo": "^55.0.0"' '  }' '}' > package.json
  r="$(project_safety_sweep)"
  printf '%s\n' "$r" | grep -E '^FAIL (SAFE-00[24567]|AUTH-001):' | grep -E 'bad-path|config-bad|cache-bad|session-bad|auth-bad|identity-bad|app.json' \
    && { echo "FAIL V2.70: the repaired fixture is still reported"; exit 1; }
  g="$(dependency_downgrade_check . HEAD package.json)"
  [ -z "$g" ] || { echo "FAIL V2.70: a major upgrade is reported as a downgrade: $g"; exit 1; }
  exit 0
) || failures=1

echo "V2.70 probe: fixture in $fx was checked and is removed"
rm -rf "$fx"

exit "$failures"
```

## V2.71 Regression — Cross-Origin, Development Tools, Owner-Only Writes, and Absence Claims

Verify that the rules added after the fourth external audit are canonical,
reachable, and effective, and that the detectors and the review procedure
close the gaps that let the audit's findings through:

- M-62: a wildcard origin is never combined with credentials, the allowed
  origins are an explicit list, and a development tool is enabled only in a
  development build. Its signals are checked by `SAFE-008`.
- M-63: a file that holds personal or sensitive data is written with an
  owner-only file mode, and an uploaded image is stored with its metadata
  stripped. Its signals are checked by `SAFE-009`.
- `SAFE-001` reads each external call to its closing parenthesis, so an options
  object of any length is inspected, and a call that is not closed is
  `NOT VERIFIED`, never clean.
- `AUTH-001` reports a client-controlled identity sent in a quoted header name or
  a connection parameter, such as a WebSocket `connectionParams` entry.
- The security review verifies an absence claim on the complete path and marks an
  unverified absence `needs_validation` with no severity.

The text checks run on the repository. The behavioral part extracts the detector
blocks from `workflows/output-compliance.md`, builds a fixture git repository
outside the repository, verifies each violation is detected with `file:line` and
each compliant twin is not, and removes the fixture. It needs `awk`, `perl`, `git`,
`grep`, and `sed`; without them the behavioral part is `NOT VERIFIED`, and the text
checks still run.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
pv="workflows/project-validation.md"
ir="skills/implementation-recipes/SKILL.md"
sr="skills/security-review/SKILL.md"
ap="references/audit-pass-criteria.md"
sa="workflows/system-audit.md"
vs="workflows/system-validation.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.71: $1 is missing $3"; failures=1; }
}

# M-62 and M-63 canonical wording and reachability on every required surface.
for surface in "$ag" "$ir" "$sr" "$oc" "$pv" "$ap" "$rmx" "$vs"; do
  need "$surface" "a wildcard origin is never combined with credentials" "the M-62 anchor"
  need "$surface" "written with an owner-only file mode" "the M-63 anchor"
done
for id in M-62 M-63; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.71: no $id row"; failures=1; }
done
for id in R-108 R-109 R-110; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.71: no $id row"; failures=1; }
done
need "$oc" "### SAFE-008, SAFE-009" "the SAFE-008 and SAFE-009 section"
need "$oc" "cors_debug_check()" "the SAFE-008 detector"
need "$oc" "upload_write_check()" "the SAFE-009 detector"
need "$oc" 'cors_debug_check . $src' "the SAFE-008 call in the whole-project sweep"
need "$oc" 'upload_write_check . $src' "the SAFE-009 call in the whole-project sweep"
need "$oc" "up to 40 lines" "the call-extent timeout rule"
need "$sr" "absence claims" "the absence-claim step of the review"
need "$sr" "complete path" "the complete-path verification of an absence"
need "$pv" "SAFE-008" "the probe reporting the CORS detector"
need "$pv" "SAFE-009" "the probe reporting the upload detector"
need "$vs" "M-62: a wildcard origin" "the V2.71 scope statement"

command -v awk >/dev/null 2>&1 && command -v perl >/dev/null 2>&1 && command -v git >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.71: awk, perl, or git is not available, the behavioral fixture was not run"; exit 2; }

# Extracts the first sh block of a section of workflows/output-compliance.md.
extract() {
  awk -v start="$1" '
    $0 ~ "^### " start { on = 1; next }
    on && /^### / { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; exit }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v271.XXXXXX")"
(
  cd "$fx" || exit 1
  git init -q .
  git config user.email "fixture@example.invalid"
  git config user.name "fixture"
  mkdir -p src
  extract "SAFE-001, SAFE-002, SAFE-003" > safe.sh
  extract "SAFE-008, SAFE-009" > new.sh
  extract "SAFE-004 to SAFE-007" > sec.sh
  [ -s safe.sh ] && [ -s new.sh ] && [ -s sec.sh ] || {
    echo "FAIL V2.71: a detector block was not extracted"; exit 1; }
  . ./safe.sh
  . ./new.sh
  . ./sec.sh

  # Violations, each with its compliant twin where one exists.
  printf '%s\n' 'export const load = async (url: string) => {' '  const r = await fetch(url);' '  return r;' '};' > src/timeout-bad.ts
  printf '%s\n' 'export const load = async (url: string) => {' '  const r = await fetch(url, {' '    method: "GET",' '    headers: { accept: "application/json" },' '    body: undefined,' '    cache: "no-store",' '    redirect: "manual",' '    credentials: "omit",' '    signal: AbortSignal.timeout(5000),' '  });' '  return r;' '};' > src/timeout-long-ok.ts
  {
    printf '%s\n' 'export const load = async (url: string) => {' '  const r = await fetch(url, {'
    i=0; while [ "$i" -lt 45 ]; do printf '    a%s: %s,\n' "$i" "$i"; i=$((i + 1)); done
    printf '%s\n' '  });' '  return r;' '};'
  } > src/timeout-unclosed.ts
  printf '%s\n' 'export const cors = { origin: "*", credentials: true };' > src/cors-bad.ts
  printf '%s\n' 'export const cors = { origin: ["https://app.example.com"], credentials: true };' > src/cors-ok.ts
  printf '%s\n' 'export const gql = { graphiql: true };' > src/gql-bad.ts
  printf '%s\n' 'export const gql = { graphiql: process.env.NODE_ENV !== "production" };' > src/gql-ok.ts
  printf '%s\n' 'export const save = async (p: string, buf: Buffer) => {' '  await fs.writeFile(p, buf);' '};' > src/upload-bad.ts
  printf '%s\n' 'import sharp from "sharp";' 'export const save = async (p: string, buf: Buffer) => {' '  const out = await sharp(buf).rotate().toBuffer();' '  await fs.writeFile(p, out, {' '    mode: 0o600,' '  });' '};' > src/upload-ok.ts
  printf '%s\n' 'export const save = async (p: string, buf: Buffer) => {' '  await fs.writeFile(p, buf, { mode: 0o600 });' '};' > src/upload-nometa.ts
  printf '%s\n' 'const conn = { connectionParams: { "x-user-id": userId } };' > src/ws-bad.ts
  printf '%s\n' 'export const load = (session: { userId: string }) => session.userId;' > src/auth-ok.ts

  t="$(safe_timeout_check . src/timeout-bad.ts src/timeout-long-ok.ts src/timeout-unclosed.ts)"
  printf '%s\n' "$t" | grep -qx 'FAIL SAFE-001: src/timeout-bad.ts:2: external call with no timeout' \
    || { echo "FAIL V2.71: SAFE-001 did not detect src/timeout-bad.ts:2: $t"; exit 1; }
  printf '%s\n' "$t" | grep 'timeout-long-ok' > /dev/null \
    && { echo "FAIL V2.71: SAFE-001 reports the compliant long call: $(printf '%s\n' "$t" | grep 'timeout-long-ok')"; exit 1; }
  printf '%s\n' "$t" | grep -qx 'NOT VERIFIED SAFE-001: src/timeout-unclosed.ts:2: the call is not closed within 40 lines, so its options are not read' \
    || { echo "FAIL V2.71: SAFE-001 did not mark the unclosed call NOT VERIFIED: $t"; exit 1; }

  c="$(cors_debug_check . src/cors-bad.ts src/cors-ok.ts src/gql-bad.ts src/gql-ok.ts)"
  printf '%s\n' "$c" | grep -q '^FAIL SAFE-008: src/cors-bad.ts:1:' \
    || { echo "FAIL V2.71: SAFE-008 did not detect the wildcard with credentials: $c"; exit 1; }
  printf '%s\n' "$c" | grep -q '^FAIL SAFE-008: src/gql-bad.ts:1:' \
    || { echo "FAIL V2.71: SAFE-008 did not detect a development tool enabled by true: $c"; exit 1; }
  printf '%s\n' "$c" | grep -qE 'src/(cors-ok|gql-ok)\.ts' \
    && { echo "FAIL V2.71: SAFE-008 reports a compliant twin: $c"; exit 1; }

  u="$(upload_write_check . src/upload-bad.ts src/upload-ok.ts src/upload-nometa.ts)"
  printf '%s\n' "$u" | grep -q '^FAIL SAFE-009: src/upload-bad.ts:2:' \
    || { echo "FAIL V2.71: SAFE-009 did not detect the default file mode: $u"; exit 1; }
  printf '%s\n' "$u" | grep -q '^NOT VERIFIED SAFE-009: src/upload-nometa.ts:' \
    || { echo "FAIL V2.71: SAFE-009 did not mark the missing metadata handling NOT VERIFIED: $u"; exit 1; }
  printf '%s\n' "$u" | grep -q 'upload-ok.ts' \
    && { echo "FAIL V2.71: SAFE-009 reports the compliant upload: $u"; exit 1; }
  printf '%s\n' "$u" | grep -q '^FAIL SAFE-009: src/upload-nometa.ts' \
    && { echo "FAIL V2.71: SAFE-009 reports the owner-only twin: $u"; exit 1; }

  a="$(identity_source_check . src/ws-bad.ts src/auth-ok.ts)"
  printf '%s\n' "$a" | grep -q '^FAIL AUTH-001: src/ws-bad.ts:1:' \
    || { echo "FAIL V2.71: AUTH-001 did not detect the connection parameter: $a"; exit 1; }
  printf '%s\n' "$a" | grep -q 'auth-ok' \
    && { echo "FAIL V2.71: AUTH-001 reports the compliant session: $a"; exit 1; }

  # The repaired fixture passes.
  printf '%s\n' 'export const load = async (url: string) => {' '  const r = await fetch(url, { signal: AbortSignal.timeout(5000) });' '  return r;' '};' > src/timeout-bad.ts
  printf '%s\n' 'export const cors = { origin: ["https://app.example.com"], credentials: true };' > src/cors-bad.ts
  printf '%s\n' 'export const gql = { graphiql: process.env.NODE_ENV !== "production" };' > src/gql-bad.ts
  printf '%s\n' 'import sharp from "sharp";' 'export const save = async (p: string, buf: Buffer) => {' '  const out = await sharp(buf).rotate().toBuffer();' '  await fs.writeFile(p, out, { mode: 0o600 });' '};' > src/upload-bad.ts
  printf '%s\n' 'export const load = (session: { userId: string }) => session.userId;' > src/ws-bad.ts
  r="$(safe_timeout_check . src/timeout-bad.ts)$(cors_debug_check . src/cors-bad.ts src/gql-bad.ts)$(upload_write_check . src/upload-bad.ts)$(identity_source_check . src/ws-bad.ts)"
  [ -z "$r" ] || { echo "FAIL V2.71: the repaired fixture is still reported: $r"; exit 1; }
  exit 0
) || failures=1

echo "V2.71 probe: fixture in $fx was checked and is removed"
rm -rf "$fx"

exit "$failures"
```

## V2.72 Regression — Visible Failures, Verified Claims, Test Consumers, and Entry Identity

Verify that the rules added after the second external audit are canonical,
reachable, and effective, and that the detector and the review steps close the
gaps that let the findings through:

- M-64: a failure handled by failing closed reaches the user as a visible state,
  and a terminal result releases what it blocked. Its wording is checked on
  every required surface; the mapping of states is a review step (R-111).
- M-65: every factual statement in a generated proposal, pull request, or report
  is derived from the evidence it cites and re-derived before it is reported.
  Its wording is checked on every required surface (R-112).
- M-49, amended: a test, a test-support helper, or a file under a test or support
  folder is not a production consumer. The DEAD-EXPORT-001 block is extracted
  from `workflows/output-compliance.md` and run on a fixture (R-113).
- M-52 and M-56, amended: an entry point with no verified identity is a finding,
  and a project that caches identity-owned data needs a sign-out path (R-114).
- The composition root exception, the unit of a documented limit, bounded
  caches, and listener release on identity change (R-115).
- The React Native live-region rule: `accessibilityLiveRegion` is Android-only.

The text checks run on the repository. The behavioral part extracts the
DEAD-EXPORT-001 block, builds a fixture outside the repository with a constant
exported through a barrel and imported only by a test, verifies it is reported
with `file:line`, verifies a production consumer is not, and removes the
fixture. It needs `awk`, `perl`, `grep`, and `sed`; without them the behavioral
part is `NOT VERIFIED`, and the text checks still run.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
ir="skills/implementation-recipes/SKILL.md"
er="skills/engineering-review/SKILL.md"
ap="references/audit-pass-criteria.md"
cm="workflows/commit.md"
dep="workflows/dependency-assessment.md"
pr="skills/pull-request/SKILL.md"
sr="skills/security-review/SKILL.md"
ax="skills/accessibility/SKILL.md"
vs="workflows/system-validation.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.72: $1 is missing $3"; failures=1; }
}

# M-64 and M-65 canonical wording and reachability on every required surface.
for surface in "$ag" "$ir" "$er" "$ap" "$rmx" "$mr" "$vs"; do
  need "$surface" "reaches the user as a visible state" "the M-64 anchor"
done
for surface in "$ag" "$cm" "$dep" "$pr" "$ap" "$rmx" "$mr" "$vs"; do
  need "$surface" "is derived from the evidence it cites" "the M-65 anchor"
done
for id in M-64 M-65; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.72: no $id row"; failures=1; }
done
for id in R-111 R-112 R-113 R-114 R-115; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.72: no $id row"; failures=1; }
done

# Amended canonical wording of M-49, M-52, M-56 and the composition root.
need "$ag" "a test, a test-support helper, or a file under a test or support folder" "the M-49 test-consumer wording"
need "$ag" "receives no verified identity at all" "the M-52 absent-identity wording"
need "$ag" "has no sign-out path that clears those partitions" "the M-56 sign-out wording"
need "$ag" "only exception to the dependency rule" "the composition root exception"
need "$oc" "test files do not count as" "the DEAD-EXPORT-001 consumer rule"
need "$oc" "*.test.*|*.spec.*" "the test-path exclusion in the DEAD-EXPORT-001 block"
need "$ir" "do not memoize a rejected promise" "the rejected-promise recipe"
need "$ir" "bytes versus utf-16" "the documented-unit recipe"
need "$ir" "composition stays in the composition root" "the composition recipe"
need "$er" "claims in comments and documents" "the comment-accuracy review step"
need "$ax" "accessibilityLiveRegion" "the React Native live-region note"
need "$ax" "android only" "the Android-only statement of the live region"
need "$ap" "a concrete adapter is constructed only in the composition root" "the composition criterion"
need "$cm" "from the evidence it cites" "the commit proposal claim rule"

command -v awk >/dev/null 2>&1 && command -v perl >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.72: awk or perl is not available, the behavioral fixture was not run"; exit 2; }

# Extracts the first sh block of a section of workflows/output-compliance.md.
extract() {
  awk -v start="$1" '
    $0 ~ "^### " start { on = 1; next }
    on && /^### / { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; exit }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v272.XXXXXX")"
(
  cd "$fx" || exit 1
  mkdir -p src/features/checkout
  extract "DEAD-EXPORT-001" > dead.sh
  [ -s dead.sh ] || { echo "FAIL V2.72: the DEAD-EXPORT-001 block was not extracted"; exit 1; }
  . ./dead.sh

  printf '%s\n' 'export { createInMemory } from "./storage";' > src/features/checkout/index.ts
  printf '%s\n' 'export const createInMemory = () => ({});' 'export const usedInProd = 1;' > src/features/checkout/storage.ts
  printf '%s\n' 'import { createInMemory } from "./index";' 'import { it } from "vitest";' > src/features/checkout/storage.test.ts
  printf '%s\n' 'import { usedInProd } from "./features/checkout/storage";' 'export const app = usedInProd;' > src/app.ts

  out="$(dead_export_check . src/features/checkout/storage.ts)"
  printf '%s\n' "$out" | grep -q '^FAIL DEAD-EXPORT-001: src/features/checkout/storage.ts:1: createInMemory' \
    || { echo "FAIL V2.72: a test-only export is not reported with file:line: $out"; exit 1; }
  printf '%s\n' "$out" | grep -q 'usedInProd' \
    && { echo "FAIL V2.72: a production consumer is reported as dead: $out"; exit 1; }

  # The repaired fixture: the test-only helper moves to the support location.
  mkdir -p src/testing
  printf '%s\n' 'export const createInMemory = () => ({});' > src/testing/in-memory.ts
  printf '%s\n' 'import { createInMemory } from "../testing/in-memory";' 'import { it } from "vitest";' > src/features/checkout/storage.test.ts
  printf '%s\n' 'export const usedInProd = 1;' > src/features/checkout/storage.ts
  printf '%s\n' 'export { createInMemory } from "./storage";' > src/features/checkout/index.ts
  out="$(dead_export_check . src/testing/in-memory.ts src/features/checkout/storage.ts)"
  [ -z "$out" ] || { echo "FAIL V2.72: the repaired fixture is still reported: $out"; exit 1; }
  exit 0
) || failures=1

echo "V2.72 probe: fixture in $fx was checked and is removed"
rm -rf "$fx"

exit "$failures"
```

## V2.73 Regression — Direction, Locale, Consumed Code, Claims, Records, and Platform Gaps

Verify that the rules added after the external audit of an internationalized
React Native change are canonical, reachable, and effective, and that the
detectors close the gaps that let its findings through:

- M-66: a direction-dependent unit resolves its direction from the active
  direction, has an RTL assertion of its own, and no formatter takes a default
  locale (R-116, `I18N-001` to `I18N-004`).
- M-67: a changed unit that consumes a pre-existing violating unit is part of the
  change set, and the task is `BLOCKED` until the consumed unit is corrected (R-117);
  `CONSUME-001` resolves the relative imports of each changed file and reports the
  inherited defect with both lines.
- M-68: a universal or exclusive claim, or a cited artifact, is enumerated before
  it is kept (R-118, `CLAIM-001`).
- M-50 and M-39, amended: a decision record names its patterns and the five SOLID
  rows, and a `NOT VERIFIED` on an applicable dimension blocks delivery (R-119,
  `DEC-001`). The live-region platform gap is reported as `A11Y-001` (R-119).

The text checks run on the repository. The behavioral part extracts the detector
detector blocks from `workflows/output-compliance.md`, builds a fixture outside the
repository that reproduces each finding of the external audit, verifies each
defect is reported with `file:line`, rebuilds the fixture in its repaired form, and
verifies it is clean. It needs `awk`, `grep`, `sed`, and `find`; without them the
behavioral part is `NOT VERIFIED`, and the text checks still run.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
ap="references/audit-pass-criteria.md"
ir="skills/implementation-recipes/SKILL.md"
ii="skills/internationalization/SKILL.md"
er="skills/engineering-review/SKILL.md"
tc="workflows/task-to-code.md"
ca="workflows/code-audit.md"
vs="workflows/system-validation.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.73: $1 is missing $3"; failures=1; }
}

# Canonical wording and reachability of M-66, M-67, and M-68 on every required surface.
for surface in "$ag" "$ii" "$ir" "$ap" "$oc" "$rmx" "$mr" "$vs"; do
  need "$surface" "resolves its layout direction from the active direction" "the M-66 anchor"
done
for surface in "$ag" "$ap" "$tc" "$ca" "$er" "$rmx" "$mr" "$vs"; do
  need "$surface" "consumes a pre-existing unit which violates a requirement" "the M-67 anchor"
done
for surface in "$ag" "$ap" "$er" "$oc" "$rmx" "$mr" "$vs"; do
  need "$surface" "enumerates every element of the scope" "the M-68 anchor"
done

# Amended canonical wording of M-39 and M-50.
need "$ag" "a \`NOT VERIFIED\` verdict on an applicable dimension blocks delivery" "the M-39 blocking wording"
need "$ag" "names every selected pattern with its force" "the M-50 record wording"
need "$oc" "its record is in the change set" "the M-50 anchor on the record check"
need "$ap" "a \`NOT VERIFIED\` verdict on an applicable dimension blocks delivery" "the M-39 wording in the criteria"

for id in M-66 M-67 M-68; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.73: no $id row"; failures=1; }
done
for id in R-116 R-117 R-118 R-119; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.73: no $id row"; failures=1; }
done

# Every detector is defined, and the health probe sweep runs each one.
for det in I18N-001 I18N-002 I18N-003 I18N-004 CONSUME-001 DEC-001 CLAIM-001 A11Y-001 A11Y-002 DIP-002; do
  grep -q "^### $det " "$root/$oc" || { echo "FAIL V2.73: $oc has no $det section"; failures=1; }
done
for call in i18n_default_locale_check i18n_direction_check i18n_rtl_test_check i18n_copy_check \
  a11y_live_region_check a11y_android_path_check platform_side_effect_check decision_record_check claim_check; do
  grep -q "^  $call \. " "$root/$oc" || { echo "FAIL V2.73: the sweep does not run $call"; failures=1; }
done

# The criteria name the detectors, so an auditor finds them from the matrix.
for det in I18N-001 I18N-003 DEC-001 CLAIM-001 A11Y-001 A11Y-002 DIP-002; do
  grep -q "$det" "$root/$ap" || { echo "FAIL V2.73: the criteria do not name $det"; failures=1; }
done

command -v awk >/dev/null 2>&1 && command -v grep >/dev/null 2>&1 && command -v find >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.73: awk, grep, or find is not available, the behavioral fixture was not run"; exit 2; }

# Extracts the sh blocks of the detector sections of workflows/output-compliance.md.
extract() {
  awk '
    /^### (I18N-00[1-4]|CONSUME-001|DEC-001|CLAIM-001|A11Y-001|A11Y-002|DIP-002) / { on = 1; next }
    /^### / { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; next }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v273.XXXXXX")"
(
  cd "$fx" || exit 1
  mkdir -p src/features/payment/hooks/use-open-settings \
    src/app src/shared/utils/format-currency src/features/payment/components/checkbox-row \
    src/features/payment/components/payment-screen src/features/notifications/services/demo-push docs/decisions
  extract > detectors.sh
  [ -s detectors.sh ] || { echo "FAIL V2.73: the detector blocks were not extracted"; exit 1; }
  . ./detectors.sh

  # The defective fixture: the findings of the external audit, in their form.
  printf '%s\n' "import { I18nManager } from 'react-native';" "export const boot = () => I18nManager.allowRTL(false);" > src/app/app.tsx
  printf '%s\n' "export const formatCurrency = (value: number, currency: string, locale = 'en-US') =>" "  new Intl.NumberFormat(locale, { style: 'currency', currency }).format(value);" "export const shortDate = (d: Date) => d.toLocaleDateString();" > src/shared/utils/format-currency/format-currency.ts
  printf '%s\n' "import { View } from 'react-native';" "export const CheckboxRow = () => <View style={{ flexDirection: 'row' }} />;" > src/features/payment/components/checkbox-row/checkbox-row.tsx
  printf '%s\n' "import { test } from 'vitest';" "test('renders', () => {});" > src/features/payment/components/checkbox-row/checkbox-row.test.tsx
  printf '%s\n' "import { CheckboxRow } from '../checkbox-row/checkbox-row';" "export const PaymentScreen = () => <CheckboxRow />;" > src/features/payment/components/payment-screen/payment-screen.tsx
  printf '%s\n' "import { test } from 'vitest';" "test('renders in es', () => {});" > src/features/payment/components/payment-screen/payment-screen.test.tsx
  printf '%s\n' "export const message = (amount: string) => \`We received \${amount} for order 42.\`;" > src/features/notifications/services/demo-push/demo-push.ts
  printf '%s\n' "# MB-09" "Forces and decisions only. Five patterns were considered." > docs/decisions/mb-09.md
  printf '%s\n' "# App" "Cada unidad conserva sus aserciones RTL. Capturas en simulador iOS 16.2." > README.md
  printf '%s\n' "export const Live = () => <View accessibilityLiveRegion=\"polite\" />;" > src/features/notifications/services/live.tsx
  printf '%s\n' "import { Linking } from 'react-native';" "export const useOpenSettings = () => () => Linking.openSettings();" > src/features/payment/hooks/use-open-settings/use-open-settings.ts

  defects="$( {
    i18n_default_locale_check . src/shared/utils/format-currency/format-currency.ts
    i18n_direction_check . src/features/payment/components/checkbox-row/checkbox-row.tsx
    i18n_rtl_test_check . src/features/payment/components/checkbox-row/checkbox-row.tsx src/features/payment/components/payment-screen/payment-screen.tsx
    i18n_copy_check . src/features/notifications/services/demo-push/demo-push.ts
    consumed_defect_check . src/features/payment/components/payment-screen/payment-screen.tsx
    decision_record_check . docs/decisions/mb-09.md
    claim_check . README.md
    a11y_live_region_check . src/features/notifications/services/live.tsx
    platform_side_effect_check . src/features/payment/hooks/use-open-settings/use-open-settings.ts
  } )"
  for want in \
    'FAIL I18N-001: src/shared/utils/format-currency/format-currency.ts:1:' \
    'FAIL I18N-001: src/shared/utils/format-currency/format-currency.ts:3:' \
    'FAIL I18N-002: src/features/payment/components/checkbox-row/checkbox-row.tsx:2:' \
    'FAIL I18N-003: src/features/payment/components/checkbox-row/checkbox-row.tsx:1:' \
    'NOT VERIFIED I18N-003: src/features/payment/components/payment-screen/payment-screen.tsx:1:' \
    'FAIL I18N-004: src/features/notifications/services/demo-push/demo-push.ts:1:' \
    'FAIL CONSUME-001: src/features/payment/components/payment-screen/payment-screen.tsx:1: imports ../checkbox-row/checkbox-row' \
    'FAIL DEC-001: docs/decisions/mb-09.md:1: no SOLID row for SRP' \
    'NOT VERIFIED CLAIM-001: README.md:2:' \
    'NOT VERIFIED A11Y-001: src/features/notifications/services/live.tsx:1:' \
    'FAIL DIP-002: src/features/payment/hooks/use-open-settings/use-open-settings.ts:1:'; do
    printf '%s\n' "$defects" | grep -qF -- "$want" \
      || { echo "FAIL V2.73: the defect was not reported with file:line: $want"; exit 1; }
  done

  # The A11Y-002 defect: an announcement with no Android path. It is checked alone, because
  # a live region elsewhere in the fixture would be an Android path.
  printf '%s\n' "export const announce = (m: string) => AccessibilityInfo.announceForAccessibility(m);" > src/features/notifications/services/announce.tsx
  a11y_defect="$(a11y_android_path_check . src/features/notifications/services/announce.tsx)"
  printf '%s\n' "$a11y_defect" | grep -qF 'NOT VERIFIED A11Y-002: src/features/notifications/services/announce.tsx:1:' \
    || { echo "FAIL V2.73: A11Y-002 did not report the iOS-only announcement with file:line"; exit 1; }

  # The repaired twin: the same files in their compliant form.
  printf '%s\n' "export const formatCurrency = (value: number, currency: string, locale: string) =>" "  new Intl.NumberFormat(locale, { style: 'currency', currency }).format(value);" "export const shortDate = (d: Date, locale: string) => d.toLocaleDateString(locale);" > src/shared/utils/format-currency/format-currency.ts
  printf '%s\n' "import { I18nManager } from 'react-native';" "export const CheckboxRow = () => <View style={{ flexDirection: I18nManager.isRTL ? 'row-reverse' : 'row' }} />;" > src/features/payment/components/checkbox-row/checkbox-row.tsx
  printf '%s\n' "import { test } from 'vitest';" "test('renders under ar', () => { expect('ar').toBeDefined(); });" > src/features/payment/components/checkbox-row/checkbox-row.test.tsx
  printf '%s\n' "import { test } from 'vitest';" "test('renders under ar', () => { expect('ar').toBeDefined(); });" > src/features/payment/components/payment-screen/payment-screen.test.tsx
  printf '%s\n' "export const message = (t: (key: string) => string, amount: string) => t('notifications.received').replace('{amount}', amount);" > src/features/notifications/services/demo-push/demo-push.ts
  printf '%s\n' "# MB-09" "| Pattern | Force | Signal | Consumers (\`file:line\`) | Stack translation |" "|---|---|---|---|---|" "| Adapter | Locale source | BE1 | \`locale.adapter.ts:8\`, \`format.ts:3\` | expo-localization adapter |" "Count: 1 selected" "| Principle | Status | Unit | Evidence |" "|---|---|---|---|" "| SRP | APPLIES | locale adapter | \`locale.adapter.ts:8\` reads the locale only |" "| OCP | PRESERVED | locale adapter | no edit |" "| LSP | NOT APPLICABLE | no substitution | none |" "| ISP | PRESERVED | locale port | small |" "| DIP | APPLIES | composition root | adapter behind port |" > docs/decisions/mb-09.md
  printf '%s\n' "# App" "Las pruebas RTL de checkbox-row y payment-screen usan el locale ar." > README.md
  printf '%s\n' "export const announce = (m: string) => AccessibilityInfo.announceForAccessibility(m);" "export const Status = () => <Text accessibilityLiveRegion=\"polite\" />;" > src/features/notifications/services/announce.tsx

  printf '%s\n' "import type { SettingsOpener } from '../../types/settings-opener';" "export const useOpenSettings = (opener: SettingsOpener) => () => opener.open();" > src/features/payment/hooks/use-open-settings/use-open-settings.ts

  repaired="$( {
    a11y_android_path_check . src/features/notifications/services/announce.tsx
    i18n_default_locale_check . src/shared/utils/format-currency/format-currency.ts
    i18n_direction_check . src/features/payment/components/checkbox-row/checkbox-row.tsx
    i18n_rtl_test_check . src/features/payment/components/checkbox-row/checkbox-row.tsx src/features/payment/components/payment-screen/payment-screen.tsx
    i18n_copy_check . src/features/notifications/services/demo-push/demo-push.ts
    consumed_defect_check . src/features/payment/components/payment-screen/payment-screen.tsx
    decision_record_check . docs/decisions/mb-09.md
    claim_check . README.md
    a11y_live_region_check . src/features/notifications/services/live.tsx
    platform_side_effect_check . src/features/payment/hooks/use-open-settings/use-open-settings.ts
  } )"
  [ -z "$repaired" ] || { echo "FAIL V2.73: the repaired fixture is still reported: $repaired"; exit 1; }
  exit 0
) || failures=1

echo "V2.73 probe: fixture in $fx was checked and is removed"
rm -rf "$fx"

exit "$failures"
```

## V2.74 Regression — Deploy Order, Application-Side Schema Ownership, and Scope Manifest

Verify that the rules added after the external audit of a SQLite-backed change
are canonical, reachable, and effective, and that the detectors close the gaps
that let its findings through:

- M-69: a database change is compatible in either deployment order, and a claim
  of order-safety is kept only with a test that executes each order (R-120).
  `DB-002` reports a migration with no deploy-order test, and `CLAIM-001`
  reports an order-safety claim as `NOT VERIFIED` with its line. The external
  audit found the claim "order-safe in both directions" false, and the migration
  with no test that executes the new release on the unmigrated database.
- M-70: application code never changes the schema; `CREATE`, `ALTER`, and `DROP`
  run only from the migration tooling, including `CREATE ... IF NOT EXISTS` (R-121).
  `DB-001` reports a store that creates its unique index at open with `file:line`,
  and passes the same statement in a migration folder. The external audit found
  this statement at store startup, which crashed the new release on an unmigrated
  database.
- M-71: every file in a change set is named in its approved scope manifest, and a
  formatter rewrite of an unnamed file is reverted for that file (R-122).
  `SCOPE-001` reports the unrelated reformatted file. The external audit found it
  as a scope change.

The text checks run on the repository. The behavioral part extracts the detector
blocks of `workflows/output-compliance.md`, builds a git fixture outside the
repository that reproduces each external finding, verifies each defect is
reported with `file:line`, rebuilds the fixture in its repaired form, and
verifies it is clean of `FAIL`. It needs `awk`, `grep`, `sed`, `find`, `mktemp`,
and `git`; without them the behavioral part is `NOT VERIFIED`, and the text
checks still run.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
ap="references/audit-pass-criteria.md"
ir="skills/implementation-recipes/SKILL.md"
db="skills/database-engineering/SKILL.md"
tc="workflows/task-to-code.md"
cm="workflows/commit.md"
pv="workflows/project-validation.md"
vs="workflows/system-validation.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.74: $1 is missing $3"; failures=1; }
}

# Canonical wording and reachability of M-69, M-70, and M-71 on every required surface.
for surface in "$ag" "$db" "$ir" "$ap" "$oc" "$tc" "$rmx" "$mr" "$vs"; do
  need "$surface" "compatible in either deployment order" "the M-69 anchor"
done
for surface in "$ag" "$db" "$ir" "$ap" "$oc" "$tc" "$rmx" "$mr" "$vs"; do
  need "$surface" "application code never changes the schema" "the M-70 anchor"
done
for surface in "$ag" "$cm" "$pv" "$ap" "$rmx" "$mr" "$vs"; do
  need "$surface" "named in its approved scope manifest" "the M-71 anchor"
done

for id in M-69 M-70 M-71; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.74: no $id row"; failures=1; }
done
for id in R-120 R-121 R-122; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.74: no $id row"; failures=1; }
done

# Every detector is defined, the health probe sweep runs the two project-wide ones,
# and the criteria name them so an auditor finds them from the matrix.
for det in DB-001 DB-002 SCOPE-001; do
  grep -q "^### $det " "$root/$oc" || { echo "FAIL V2.74: $oc has no $det section"; failures=1; }
done
for call in db_runtime_ddl_check db_deploy_order_check; do
  grep -q "^  $call \. " "$root/$oc" || { echo "FAIL V2.74: the sweep does not run $call"; failures=1; }
done
grep -q "compatibility claim: keep it only with a test that executes each deployment order" "$root/$oc" \
  || { echo "FAIL V2.74: CLAIM-001 does not flag order-safety claims"; failures=1; }
for det in DB-001 DB-002 SCOPE-001; do
  grep -q "$det" "$root/$ap" || { echo "FAIL V2.74: the criteria do not name $det"; failures=1; }
done

command -v awk >/dev/null 2>&1 && command -v grep >/dev/null 2>&1 && command -v find >/dev/null 2>&1 \
  && command -v mktemp >/dev/null 2>&1 && command -v git >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.74: awk, grep, find, mktemp, or git is not available, the behavioral fixture was not run"; exit 2; }

# Extracts the sh blocks of the detector sections of workflows/output-compliance.md.
extract() {
  awk '
    /^### (DB-001|DB-002|SCOPE-001|CLAIM-001) / { on = 1; next }
    /^### / { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; next }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v274.XXXXXX")"
(
  cd "$fx" || exit 1
  mkdir -p src/adapters migrations src/config src/features
  extract > detectors.sh
  [ -s detectors.sh ] || { echo "FAIL V2.74: the detector blocks were not extracted"; exit 1; }
  . ./detectors.sh

  # The defective fixture: the external audit's store, its migration, its test, and its claim.
  printf '%s\n' "export const openStore = (db: Db) => {" \
    "  db.exec(\"CREATE UNIQUE INDEX IF NOT EXISTS idx_default_method ON auth_users(default_method_id)\");" \
    "};" > src/adapters/sqlite-auth-store.ts
  printf '%s\n' "export const up = (db: Db) => db.exec(\"CREATE UNIQUE INDEX IF NOT EXISTS idx_default_method ON auth_users(default_method_id)\");" \
    > migrations/003-default-method.ts
  printf '%s\n' "import { test } from 'vitest';" "test('opens the store', () => {});" > src/adapters/sqlite-auth-store.test.ts
  printf '%s\n' "// Expand is order-safe in both directions." "export const expand = () => 1;" > src/adapters/expand.ts

  defects="$( {
    db_runtime_ddl_check . src/adapters/sqlite-auth-store.ts migrations/003-default-method.ts src/adapters/sqlite-auth-store.test.ts
    db_deploy_order_check . migrations/003-default-method.ts -- src/adapters/sqlite-auth-store.test.ts
    claim_check . src/adapters/expand.ts
  } )"
  for want in \
    'FAIL DB-001: src/adapters/sqlite-auth-store.ts:2:' \
    'FAIL DB-002: migrations/003-default-method.ts:1:' \
    'NOT VERIFIED CLAIM-001: src/adapters/expand.ts:1:'; do
    printf '%s\n' "$defects" | grep -qF -- "$want" \
      || { echo "FAIL V2.74: the defect was not reported with file:line: $want"; exit 1; }
  done
  # Negative control: the same statement in a migration folder is not runtime DDL.
  printf '%s\n' "$defects" | grep -q 'FAIL DB-001: migrations/' \
    && { echo "FAIL V2.74: DB-001 reports the migration that owns the statement"; exit 1; }

  # The repaired twin: the store has no DDL, a named legacy-schema test exists, and the claim is reworded.
  printf '%s\n' "export const openStore = (db: Db) => db.prepare('SELECT 1').get();" > src/adapters/sqlite-auth-store.ts
  printf '%s\n' "import { test } from 'vitest';" "test('new code on a legacy database without the migration fails closed', () => {});" \
    > src/adapters/sqlite-auth-store.test.ts
  printf '%s\n' "// Expand adds a nullable column; the legacy-schema test covers the other order." "export const expand = () => 1;" > src/adapters/expand.ts

  repaired="$( {
    db_runtime_ddl_check . src/adapters/sqlite-auth-store.ts migrations/003-default-method.ts src/adapters/sqlite-auth-store.test.ts
    db_deploy_order_check . migrations/003-default-method.ts -- src/adapters/sqlite-auth-store.test.ts
    claim_check . src/adapters/expand.ts
  } )"
  printf '%s\n' "$repaired" | grep -q '^FAIL' \
    && { echo "FAIL V2.74: the repaired fixture still reports a failure: $repaired"; exit 1; }
  printf '%s\n' "$repaired" | grep -qF 'NOT VERIFIED DB-002: migrations/003-default-method.ts:1:' \
    || { echo "FAIL V2.74: a named deploy-order test is not reported as NOT VERIFIED"; exit 1; }

  # SCOPE-001 on a git fixture: an unrelated reformatted file is reported, the approved file is not. The manifest lives outside the repository (M-90), so it is never an untracked change.
  git init -q . || { echo "FAIL V2.74: git init failed in the fixture"; exit 1; }
  printf 'a\n' > src/config/job-env.ts
  printf 'b\n' > src/features/payment.ts
  git add -A && git -c user.email=probe@local -c user.name=probe commit -qm base || exit 1
  printf 'a2\n' > src/config/job-env.ts
  printf 'b2\n' > src/features/payment.ts
  printf '%s\n' src/features/payment.ts > "${fx}.approved"
  scope="$(scope_file_check . "${fx}.approved")"
  printf '%s\n' "$scope" | grep -qF 'FAIL SCOPE-001: src/config/job-env.ts:1:' \
    || { echo "FAIL V2.74: SCOPE-001 did not report the unnamed file: $scope"; exit 1; }
  printf '%s\n' "$scope" | grep -q 'src/features/payment.ts' \
    && { echo "FAIL V2.74: SCOPE-001 reports the file the manifest names"; exit 1; }
  printf '%s\n' src/features/payment.ts src/config/job-env.ts > "${fx}.approved"
  [ -z "$(scope_file_check . "${fx}.approved")" ] \
    || { echo "FAIL V2.74: SCOPE-001 reports a file after the manifest names it"; exit 1; }
  exit 0
) || failures=1

echo "V2.74 probe: fixture in $fx was checked and is removed"
rm -rf "$fx" "${fx}.approved"

exit "$failures"
```

## V2.75 Regression — Inventory Enumeration, Repository State, and Unexecuted Steps

Verify that the rules added after the external audit of a destructive-operation
analysis are canonical, reachable, and effective, and that the detectors close the
gaps that let its findings through:

- M-72: an inventory in a report is produced by an enumeration of the target in the
  same session, cited with its command and count, and every listed name is returned by
  it (R-123). `INV-001` reports a claimed name the enumeration does not return and an
  enumerated item the claim neither lists nor excludes. The external audit found three
  claimed database files absent on disk and three existing ones omitted.
- M-73: a read-only gate records the state of the target repository before and after
  its run, and a state change is an unattributed anomaly, not a claim of authorship
  (R-124). `STATE-001` reports the change with its reflog entry as `NOT VERIFIED`. The
  external audit found a commit in the target during its gate, and a README that named
  its author without evidence.
- M-74: a required step the agent does not execute is `NOT DONE` with its reason and is
  put to the user, and an end state reached by a side effect is not completion (R-125).
  The external audit found a teardown omitted and the clean tree attributed to a commit
  instead. Its enforcement is the flow and the text checks below, not a detector: this
  check verifies the wording and reachability, and the limit is stated in the report.

The text checks run on the repository. The behavioral part extracts the detector
blocks of `workflows/output-compliance.md`, builds git and inventory fixtures outside
the repository that reproduce the external findings, verifies each defect is reported
with `file:line`, verifies the compliant twin is clean, and removes the fixture. It
needs `awk`, `grep`, `sed`, `mktemp`, `cmp`, `cksum`, and `git`; without them the
behavioral part is `NOT VERIFIED`, and the text checks still run.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
ap="references/audit-pass-criteria.md"
db="skills/database-engineering/SKILL.md"
cg="workflows/code-audit.md"
sd="workflows/secure-delivery.md"
tc="workflows/task-to-code.md"
vs="workflows/system-validation.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.75: $1 is missing $3"; failures=1; }
}

# Canonical wording and reachability of M-72, M-73, and M-74 on every required surface.
for surface in "$ag" "$db" "$ap" "$oc" "$rmx" "$mr" "$vs"; do
  need "$surface" "produced by an enumeration of the target" "the M-72 anchor"
done
for surface in "$ag" "$cg" "$sd" "$ap" "$rmx" "$mr" "$vs"; do
  need "$surface" "records the state of the target repository" "the M-73 anchor"
done
for surface in "$ag" "$tc" "$ap" "$rmx" "$mr" "$vs"; do
  need "$surface" "a required step that the agent does not execute" "the M-74 anchor"
done

for id in M-72 M-73 M-74; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.75: no $id row"; failures=1; }
done
for id in R-123 R-124 R-125; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.75: no $id row"; failures=1; }
done

# Both detectors are defined, and the criteria name them so an auditor finds them.
for det in INV-001 STATE-001; do
  grep -q "^### $det " "$root/$oc" || { echo "FAIL V2.75: $oc has no $det section"; failures=1; }
  grep -q "$det" "$root/$ap" || { echo "FAIL V2.75: the criteria do not name $det"; failures=1; }
done

command -v awk >/dev/null 2>&1 && command -v grep >/dev/null 2>&1 && command -v sed >/dev/null 2>&1 \
  && command -v mktemp >/dev/null 2>&1 && command -v git >/dev/null 2>&1 \
  && command -v cmp >/dev/null 2>&1 && command -v cksum >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.75: awk, grep, sed, mktemp, cmp, cksum, or git is not available, the behavioral fixtures were not run"; exit 2; }

# Extracts the sh blocks of the INV-001 and STATE-001 sections of workflows/output-compliance.md.
extract() {
  awk '
    /^### (INV-001|STATE-001) / { on = 1; next }
    /^### / { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; next }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v275.XXXXXX")"
snap="$(mktemp -d "${TMPDIR:-/tmp}/v275snap.XXXXXX")"
(
  cd "$fx" || exit 1
  extract > detectors.sh
  [ -s detectors.sh ] || { echo "FAIL V2.75: the detector blocks were not extracted"; exit 1; }
  . ./detectors.sh

  # INV-001, defective: the claim names three files that are not on disk and omits three that are.
  printf '%s\n' orders.sqlite receipts.sqlite loyalty.sqlite payments.sqlite sessions.sqlite catalog.sqlite > enum.txt
  printf '%s\n' auth.sqlite imports.sqlite order-callbacks.sqlite payments.sqlite sessions.sqlite catalog.sqlite > claim.txt
  defects="$(inventory_check enum.txt claim.txt)"
  for want in \
    'FAIL INV-001: claim.txt:1: auth.sqlite is claimed' \
    'FAIL INV-001: claim.txt:2: imports.sqlite is claimed' \
    'FAIL INV-001: claim.txt:3: order-callbacks.sqlite is claimed' \
    'FAIL INV-001: enum.txt:1: orders.sqlite is enumerated' \
    'FAIL INV-001: enum.txt:2: receipts.sqlite is enumerated' \
    'FAIL INV-001: enum.txt:3: loyalty.sqlite is enumerated'; do
    printf '%s\n' "$defects" | grep -qF -- "$want" \
      || { echo "FAIL V2.75: the inventory defect was not reported with file:line: $want"; exit 1; }
  done
  # Negative control: a name both lists and the enumeration share is not reported.
  printf '%s\n' "$defects" | grep -q 'payments.sqlite' \
    && { echo "FAIL V2.75: INV-001 reports a name the enumeration and the claim share"; exit 1; }

  # INV-001, repaired: the claim equals the enumeration, and a dropped store is excluded with its reason.
  cp enum.txt claim-exact.txt
  [ -z "$(inventory_check enum.txt claim-exact.txt)" ] \
    || { echo "FAIL V2.75: INV-001 reports a claim that equals the enumeration"; exit 1; }
  printf '%s\n' orders.sqlite receipts.sqlite payments.sqlite sessions.sqlite catalog.sqlite '# excluded loyalty.sqlite test fixture, not production data' > claim-excluded.txt
  [ -z "$(inventory_check enum.txt claim-excluded.txt)" ] \
    || { echo "FAIL V2.75: INV-001 reports a store excluded with a reason"; exit 1; }

  # STATE-001: an unchanged snapshot is clean; a commit made during the gate is an unattributed anomaly.
  git init -q . || { echo "FAIL V2.75: git init failed in the fixture"; exit 1; }
  printf 'a\n' > f.txt
  git add -A && git -c user.email=probe@local -c user.name=probe commit -qm base || exit 1
  repo_state_snapshot . > "$snap/before.txt"
  state_check . "$snap/before.txt" > "$snap/clean.txt"
  [ -z "$(cat "$snap/clean.txt")" ] && state_check . "$snap/before.txt" >/dev/null \
    || { echo "FAIL V2.75: STATE-001 reports an unchanged state"; exit 1; }

  printf 'b\n' > f.txt
  git add -A && git -c user.email=probe@local -c user.name=probe commit -qm during-gate || exit 1
  anomaly="$(state_check . "$snap/before.txt")"
  printf '%s\n' "$anomaly" | grep -qF 'NOT VERIFIED STATE-001' \
    || { echo "FAIL V2.75: STATE-001 did not report the change made during the gate: $anomaly"; exit 1; }
  printf '%s\n' "$anomaly" | grep -q '^FAIL' \
    && { echo "FAIL V2.75: STATE-001 reports the unattributed change as FAIL instead of NOT VERIFIED"; exit 1; }
  printf '%s\n' "$anomaly" | grep -qF 'during-gate' \
    || { echo "FAIL V2.75: STATE-001 did not print the reflog entry of the change"; exit 1; }
  exit 0
) || failures=1

echo "V2.75 probe: fixtures in $fx and $snap were checked and are removed"
rm -rf "$fx" "$snap"

exit "$failures"
```

## V2.76 Regression — Performance Claims, Technical-Property Claims, and Sweep Coverage

Verify that the rule added after the external audit of a database-performance
finding is canonical, reachable, and effective, and that the detector reads the
claims the external audit found:

- M-65: a performance claim (a query, index, or path said to be faster or fixed)
  is kept only with a before-and-after measurement of the named query, and a
  technical property in a comment or document (a covering index, an index-only scan,
  a constant-time operation) is kept only with the plan or the code that decides it
  (R-126). The external audit marked a query fixed with a `PASS` and no measurement,
  and a source comment claimed a covering index that the query did not have.
- `CLAIM-001` reports both claim kinds as `NOT VERIFIED` with `file:line`, in documents
  and in source comments, and the whole-project sweep passes the source files to it
  (`claim_check . $mds $src`), so a comment in `.ts` is not skipped.
- The health probe lists `DB-001`, `DB-002`, and `CLAIM-001` among the sweep items, so a
  pre-existing schema statement in a store is reported by the probe.

The text checks run on the repository. The behavioral part extracts the `CLAIM-001`
block of `workflows/output-compliance.md`, builds a fixture outside the repository
with a `.ts` comment and a README that make the claims, verifies each is reported
with `file:line`, verifies a compliant twin is silent, and removes the fixture. It
needs `awk`, `grep`, `sed`, and `mktemp`; without them the behavioral part is
`NOT VERIFIED`, and the text checks still run.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
ap="references/audit-pass-criteria.md"
pv="workflows/project-validation.md"
db="skills/database-engineering/SKILL.md"
vs="workflows/system-validation.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.76: $1 is missing $3"; failures=1; }
}

# M-65 performance and technical-property wording on every required surface.
for surface in "$ag" "$mr" "$ap" "$db" "$rmx" "$vs"; do
  need "$surface" "before-and-after measurement of the named query" "the performance anchor (M-65)"
done
for surface in "$ag" "$mr" "$ap" "$rmx" "$vs"; do
  need "$surface" "covering index" "the technical-property anchor (M-65)"
done
grep -q "^| R-126 |" "$root/$rmx" || { echo "FAIL V2.76: no R-126 row"; failures=1; }

# The detector is defined, the sweep passes source files to it, and the probe lists the sweep items.
grep -q "^### CLAIM-001 " "$root/$oc" || { echo "FAIL V2.76: $oc has no CLAIM-001 section"; failures=1; }
grep -qF 'claim_check . $mds $src' "$root/$oc" || { echo "FAIL V2.76: the sweep does not pass source files to CLAIM-001"; failures=1; }
need "$pv" "the same sweep also reports schema DDL outside a migration" "the probe sweep items"
need "$pv" "DB-001" "the probe item for DB-001"
need "$pv" "DB-002" "the probe item for DB-002"
need "$pv" "NOT VERIFIED CLAIM-001" "the probe item for CLAIM-001"

command -v awk >/dev/null 2>&1 && command -v grep >/dev/null 2>&1 && command -v sed >/dev/null 2>&1 \
  && command -v mktemp >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.76: awk, grep, sed, or mktemp is not available, the behavioral fixture was not run"; exit 2; }

# Extracts the sh block of the CLAIM-001 section of workflows/output-compliance.md.
extract() {
  awk '
    /^### CLAIM-001 / { on = 1; next }
    /^### / { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; next }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v276.XXXXXX")"
(
  cd "$fx" || exit 1
  mkdir -p src
  extract > check.sh
  . ./check.sh || { echo "FAIL V2.76: CLAIM-001 does not load"; exit 1; }

  # The defective fixture: a source comment that claims a covering index, and a README that claims a faster query.
  printf '%s\n' "// The index is a covering index, so this query reads no table row." "export const find = () => 1;" > src/store.ts
  printf '%s\n' "# App" "The order query is faster after the change." > README.md
  defects="$(claim_check . src/store.ts README.md)"
  printf '%s\n' "$defects" | grep -qF 'NOT VERIFIED CLAIM-001: src/store.ts:1:' \
    || { echo "FAIL V2.76: CLAIM-001 did not report the covering-index comment with file:line: $defects"; exit 1; }
  printf '%s\n' "$defects" | grep -qF 'technical property claim' \
    || { echo "FAIL V2.76: CLAIM-001 did not label the technical-property claim"; exit 1; }
  printf '%s\n' "$defects" | grep -qF 'NOT VERIFIED CLAIM-001: README.md:2:' \
    || { echo "FAIL V2.76: CLAIM-001 did not report the performance claim with file:line: $defects"; exit 1; }
  printf '%s\n' "$defects" | grep -qF 'performance claim' \
    || { echo "FAIL V2.76: CLAIM-001 did not label the performance claim"; exit 1; }
  printf '%s\n' "$defects" | grep -q '^FAIL' \
    && { echo "FAIL V2.76: CLAIM-001 reports a claim as FAIL instead of NOT VERIFIED"; exit 1; }

  # The compliant twin: the same files with the claim replaced by the evidence that decides it.
  printf '%s\n' "// The query plan is recorded in docs/plan.md." "export const find = () => 1;" > src/store.ts
  printf '%s\n' "# App" "The order query timing is recorded in docs/measure.md." > README.md
  [ -z "$(claim_check . src/store.ts README.md)" ] \
    || { echo "FAIL V2.76: CLAIM-001 reports the compliant twin"; exit 1; }
  exit 0
) || failures=1

echo "V2.76 probe: fixture in $fx was checked and is removed"
rm -rf "$fx"

exit "$failures"
```

## V2.77 Regression — Detector Coverage, Open Ambiguity, and Propagated Corrections

Verify that a detector the system defines cannot be skipped by the procedure that a
verdict cites, and that a dimension with an open `AMBIGUOUS` criterion or a `PASS`
without evidence cannot be reported as passing. The external audit found a database
finding that the change-set procedure did not run, a performance criterion judged
`PASS` with no measurement, an ambiguity resolved by the auditor, and a claim corrected
in the test and the contract but left in a source comment:

- M-75: a detector or check the workflow defines is a control only when the procedure
  runs it (R-127). `V2.77` reads every rule heading of `workflows/output-compliance.md`
  and fails when the Procedure's coverage table does not name its identifier. The
  removal probe proves the check reports a row that is taken out.
- M-75: a criterion `AMBIGUOUS` on an applicable dimension is never `PASS`, and a `PASS`
  cites its evidence (R-128), in the criteria and in the code audit.
- M-75: a corrected claim is searched across the change set and the sweep (`CLAIM-001`
  and the Procedure step 5).

The text checks run on the repository. The mechanical part reads the headings and the
Procedure section, and the removal probe runs on a copy outside the repository. It needs
`awk`, `grep`, `sed`, `mktemp`, and `cp`; without them the probe is `NOT VERIFIED`.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
cg="workflows/code-audit.md"
ap="references/audit-pass-criteria.md"
vs="workflows/system-validation.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.77: $1 is missing $3"; failures=1; }
}

# Prints a FAIL line for every rule identifier of the headings that the Procedure does not name.
# $1 = the output-compliance file to check.
coverage_missing() {
  file="$1"
  proc="$(awk '/^## Procedure$/{on=1;next} /^## /{on=0} on' "$file")"
  rules="$(awk '/^### /' "$file")"
  printf '%s\n' "$rules" | while IFS= read -r h; do
    head="$(printf '%s' "$h" | sed 's/^### //; s/ —.*//')"
    for id in $(printf '%s' "$head" | grep -oE '[A-Z][A-Z0-9]*-[0-9]+'); do
      printf '%s\n' "$proc" | grep -qF -- "$id" \
        || echo "FAIL V2.77: the Procedure coverage table does not name $id (heading: $head)"
    done
  done
}

# The coverage check on the repository: every detector heading is named by the Procedure.
missing="$(coverage_missing "$root/$oc")"
[ -z "$missing" ] || { printf '%s\n' "$missing"; failures=1; }
grep -qF '| `DB-001` |' "$root/$oc" || { echo "FAIL V2.77: the coverage table has no DB-001 row"; failures=1; }
grep -q '^## Procedure$' "$root/$oc" || { echo "FAIL V2.77: $oc has no Procedure section"; failures=1; }

# M-75 wording and reachability.
for surface in "$ag" "$mr" "$rmx" "$vs" "$oc" "$cg" "$ap"; do
  need "$surface" "is a control only when the procedure that a verdict cites runs it" "the M-75 anchor"
done
for surface in "$ag" "$mr" "$ap" "$rmx" "$vs"; do
  need "$surface" 'is never `PASS`' "the AMBIGUOUS-is-never-PASS rule (M-75)"
done
need "$cg" "makes that dimension and the verdict" "the AMBIGUOUS verdict rule (M-75)"
need "$cg" "every row whose condition holds" "the coverage-table step of the code audit (M-75)"
need "$ap" "cites its evidence" "the PASS-with-evidence rule (M-75)"
for id in R-127 R-128; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.77: no $id row"; failures=1; }
done
grep -q "^| M-75 |" "$root/$mr" || { echo "FAIL V2.77: no M-75 row"; failures=1; }

command -v awk >/dev/null 2>&1 && command -v grep >/dev/null 2>&1 && command -v sed >/dev/null 2>&1 \
  && command -v mktemp >/dev/null 2>&1 && command -v cp >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.77: awk, grep, sed, mktemp, or cp is not available, the removal probe was not run"; exit 2; }

# Removal probe: a copy with the DB-001 row taken out must be reported; the repaired copy must not.
fx="$(mktemp -d "${TMPDIR:-/tmp}/v277.XXXXXX")"
(
  cp "$root/$oc" "$fx/removed.md"
  sed '/^| `DB-001` |/d' "$root/$oc" > "$fx/removed.md"
  removed="$(coverage_missing "$fx/removed.md")"
  printf '%s\n' "$removed" | grep -qF 'FAIL V2.77: the Procedure coverage table does not name DB-001' \
    || { echo "FAIL V2.77: the removed DB-001 row was not reported: $removed"; exit 1; }
  printf '%s\n' "$removed" | grep -qF 'DB-002' \
    && { echo "FAIL V2.77: the probe reports a row that was not removed"; exit 1; }
  cp "$root/$oc" "$fx/repaired.md"
  [ -z "$(coverage_missing "$fx/repaired.md")" ] \
    || { echo "FAIL V2.77: the repaired copy is still reported"; exit 1; }
  exit 0
) || failures=1

echo "V2.77 probe: fixture in $fx was checked and is removed"
rm -rf "$fx"

exit "$failures"
```

## V2.78 Regression — Controlled Clocks, Edge and Error Coverage, Design Claims, and Scope Values

Verify that the four rules added after the external audit that found a null-returning
work callback without a test, an SRP row contradicted by its unit, a documented error
path with no test, a timer hold that made a test depend on the clock, and an unrequested
change of a timeout value are each stated on every surface, and that each detector
reports the violation with `file:line` and passes its compliant twin:

- M-76: a test reaches its state through a controlled clock or an event, never through a
  real timer hold (R-129). `TEST-003` reports a real timer in a test with no fake timers.
- M-77: a test covers each admitted value and each documented error outcome (R-130). This
  is an auditor enumeration, reported in Dimension 7, and the check verifies its wording.
- M-78: an SRP row names the unit and its responsibilities, and a pattern names two
  consumers (R-131). `DEC-001` reports an SRP row with no `file:line` and a single-consumer
  pattern.
- M-79: a changed value of existing behavior is named in the scope manifest (R-132).
  `SCOPE-002` reports a changed timeout with `file:line` unless the manifest names it.

The static part reads the repository. The behavioral part extracts the `sh` blocks of the
three sections from `workflows/output-compliance.md` and runs them on fixtures in a
temporary directory outside the repository, which is removed. It needs `awk`, `grep`,
`sed`, `git`, and `mktemp`; without them the behavioral part is `NOT VERIFIED`.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
ap="references/audit-pass-criteria.md"
dp="references/design-patterns.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.78: $1 is missing $3"; failures=1; }
}

# Canonical wording of each new rule on every surface that must carry it.
need "$ag" "never through a real timer hold" "the M-76 anchor"
for s in "$mr" "$ap" "$oc" "$rmx"; do need "$s" "never through a real timer hold" "the M-76 anchor"; done
need "$ag" "every error outcome that a contract or decision record names" "the M-77 anchor"
for s in "$mr" "$ap" "$rmx"; do need "$s" "every error outcome that a contract or decision record names" "the M-77 anchor"; done
need "$ag" "an SRP row names the unit" "the M-78 anchor"
for s in "$mr" "$ap" "$oc" "$dp" "$rmx"; do need "$s" "an SRP row names the unit" "the M-78 anchor"; done
need "$ag" "a changed value of existing behavior" "the M-79 anchor"
for s in "$mr" "$ap" "$oc" "$rmx"; do need "$s" "a changed value of existing behavior" "the M-79 anchor"; done

for id in M-76 M-77 M-78 M-79; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.78: no $id row in the registry"; failures=1; }
done
for id in R-129 R-130 R-131 R-132; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.78: no $id row in the regression matrix"; failures=1; }
done

# The detectors exist, are named in the coverage table, and the sweep runs TEST-003.
for id in TEST-003 SCOPE-002; do
  grep -q "^### $id " "$root/$oc" || { echo "FAIL V2.78: $oc has no $id section"; failures=1; }
  grep -qF "| \`$id\` |" "$root/$oc" || { echo "FAIL V2.78: the coverage table has no $id row"; failures=1; }
done
grep -qF 'timer_hold_check . $tests' "$root/$oc" || { echo "FAIL V2.78: the sweep does not run TEST-003"; failures=1; }
grep -q 'the SRP row cites no file:line' "$root/$oc" \
  || { echo "FAIL V2.78: DEC-001 does not check the SRP reference"; failures=1; }
grep -q 'consumer(s) with file:line' "$root/$oc" || { echo "FAIL V2.78: DEC-001 does not count the consumers"; failures=1; }
grep -q 'Consumers (`file:line`)' "$root/$dp" || { echo "FAIL V2.78: the Presentation Record has no Consumers column"; failures=1; }

command -v awk >/dev/null 2>&1 && command -v grep >/dev/null 2>&1 && command -v sed >/dev/null 2>&1 \
  && command -v git >/dev/null 2>&1 && command -v mktemp >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.78: awk, grep, sed, git, or mktemp is not available, the behavioral fixtures were not run"; exit 2; }

# Extracts the sh block of one section of workflows/output-compliance.md. $1 = the rule identifier.
extract_sec() {
  awk -v h="$1" '
    $0 ~ ("^### " h " ") { on = 1; next }
    /^##/ { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; next }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v278.XXXXXX")"
(
  cd "$fx" || exit 1
  extract_sec TEST-003 > timer.sh
  extract_sec DEC-001 > dec.sh
  extract_sec SCOPE-002 > scope.sh
  . ./timer.sh && . ./dec.sh && . ./scope.sh || { echo "FAIL V2.78: the detector blocks do not load"; exit 1; }

  # M-76: a real timer hold is reported with file:line; the controlled-clock twin is not.
  mkdir -p tests/support
  printf '%s\n' "export const holdWriteLock = async () => {" \
    "  await new Promise((resolve) => setTimeout(resolve, 6000));" "};" > tests/support/hold-write-lock.ts
  printf '%s\n' "import { vi } from 'vitest';" "vi.useFakeTimers();" \
    "export const holdControlled = () => setTimeout(() => {}, 6000);" > tests/support/hold-controlled.ts
  timers="$(timer_hold_check . tests/support/hold-write-lock.ts tests/support/hold-controlled.ts)"
  printf '%s\n' "$timers" | grep -q 'FAIL TEST-003: tests/support/hold-write-lock.ts:2:' \
    || { echo "FAIL V2.78: TEST-003 did not report the real timer hold with file:line: $timers"; exit 1; }
  printf '%s\n' "$timers" | grep -q 'hold-controlled' \
    && { echo "FAIL V2.78: TEST-003 reports the controlled-clock twin"; exit 1; }

  # M-78: an SRP row without file:line and a single-consumer pattern are reported; the twin passes.
  mkdir -p docs/decisions
  {
    echo "# Decision"; echo
    echo "| Pattern | Force | Signal | Consumers (\`file:line\`) | Stack translation |"
    echo "|---|---|---|---|---|"
    echo "| Template Method | The skeleton is shared | BE10 | \`a.ts:1\` | Abstract class |"; echo
    echo "Count: 1 selected."; echo
    echo "| Principle | Status | Unit or mechanism | Evidence or reason |"
    echo "|---|---|---|---|"
    echo "| SRP | APPLIES | runner | keeps the transaction and the retry |"
    for p in OCP LSP ISP DIP; do echo "| $p | NOT APPLICABLE | none | no unit in scope |"; done
  } > docs/decisions/defective.md
  {
    echo "# Decision"; echo
    echo "| Pattern | Force | Signal | Consumers (\`file:line\`) | Stack translation |"
    echo "|---|---|---|---|---|"
    echo "| Template Method | The skeleton is shared | BE10 | \`a.ts:1\`, \`b.ts:4\` | Abstract class |"; echo
    echo "Count: 1 selected."; echo
    echo "| Principle | Status | Unit or mechanism | Evidence or reason |"
    echo "|---|---|---|---|"
    echo "| SRP | APPLIES | runner | \`runner.ts:12\` keeps the transaction, \`retry.ts:8\` keeps the retry |"
    for p in OCP LSP ISP DIP; do echo "| $p | NOT APPLICABLE | none | no unit in scope |"; done
  } > docs/decisions/compliant.md
  bad="$(decision_record_check . docs/decisions/defective.md)"
  printf '%s\n' "$bad" | grep -q 'FAIL DEC-001: docs/decisions/defective.md: the SRP row cites no file:line' \
    || { echo "FAIL V2.78: DEC-001 did not report the SRP row without file:line: $bad"; exit 1; }
  printf '%s\n' "$bad" | grep -q 'names 1 consumer(s)' \
    || { echo "FAIL V2.78: DEC-001 did not report the single-consumer pattern: $bad"; exit 1; }
  good="$(decision_record_check . docs/decisions/compliant.md)"
  [ -z "$good" ] || { echo "FAIL V2.78: DEC-001 reports the compliant twin: $good"; exit 1; }

  # M-79: a timeout moved from 5000 to 500 with no manifest line is reported with file:line; a named one passes.
  git init -q repo && cd repo || exit 1
  printf '%s\n' "export const config = {" "  busy_timeout: 5000," "};" > store.ts
  git add -A && git -c user.name=v278 -c user.email=v278@example.invalid commit -qm base
  sed 's/5000/500/' store.ts > store.tmp && mv store.tmp store.ts
  : > "$fx/empty.txt"
  unnamed="$(changed_value_check . "$fx/empty.txt" HEAD)"
  printf '%s\n' "$unnamed" | grep -q 'FAIL SCOPE-002: store.ts:2: candidate changed value 5000' \
    || { echo "FAIL V2.78: SCOPE-002 did not report the unnamed timeout with file:line: $unnamed"; exit 1; }
  printf 'store.ts busy_timeout 5000 => 500\n' > "$fx/named.txt"
  named="$(changed_value_check . "$fx/named.txt" HEAD)"
  [ -z "$named" ] || { echo "FAIL V2.78: SCOPE-002 reports the manifest-named timeout: $named"; exit 1; }
  exit 0
) || failures=1

echo "V2.78 probe: fixtures in $fx were checked and are removed"
rm -rf "$fx"

exit "$failures"
```

## V2.79 Regression — Commit Group Closure, Bound Tests, Comment Claims, and Citation Resolution

Verify that four gaps the external audit exposed are closed on every surface and by a
detector that reports the defect with `file:line`, and passes its compliant twin:

- M-05 and M-71: a commit group is self-contained, so no group depends on an uncommitted
  file of another group (R-133). `COMMIT-001` reports the dependency, and
  `workflows/commit.md` runs it on the proposed groups.
- M-54 and M-76: a declared bound is proven by a test that holds its dependency, for
  declared bounds, not only HTTP calls (R-134). `BOUND-001` reports a declared bound that
  no test holds.
- M-65: an ordering or traversal claim in a comment is checked against the code (R-135).
  `COMMENT-001` lists it as `NOT VERIFIED` until the reviewer cites the code.
- M-78: a citation in a design record resolves to the code it describes (R-136).
  `DEC-002` reports a citation that points to another line, a line past the end of the
  file, or a missing file.
- A whole-project gate that is red on files outside the change set is attributed as
  `BASELINE_FAILURE` in the commit proposal and never as the change set failing.

The static part reads the repository. The behavioral part extracts the `sh` blocks of the
four sections from `workflows/output-compliance.md` and runs them on fixtures in a
temporary directory outside the repository, which is removed. It needs `awk`, `grep`,
`sed`, `git`, `wc`, and `mktemp`; without them the behavioral part is `NOT VERIFIED`.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
cm="workflows/commit.md"
pv="workflows/project-validation.md"
ap="references/audit-pass-criteria.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.79: $1 is missing $3"; failures=1; }
}

# The four identifiers exist as sections, in the coverage table, and in the regression matrix.
for id in COMMIT-001 BOUND-001 COMMENT-001 DEC-002; do
  grep -q "^### $id " "$root/$oc" || { echo "FAIL V2.79: $oc has no $id section"; failures=1; }
  grep -qF "| \`$id\` |" "$root/$oc" || { echo "FAIL V2.79: the coverage table has no $id row"; failures=1; }
done
for id in R-133 R-134 R-135 R-136; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.79: no $id row in the regression matrix"; failures=1; }
done

# The sweep runs the project-wide detectors, and the commit workflow runs the group closure.
grep -qF 'bound_test_check . $src -- $tests' "$root/$oc" || { echo "FAIL V2.79: the sweep does not run BOUND-001"; failures=1; }
grep -qF 'comment_claim_check . $src' "$root/$oc" || { echo "FAIL V2.79: the sweep does not run COMMENT-001"; failures=1; }
grep -qF 'decision_citation_check . $decisions' "$root/$oc" || { echo "FAIL V2.79: the sweep does not run DEC-002"; failures=1; }
grep -qF 'commit_group_closure_check' "$root/$cm" || { echo "FAIL V2.79: the commit workflow does not run COMMIT-001 on each group"; failures=1; }
grep -qF 'BASELINE_FAILURE' "$root/$cm" || { echo "FAIL V2.79: the commit gate has no BASELINE_FAILURE attribution"; failures=1; }
grep -qF 'intersect its failing files with the change set' "$root/$pv" \
  || { echo "FAIL V2.79: the project gate does not attribute failing files to the change set"; failures=1; }

# The audit criteria name the new detectors on the dimensions they serve.
for id in COMMENT-001 BOUND-001 DEC-002 COMMIT-001; do
  grep -qF "\`$id\`" "$root/$ap" || { echo "FAIL V2.79: $ap does not name $id"; failures=1; }
done

# Canonical wording of the rules these detectors enforce is still on the surfaces that carry it.
need "$ag" "never through a real timer hold" "the M-76 anchor"
need "$ag" "bounds the call only when the value reaches the timer" "the M-54 anchor"
need "$ag" "A timeout declared as a value" "the M-54 declared-value rule"

command -v awk >/dev/null 2>&1 && command -v grep >/dev/null 2>&1 && command -v sed >/dev/null 2>&1 \
  && command -v git >/dev/null 2>&1 && command -v wc >/dev/null 2>&1 && command -v mktemp >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.79: awk, grep, sed, git, wc, or mktemp is not available, the behavioral fixtures were not run"; exit 2; }

# Extracts the sh block of one section of workflows/output-compliance.md. $1 = the rule identifier.
extract_sec() {
  awk -v h="$1" '
    $0 ~ ("^### " h " ") { on = 1; next }
    /^##/ { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; next }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v279.XXXXXX")"
(
  cd "$fx" || exit 1
  extract_sec COMMIT-001 > commit.sh
  extract_sec BOUND-001 > bound.sh
  extract_sec COMMENT-001 > comment.sh
  extract_sec DEC-002 > citation.sh
  . ./commit.sh && . ./bound.sh && . ./comment.sh && . ./citation.sh \
    || { echo "FAIL V2.79: the detector blocks do not load"; exit 1; }

  # R-133: a group that imports an uncommitted file is reported; the twin that includes it passes.
  git init -q repo && cd repo || exit 1
  mkdir -p src/db docs/decisions
  printf '%s\n' "# repo" > README.md
  git add -A && git -c user.name=v279 -c user.email=v279@example.invalid commit -qm base
  printf '%s\n' 'import { store } from "./store";' 'export const a = store;' > src/db/a.ts
  printf '%s\n' 'export const store = 1;' > src/db/store.ts
  open="$(commit_group_closure_check . HEAD src/db/a.ts)"
  printf '%s\n' "$open" | grep -q 'FAIL COMMIT-001: src/db/a.ts:1: import ./store resolves neither' \
    || { echo "FAIL V2.79: COMMIT-001 did not report the uncommitted import with file:line: $open"; exit 1; }
  closed="$(commit_group_closure_check . HEAD src/db/a.ts src/db/store.ts)"
  [ -z "$closed" ] || { echo "FAIL V2.79: COMMIT-001 reports the closed twin: $closed"; exit 1; }

  # R-134: a declared busy timeout with no holding test is reported; the twin with a never-resolving test passes.
  printf '%s\n' 'export const cfg = { busy_timeout: 5000 };' > src/db/timed.ts
  printf '%s\n' 'import { cfg } from "../src/db/timed";' 'test("x", () => cfg);' > src/db/timed.plain.test.ts
  nohold="$(bound_test_check . src/db/timed.ts -- src/db/timed.plain.test.ts)"
  printf '%s\n' "$nohold" | grep -q 'FAIL BOUND-001: src/db/timed.ts:1: no test that imports timed holds the dependency' \
    || { echo "FAIL V2.79: BOUND-001 did not report the bound with no holding test: $nohold"; exit 1; }
  printf '%s\n' 'import { cfg } from "../src/db/timed";' 'test("hold", () => { new Promise(() => {}); cfg; });' > src/db/timed.hold.test.ts
  held="$(bound_test_check . src/db/timed.ts -- src/db/timed.hold.test.ts)"
  [ -z "$held" ] || { echo "FAIL V2.79: BOUND-001 reports the held twin: $held"; exit 1; }

  # R-135: an ordering claim in a comment is listed with file:line; a plain comment is not.
  printf '%s\n' '/**' ' * Counts the rows in the order the cursor walks them.' ' */' 'export const f = () => 1;' > src/db/claim.ts
  printf '%s\n' '// plain comment' 'export const g = () => 1;' > src/db/plain.ts
  claims="$(comment_claim_check . src/db/claim.ts src/db/plain.ts)"
  printf '%s\n' "$claims" | grep -q 'NOT VERIFIED COMMENT-001: src/db/claim.ts:2:' \
    || { echo "FAIL V2.79: COMMENT-001 did not list the ordering claim with file:line: $claims"; exit 1; }
  printf '%s\n' "$claims" | grep -q 'plain.ts' \
    && { echo "FAIL V2.79: COMMENT-001 lists the plain comment: $claims"; exit 1; }

  # R-136: a citation to the wrong line and one past the end are reported; the citation to the identifier's line passes.
  awk 'BEGIN { for (i = 1; i <= 200; i++) { if (i == 59) print "export const parse = () => 1;"; else if (i == 150) print "const other = 1;"; else print "// line " i } }' > src/cli.ts
  printf '%s\n' '| Principle | Status | Unit | Evidence |' '|---|---|---|---|' \
    '| SRP | APPLIES | runner | `parse` at `src/cli.ts:150` |' \
    '| SRP | APPLIES | runner | `parse` at `src/cli.ts:999` |' > docs/decisions/defective.md
  printf '%s\n' '| Principle | Status | Unit | Evidence |' '|---|---|---|---|' \
    '| SRP | APPLIES | runner | `parse` at `src/cli.ts:59` |' > docs/decisions/compliant.md
  bad="$(decision_citation_check . docs/decisions/defective.md)"
  printf '%s\n' "$bad" | grep -q 'FAIL DEC-002: docs/decisions/defective.md:3: src/cli.ts:150 does not show parse' \
    || { echo "FAIL V2.79: DEC-002 did not report the wrong-line citation with file:line: $bad"; exit 1; }
  printf '%s\n' "$bad" | grep -q 'src/cli.ts:999 is beyond the end of the file' \
    || { echo "FAIL V2.79: DEC-002 did not report the citation past the end: $bad"; exit 1; }
  good="$(decision_citation_check . docs/decisions/compliant.md)"
  [ -z "$good" ] || { echo "FAIL V2.79: DEC-002 reports the compliant twin: $good"; exit 1; }
  exit 0
) || failures=1

echo "V2.79 probe: fixtures in $fx were checked and are removed"
rm -rf "$fx"

exit "$failures"
```

## V2.80 Regression — Verdict Matrix Consistency, Legacy Records, and Runtime Equivalence

Verify that four gaps an external audit exposed are closed on every surface and by a
detector that reports the defect with `file:line`, and passes its compliant twin:

- M-80: a report's verdict matrix is consistent with its own rules (R-137). A
  `NOT VERIFIED` reason is never an unexecuted review; a `PASS` never carries an open
  item (`pendiente`, `diff leído`); a grouped or missing dimension is a defect; a
  conclusion never claims a closure a row leaves open; a red whole-project gate is
  `BASELINE_FAILURE`; an equivalence claim is `NOT VERIFIED` until executed. `REPORT-001`
  reports each defect, and `workflows/system-audit.md` Phase 5 runs it before delivery.
- The legacy-record gap: a change that alters how stored values are read or normalized
  states the treatment of existing records. Dimension 5 of `references/audit-pass-criteria.md`
  carries the criterion, and an undecided treatment is `AMBIGUOUS`, never `PASS`.
- The runtime-equivalence gap: a validation claimed as equivalent on another runtime is
  `NOT VERIFIED` with the version named (M-80, `REPORT-001`).
- The baseline gap: a whole-project gate that is red outside the change set is reported as
  `BASELINE_FAILURE` in the audit and review paths, not only in the commit path.

The static part reads the repository. The behavioral part extracts the `sh` block of
`REPORT-001` from `workflows/output-compliance.md`, runs it on a defective matrix that
carries each defect and on its compliant twin, in a temporary directory outside the
repository, which is removed. It needs `awk`, `grep`, `sed`, and `mktemp`; without them the
behavioral part is `NOT VERIFIED`.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
ap="references/audit-pass-criteria.md"
sa="workflows/system-audit.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.80: $1 is missing $3"; failures=1; }
}

# The rule, its registry row, its regression row, and its detector are on every surface.
grep -q '^| M-80 |' "$root/$mr" || { echo "FAIL V2.80: no M-80 row in the registry"; failures=1; }
grep -q '^| R-137 |' "$root/$rmx" || { echo "FAIL V2.80: no R-137 row in the regression matrix"; failures=1; }
grep -qF '| `REPORT-001` |' "$root/$oc" || { echo "FAIL V2.80: the coverage table has no REPORT-001 row"; failures=1; }
grep -q '^### REPORT-001 ' "$root/$oc" || { echo "FAIL V2.80: $oc has no REPORT-001 section"; failures=1; }
grep -qF 'REPORT-001' "$root/$sa" || { echo "FAIL V2.80: the system audit does not run REPORT-001 before delivery"; failures=1; }
grep -qF 'REPORT-001' "$root/$ap" || { echo "FAIL V2.80: the audit criteria do not name REPORT-001"; failures=1; }
need "$ag" "verdict matrix is consistent with its own rules" "the M-80 canonical wording"
need "$ap" "verdict matrix is consistent with its own rules" "the M-80 criterion"
need "$ap" "legacy" "the legacy-record criterion of Dimension 5"
need "$oc" "verdict matrix is consistent with its own rules" "the M-80 detector statement"
need "$sa" "verdict matrix is consistent with its own rules" "the M-80 verdict check of Phase 5"
need "$rmx" "verdict matrix is consistent with its own rules" "the R-137 statement"

command -v awk >/dev/null 2>&1 && command -v grep >/dev/null 2>&1 && command -v sed >/dev/null 2>&1 \
  && command -v mktemp >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.80: awk, grep, sed, or mktemp is not available, the behavioral fixtures were not run"; exit 2; }

# Extracts the sh block of one section of workflows/output-compliance.md. $1 = the rule identifier.
extract_sec() {
  awk -v h="$1" '
    $0 ~ ("^### " h " ") { on = 1; next }
    /^##/ { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; next }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v280.XXXXXX")"
(
  cd "$fx" || exit 1
  extract_sec REPORT-001 > report.sh
  . ./report.sh || { echo "FAIL V2.80: the REPORT-001 block does not load"; exit 1; }
  mkdir -p reports
  cd "$fx" || exit 1

  # The defective matrix: one line per defect of the external audit.
  printf '%s\n' \
    '# Audit' \
    '| Dimensión | Estado | Motivo |' \
    '|---|---|---|' \
    '| 1 Arquitectura | NOT VERIFIED | No ejecuté la verificación estructural. |' \
    '| 2 Patrones | PASS | Sin patrones nuevos en el diff leído. |' \
    '| 3 Calidad | PASS | Pendiente Node 22. |' \
    '| 4 API | AMBIGUOUS | Sin contrato definido. |' \
    '| 9, 10, 12, 13 | N/A | Sin interfaz de usuario. |' \
    '| 5 Datos | NOT VERIFIED | Revisión de registros legados pendiente. |' \
    'Cierra ambas brechas del sistema.' \
    'npm run check falla en format:check por 17 archivos fuera del cambio.' \
    'La versión Node 22 es equivalente a la probada.' > reports/defect.md

  # The compliant twin: every dimension has a row, no open item, a red gate attributed, and no equivalence claim.
  {
    printf '%s\n' '# Audit' '| Dimensión | Estado | Motivo |' '|---|---|---|'
    printf '%s\n' \
      '| 1 Arquitectura | PASS | Evidencia: output-compliance sin hallazgos en el cambio. |' \
      '| 2 Patrones | PASS | Selección vacía justificada. |' \
      '| 3 Calidad | PASS | Evidencia: npm run lint con código de salida 0. |' \
      '| 4 API | NOT VERIFIED | Requiere revisión de contrato externo. |' \
      '| 5 Datos | AMBIGUOUS | El tratamiento de los registros legados lo decide el usuario. |' \
      '| 6 Seguridad | N/A | El cambio no añade autenticación ni secretos. |' \
      '| 7 Pruebas | PASS | Evidencia: 515 pruebas con código de salida 0. |' \
      '| 8 Rendimiento | N/A | Sin superficie web. |' \
      '| 9 Interfaz | N/A | Sin interfaz de usuario. |' \
      '| 10 Accesibilidad | N/A | Sin interfaz de usuario. |' \
      '| 11 Internacionalización | PASS | El formateador exige locale. |' \
      '| 12 Estados | N/A | Sin interfaz de usuario. |' \
      '| 13 Responsive | N/A | Sin interfaz de usuario. |' \
      '| 14 Organización | N/A | Un solo paquete. |' \
      'npm run check falla en format:check: 17 archivos, BASELINE_FAILURE, fuera del cambio.'
  } > reports/twin.md

  defect="$(report_matrix_check . reports/defect.md)"
  for want in \
    'FAIL REPORT-001: reports/defect.md:4: NOT VERIFIED reason is an unexecuted review' \
    'FAIL REPORT-001: reports/defect.md:5: PASS carries an open item' \
    'FAIL REPORT-001: reports/defect.md:6: PASS carries an open item' \
    'FAIL REPORT-001: reports/defect.md:8: grouped dimension' \
    'FAIL REPORT-001: reports/defect.md: lines 10 claim a closure' \
    'FAIL REPORT-001: reports/defect.md:11: a red whole-project gate is not attributed BASELINE_FAILURE' \
    'NOT VERIFIED REPORT-001: reports/defect.md:12: a runtime or version equivalence is claimed' \
    'FAIL REPORT-001: reports/defect.md: dimension 6 has no row' \
    'FAIL REPORT-001: reports/defect.md: dimension 11 has no row'; do
    printf '%s\n' "$defect" | grep -qF -- "$want" \
      || { echo "FAIL V2.80: REPORT-001 did not report '$want' with file:line: $defect"; failures=1; }
  done

  twin="$(report_matrix_check . reports/twin.md)"
  [ -z "$twin" ] || { echo "FAIL V2.80: REPORT-001 reports the compliant twin: $twin"; failures=1; }
  exit 0
) || failures=1

echo "V2.80 probe: fixtures in $fx were checked and are removed"
rm -rf "$fx"

exit "$failures"
```

## V2.81 Regression — Typed Errors, Universal and Cost Claims, and the Detectors' Coverage

Verify that three gaps an external audit exposed are closed on every surface and by a
detector that reports the defect with `file:line`, and passes its compliant twin:

- M-81: a failure raised from a module uses the module's typed error class, never a
  generic `Error` (R-138). The rule was held only by a Skill and the audit criteria, so it
  did not reach every client; it is now canonical in `AGENTS.md`, and `ERR-001` reports a
  bare `Error` in a project that declares a typed error class. The error model is read from
  the whole project (see V2.82): the folder-local reading let a module escape the rule.
- M-65 and M-68: a universal claim written with `always` or `never`, and a cost or speed
  claim such as `lo caro` in a comment, are listed as `NOT VERIFIED` by `CLAIM-001`. The
  detector vocabulary was narrower than the rule, which says `the equivalent` of `every`.
- Coverage: every detector of `workflows/output-compliance.md` that the sweep runs has its
  coverage row and its sweep call, so a removed call is reported (R-138, the V2.77 rule).

The static part reads the repository. The behavioral part extracts the `sh` blocks of
`ERR-001` and `CLAIM-001` from `workflows/output-compliance.md`, runs them on fixtures in a
temporary directory outside the repository, which is removed, and runs a removal probe on
a copy of the sweep. It needs `awk`, `grep`, `sed`, and `mktemp`; without them the
behavioral part is `NOT VERIFIED`.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
ap="references/audit-pass-criteria.md"
ir="skills/implementation-recipes/SKILL.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.81: $1 is missing $3"; failures=1; }
}

# The rule, its registry row, its regression row, and its detector are on every surface.
grep -q '^| M-81 |' "$root/$mr" || { echo "FAIL V2.81: no M-81 row in the registry"; failures=1; }
grep -q '^| R-138 |' "$root/$rmx" || { echo "FAIL V2.81: no R-138 row in the regression matrix"; failures=1; }
grep -q '^### ERR-001 ' "$root/$oc" || { echo "FAIL V2.81: $oc has no ERR-001 section"; failures=1; }
grep -qF '| `ERR-001` |' "$root/$oc" || { echo "FAIL V2.81: the coverage table has no ERR-001 row"; failures=1; }
grep -qF 'error_model_check . $src' "$root/$oc" || { echo "FAIL V2.81: the sweep does not run ERR-001"; failures=1; }
need "$ag" "a failure raised from a module uses the module's typed error class" "the M-81 canonical wording"
need "$mr" "a failure raised from a module uses the module" "the M-81 registry anchor"
need "$ap" "a failure raised from a module uses the module's typed error class" "the M-81 criterion of Dimension 3"
need "$ir" "a failure raised from a module uses" "the M-81 recipe"
need "$oc" "a failure raised from a module uses the module's typed error class" "the M-81 detector statement"
need "$rmx" "R-138" "the R-138 row"
grep -qF 'all|always|never|' "$root/$oc" || { echo "FAIL V2.81: CLAIM-001 has no always and never vocabulary"; failures=1; }
grep -qF 'lo caro' "$root/$oc" || { echo "FAIL V2.81: CLAIM-001 has no cost vocabulary"; failures=1; }

command -v awk >/dev/null 2>&1 && command -v grep >/dev/null 2>&1 && command -v sed >/dev/null 2>&1 \
  && command -v mktemp >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.81: awk, grep, sed, or mktemp is not available, the behavioral fixtures were not run"; exit 2; }

# Extracts the sh block of one section of workflows/output-compliance.md. $1 = the rule identifier.
extract_sec() {
  awk -v h="$1" '
    $0 ~ ("^### " h " ") { on = 1; next }
    /^##/ { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; next }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v281.XXXXXX")"
(
  cd "$fx" || exit 1
  extract_sec ERR-001 > err.sh
  extract_sec CLAIM-001 > claim.sh
  . ./err.sh && . ./claim.sh || { echo "FAIL V2.81: the detector blocks do not load"; exit 1; }
  mkdir -p src/orders src/billing docs

  # M-81, defect: a bare Error in a module whose sibling error file extends Error.
  printf '%s\n' 'export class OrderImportError extends Error {}' > src/orders/order-import-error.ts
  printf '%s\n' 'export const parse = (v: string) => { throw new Error("bad " + v); };' > src/orders/order-timestamp.ts
  printf '%s\n' 'export const parseOk = (v: string) => { throw new OrderImportError(v); };' > src/orders/order-timestamp-ok.ts
  # M-81, project scope: a module whose folder holds no error file is matched when the project
  # declares a typed class elsewhere (the folder-local reading let it escape).
  mkdir -p src/payments
  printf '%s\n' 'export const pay = () => { throw new Error("declined"); };' > src/payments/pay.ts

  defect="$(error_model_check . src/orders/order-timestamp.ts src/payments/pay.ts)"
  printf '%s\n' "$defect" | grep -qF 'FAIL ERR-001: src/orders/order-timestamp.ts:1:' \
    || { echo "FAIL V2.81: ERR-001 did not report the bare Error with file:line: $defect"; exit 1; }
  printf '%s\n' "$defect" | grep -qF 'FAIL ERR-001: src/payments/pay.ts:1:' \
    || { echo "FAIL V2.81: ERR-001 did not report the module with no error file in its folder: $defect"; exit 1; }

  # M-81, scope limit: a project that declares no typed error class is not matched.
  mkdir -p nomodel/src/billing
  printf '%s\n' 'export const charge = () => { throw new Error("card"); };' > nomodel/src/billing/charge.ts
  nomodel="$(error_model_check nomodel src/billing/charge.ts)"
  [ -z "$nomodel" ] || { echo "FAIL V2.81: ERR-001 reports a project with no error model: $nomodel"; exit 1; }
  twin="$(error_model_check . src/orders/order-timestamp-ok.ts)"
  [ -z "$twin" ] || { echo "FAIL V2.81: ERR-001 reports the typed twin: $twin"; exit 1; }

  # CLAIM-001: an always-claim in a document and a cost claim in a comment are listed with file:line.
  printf '%s\n' '# Contract' '' 'The offset is always UTC.' > docs/contract.md
  printf '%s\n' '// Building the formatter is lo caro.' 'export const f = () => 1;' > src/format.ts
  printf '%s\n' '// plain comment' 'export const g = () => 1;' > src/format-plain.ts
  printf '%s\n' '# Contract' '' 'The offset is preserved as stored.' > docs/contract-ok.md
  claims="$(claim_check . docs/contract.md src/format.ts src/format-plain.ts)"
  printf '%s\n' "$claims" | grep -qF 'NOT VERIFIED CLAIM-001: docs/contract.md:3:' \
    || { echo "FAIL V2.81: CLAIM-001 did not list the always-claim with file:line: $claims"; exit 1; }
  printf '%s\n' "$claims" | grep -qF 'NOT VERIFIED CLAIM-001: src/format.ts:1:' \
    || { echo "FAIL V2.81: CLAIM-001 did not list the cost claim with file:line: $claims"; exit 1; }
  printf '%s\n' "$claims" | grep -q 'format-plain' \
    && { echo "FAIL V2.81: CLAIM-001 lists the plain comment: $claims"; exit 1; }
  quiet="$(claim_check . docs/contract-ok.md src/format-plain.ts)"
  [ -z "$quiet" ] || { echo "FAIL V2.81: CLAIM-001 reports the compliant twin: $quiet"; exit 1; }
  exit 0
) || failures=1

# Removal probe: a copy of the sweep with the ERR-001 call taken out must no longer run it.
sed '/^  error_model_check \. \$src$/d' "$root/$oc" > "$fx/sweep-removed.md"
grep -qF 'error_model_check . $src' "$fx/sweep-removed.md" \
  && { echo "FAIL V2.81: the removal probe did not remove the ERR-001 call"; failures=1; }

echo "V2.81 probe: fixtures in $fx were checked and are removed"
rm -rf "$fx"

exit "$failures"
```

## V2.82 Regression — Project-Wide Error Model, Error Codes in Both Directions, Entry Points, Validation Order, Read Transactions, and Retry Budgets

Verify that six gaps an external audit exposed are closed on every surface and by a
detector that reports the defect with `file:line` and passes its compliant twin:

- M-81: the error model is a project property. A module whose folder holds no error file
  is matched when the project declares a typed class elsewhere (R-138). The folder-local
  wording is retired and may not return.
- M-77: error codes are declared in both directions. `ERR-002` reports a documented code
  that no test names; `ERR-003` reports an emitted code that no contract declares (R-140).
- M-82: a project's entry point is tested through its real invocation, and its exit status
  is asserted. `ENTRY-001` reports an entry point no test invokes, or one whose status no
  test asserts (R-139).
- M-83: input is validated before any side effect. `VAL-001` reports a resource opened
  before its validator, and lists an opening whose validator is in another file as
  `NOT VERIFIED` (R-141).
- M-84: a paginated read in a loop runs in one read transaction or snapshot. `DB-003`
  reports a page loop with no transaction marker (R-142).
- M-54: a retry budget is a declared bound. `BOUND-001` covers `maxAttempts` and
  `retries` budgets, not only timeouts (R-143).

The static part reads the repository. The behavioral part extracts the `sh` blocks of the
detectors from `workflows/output-compliance.md`, runs them on fixtures in a temporary
directory outside the repository, which is removed, and runs a removal probe on a copy of
the sweep. It needs `awk`, `grep`, `sed`, `tr`, and `mktemp`; without them the behavioral
part is `NOT VERIFIED`.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
ap="references/audit-pass-criteria.md"
ir="skills/implementation-recipes/SKILL.md"
ts="skills/testing-strategy/SKILL.md"
db="skills/database-engineering/SKILL.md"
vs="workflows/system-validation.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.82: $1 is missing $3"; failures=1; }
}

# The canonical wording of the four rules, and of the two expanded rules, is in AGENTS.md.
need "$ag" "unless the project has no error model and that is stated" "the M-81 project-scope wording"
need "$ag" "every error code a boundary emits is declared in the contract" "the M-77 two-direction wording"
need "$ag" "is tested through its real invocation" "the M-82 canonical wording"
need "$ag" "is validated before any side effect" "the M-83 canonical wording"
need "$ag" "runs in one read transaction or snapshot" "the M-84 canonical wording"
need "$ag" "a retry budget (the number of attempts" "the M-54 retry-budget wording"

# Every required surface of M-81 to M-84 carries its anchor.
need "$ir" "unless the project has no error model and that is stated" "the M-81 recipe"
need "$ap" "unless the project has no error model and that is stated" "the M-81 criterion of Dimension 3"
need "$oc" "unless the project has no error model and that is stated" "the M-81 detector statement"
need "$ts" "tested through its real invocation" "the M-82 testing-strategy instruction"
need "$ir" "tested through its real invocation" "the M-82 recipe"
need "$ap" "is tested through its real invocation" "the M-82 criterion of Dimension 7"
need "$oc" "tested through its real invocation" "the M-82 detector statement"
need "$mr" "tested through its real invocation" "the M-82 registry anchor"
need "$ir" "is validated before any side effect" "the M-83 recipe"
need "$ap" "is validated before any side effect" "the M-83 criterion of Dimension 3"
need "$oc" "is validated before any side effect" "the M-83 detector statement"
need "$mr" "is validated before any side effect" "the M-83 registry anchor"
need "$db" "runs in one read transaction or snapshot" "the M-84 database instruction"
need "$ir" "runs in one read transaction or snapshot" "the M-84 recipe"
need "$ap" "runs in one read transaction or snapshot" "the M-84 criterion of Dimension 5"
need "$oc" "runs in one read transaction or snapshot" "the M-84 detector statement"
need "$mr" "runs in one read transaction or snapshot" "the M-84 registry anchor"
need "$ap" "every error code a boundary emits is declared" "the M-77 criterion of Dimension 4"
need "$vs" "is validated before any side effect" "the M-83 statement of this check"
need "$vs" "runs in one read transaction or snapshot" "the M-84 statement of this check"

# Registry rows, regression rows, sections, coverage rows, and sweep calls.
for id in M-82 M-83 M-84; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.82: no $id row in the registry"; failures=1; }
done
for id in R-139 R-140 R-141 R-142 R-143; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.82: no $id row in the regression matrix"; failures=1; }
done
for id in ERR-002 ERR-003 ENTRY-001 VAL-001 DB-003; do
  grep -qF "| \`$id\` |" "$root/$oc" || { echo "FAIL V2.82: the coverage table has no $id row"; failures=1; }
done
for h in "### ERR-002 " "### ENTRY-001 " "### VAL-001 " "### DB-003 "; do
  grep -q "^$h" "$root/$oc" || { echo "FAIL V2.82: $oc has no section starting with '$h'"; failures=1; }
done
for call in 'error_outcome_check . $src -- $tests -- $contracts' 'entry_point_check . $src -- $tests' 'validation_order_check . $src' 'read_transaction_check . $src' 'error_model_check . $src' 'bound_test_check . $src -- $tests'; do
  grep -qF -- "  $call" "$root/$oc" || { echo "FAIL V2.82: the sweep does not run: $call"; failures=1; }
done

# The folder-local wording of the error model is retired. The registry and this check record it on purpose.
retired="$(grep -rlF 'a module has an error model when a sibling file' "$root" --include='*.md' | grep -vE '/(references/mandatory-rules\.md|workflows/system-validation\.md)$')"
[ -z "$retired" ] || { echo "FAIL V2.82: the folder-local error model wording is still present in: $retired"; failures=1; }

command -v awk >/dev/null 2>&1 && command -v grep >/dev/null 2>&1 && command -v sed >/dev/null 2>&1 \
  && command -v tr >/dev/null 2>&1 && command -v mktemp >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.82: awk, grep, sed, tr, or mktemp is not available, the behavioral fixtures were not run"; exit 2; }

# Extracts the sh blocks of one section of workflows/output-compliance.md. $1 = the rule identifier.
extract_sec() {
  awk -v h="$1" '
    $0 ~ ("^### " h " ") { on = 1; next }
    /^##/ { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; next }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v282.XXXXXX")"
(
  cd "$fx" || exit 1
  extract_sec ERR-001 > err1.sh
  extract_sec ERR-002 > err2.sh
  extract_sec ENTRY-001 > entry.sh
  extract_sec VAL-001 > val.sh
  extract_sec DB-003 > db3.sh
  extract_sec BOUND-001 > bound.sh
  extract_sec REPORT-001 > rep.sh
  . ./err1.sh && . ./err2.sh && . ./entry.sh && . ./val.sh && . ./db3.sh && . ./bound.sh \
    || { echo "FAIL V2.82: the detector blocks do not load"; exit 1; }

  # M-81, defect: a module whose folder holds no error file, in a project that declares a typed class elsewhere.
  mkdir -p src/orders src/payments
  printf '%s\n' 'export class OrderImportError extends Error {}' > src/orders/order-import-error.ts
  printf '%s\n' 'export const pay = () => { throw new Error("declined"); };' > src/payments/pay.ts
  printf '%s\n' 'export const pay = () => { throw new OrderImportError("declined"); };' > src/payments/pay-ok.ts
  e1="$(error_model_check . src/payments/pay.ts)"
  printf '%s\n' "$e1" | grep -qF 'FAIL ERR-001: src/payments/pay.ts:1:' \
    || { echo "FAIL V2.82: ERR-001 did not report the module outside the error folder: $e1"; exit 1; }
  e2="$(error_model_check . src/payments/pay-ok.ts)"
  [ -z "$e2" ] || { echo "FAIL V2.82: ERR-001 reports the typed twin: $e2"; exit 1; }

  # M-77, defect: a documented code with no test, and an emitted code no contract declares.
  mkdir -p tests docs
  printf '%s\n' '# ETL contract' '' '`ETL_SOURCE_FAILED` is raised when the source fails.' '`ETL_OK_CODE` is an error code for a clean run.' > docs/contract.md
  printf '%s\n' 'throw new EtlError("ETL_SOURCE_FAILED");' 'const code = "ETL_UNDECLARED";' 'export const ok = "ETL_OK_CODE";' > src/etl.ts
  printf '%s\n' 'test("ETL_OK_CODE", () => {});' > tests/etl.test.ts
  e3="$(error_outcome_check . src/etl.ts -- tests/etl.test.ts -- docs/contract.md)"
  printf '%s\n' "$e3" | grep -qF 'FAIL ERR-002: docs/contract.md:3:' \
    || { echo "FAIL V2.82: ERR-002 did not report the documented code with no test: $e3"; exit 1; }
  printf '%s\n' "$e3" | grep -qF 'FAIL ERR-003: src/etl.ts:2:' \
    || { echo "FAIL V2.82: ERR-003 did not report the emitted code no contract declares: $e3"; exit 1; }
  printf '%s\n' 'export const ok = "ETL_OK_CODE";' > src/etl-ok.ts
  printf '%s\n' '# ETL contract' '' '`ETL_OK_CODE` is an error code for a clean run.' > docs/contract-ok.md
  e4="$(error_outcome_check . src/etl-ok.ts -- tests/etl.test.ts -- docs/contract-ok.md)"
  [ -z "$e4" ] || { echo "FAIL V2.82: ERR-002 or ERR-003 reports the declared and tested twin: $e4"; exit 1; }

  # M-82, defect: an entry point no test invokes; then one whose exit status no test asserts.
  mkdir -p src/cli
  printf '%s\n' 'const args = process.argv.slice(2);' 'process.exit(args.length ? 0 : 2);' > src/cli/run.ts
  printf '%s\n' 'import { add } from "../src/math";' 'test("adds", () => {});' > tests/math.test.ts
  e5="$(entry_point_check . src/cli/run.ts -- tests/math.test.ts)"
  printf '%s\n' "$e5" | grep -qF 'FAIL ENTRY-001: src/cli/run.ts:1: an entry point that no test invokes' \
    || { echo "FAIL V2.82: ENTRY-001 did not report the entry point no test invokes: $e5"; exit 1; }
  printf '%s\n' 'import { execFile } from "node:child_process";' 'test("runs", async () => { await execFile("tsx", ["src/cli/run.ts"]); });' > tests/run-noexit.test.ts
  e6="$(entry_point_check . src/cli/run.ts -- tests/run-noexit.test.ts)"
  printf '%s\n' "$e6" | grep -qF 'FAIL ENTRY-001: src/cli/run.ts:1: the entry point sets an exit status' \
    || { echo "FAIL V2.82: ENTRY-001 did not report the status no test asserts: $e6"; exit 1; }
  printf '%s\n' 'import { execFile } from "node:child_process";' 'test("exits 2 without arguments", async () => { const r = await execFile("tsx", ["src/cli/run.ts"]); expect(r.exitCode).toBe(2); });' > tests/run.test.ts
  e7="$(entry_point_check . src/cli/run.ts -- tests/run.test.ts)"
  [ -z "$e7" ] || { echo "FAIL V2.82: ENTRY-001 reports the tested twin: $e7"; exit 1; }

  # M-83, defect: a database opened before the argument is parsed; the twin validates first;
  # an opening whose validator is in another file is listed as NOT VERIFIED.
  printf '%s\n' 'const db = new Database(process.argv[3]);' 'const day = parseDay(process.argv[2]);' > src/cli/bad.ts
  printf '%s\n' 'const day = parseDay(process.argv[2]);' 'const db = new Database(process.argv[3]);' > src/cli/good.ts
  printf '%s\n' 'const db = new Database(process.argv[3]);' > src/cli/split.ts
  v1="$(validation_order_check . src/cli/bad.ts)"
  printf '%s\n' "$v1" | grep -qF 'FAIL VAL-001: src/cli/bad.ts:1:' \
    || { echo "FAIL V2.82: VAL-001 did not report the database opened before its validator: $v1"; exit 1; }
  v2="$(validation_order_check . src/cli/good.ts)"
  [ -z "$v2" ] || { echo "FAIL V2.82: VAL-001 reports the validate-first twin: $v2"; exit 1; }
  v3="$(validation_order_check . src/cli/split.ts)"
  printf '%s\n' "$v3" | grep -qF 'NOT VERIFIED VAL-001: src/cli/split.ts:1:' \
    || { echo "FAIL V2.82: VAL-001 did not list the opening with no validator in its file: $v3"; exit 1; }

  # M-84, defect: a page loop with no read transaction; the twin wraps it in one.
  mkdir -p src/repo
  printf '%s\n' 'export const readAll = (db) => {' '  let page = 0; const out = [];' '  while (true) {' '    const rows = db.prepare("SELECT id FROM charges LIMIT ? OFFSET ?").all(50, page * 50);' '    if (rows.length === 0) break; out.push(...rows); page++;' '  }' '  return out;' '};' > src/repo/page.ts
  printf '%s\n' 'export const readAll = (db) => db.transaction(() => {' '  let page = 0; const out = [];' '  while (true) {' '    const rows = db.prepare("SELECT id FROM charges LIMIT ? OFFSET ?").all(50, page * 50);' '    if (rows.length === 0) break; out.push(...rows); page++;' '  }' '  return out;' '})();' > src/repo/page-tx.ts
  d1="$(read_transaction_check . src/repo/page.ts)"
  printf '%s\n' "$d1" | grep -qF 'FAIL DB-003: src/repo/page.ts:4:' \
    || { echo "FAIL V2.82: DB-003 did not report the page loop with no transaction: $d1"; exit 1; }
  d2="$(read_transaction_check . src/repo/page-tx.ts)"
  [ -z "$d2" ] || { echo "FAIL V2.82: DB-003 reports the transactional twin: $d2"; exit 1; }

  # M-54, defect: a retry budget no test holds; then a test that holds the dependency.
  mkdir -p src/net
  printf '%s\n' 'export const MAX = { maxAttempts: 3 };' 'export const run = () => MAX.maxAttempts;' > src/net/retry.ts
  printf '%s\n' 'import { run } from "../src/other";' 'test("other", () => {});' > tests/other.test.ts
  b1="$(bound_test_check . src/net/retry.ts -- tests/other.test.ts)"
  printf '%s\n' "$b1" | grep -qF 'FAIL BOUND-001: src/net/retry.ts:1: a bound is declared, and no test imports retry' \
    || { echo "FAIL V2.82: BOUND-001 did not report the retry budget with no holding test: $b1"; exit 1; }
  printf '%s\n' 'import { run } from "../src/net/retry";' 'test("retries until the budget is spent", () => { new Promise(() => {}); });' > tests/retry.test.ts
  b2="$(bound_test_check . src/net/retry.ts -- tests/retry.test.ts)"
  [ -z "$b2" ] || { echo "FAIL V2.82: BOUND-001 reports the holding twin: $b2"; exit 1; }

  # M-80, the verdict matrix: an unnumbered matrix is not held to 14 rows (its own rule
  # says only a numbered matrix must number every dimension); a numbered matrix of 3 rows is.
  . ./rep.sh || { echo "FAIL V2.82: the REPORT-001 block does not load"; exit 1; }
  printf '%s\n' '| Dimension | Status | Reason |' '|---|---|---|' '| Structure | PASS | the checks ran with exit 0 |' '| Behavior | PASS | the fixtures ran |' > unnumbered.md
  r1="$(report_matrix_check . unnumbered.md)"
  [ -z "$r1" ] || { echo "FAIL V2.82: REPORT-001 holds an unnumbered matrix to 14 rows: $r1"; exit 1; }
  printf '%s\n' '| Dimension | Status | Reason |' '|---|---|---|' '| 1 | PASS | ok |' '| 2 | PASS | ok |' '| 3 | PASS | ok |' > numbered.md
  r2="$(report_matrix_check . numbered.md)"
  printf '%s\n' "$r2" | grep -qF 'the numbered dimension matrix has 3 rows' \
    || { echo "FAIL V2.82: REPORT-001 did not hold a numbered 3-row matrix to 14: $r2"; exit 1; }
  exit 0
) || failures=1

# Removal probe: a copy of the sweep with the ERR-002 call taken out must no longer run it.
sed '/^  error_outcome_check \. \$src -- \$tests -- \$contracts$/d' "$root/$oc" > "$fx/sweep-removed.md"
grep -qF 'error_outcome_check . $src -- $tests -- $contracts' "$fx/sweep-removed.md" \
  && { echo "FAIL V2.82: the removal probe did not remove the ERR-002 call"; failures=1; }

echo "V2.82 probe: fixtures in $fx were checked and are removed"
rm -rf "$fx"

exit "$failures"
```

## V2.83 Regression — Commit Gate, Pre-existing Findings, Consumer Groups, Citations, Index Coverage, Permission Grants, Commit Scope, Performance Metrics, and Teardown

Verify that the holes an external audit exposed are closed on every surface, and that
each detector reports its defect with `file:line` and passes its compliant twin:

- M-08: the structural compliance check is a blocking gate of the commit workflow. The
  commit proposal runs it in Phase 2 and blocks in Phase 13 on its `FAIL`, and
  project validation states that a `FAIL` blocks the gate that runs it (R-144). The
  TYPE-001 detection is run on a mixed fixture by the behavioral audit of
  `workflows/system-audit.md`, not by this script.
- M-47 and M-78: a pre-existing violation in a modified test file is printed as a
  `FINDING` and is not dropped; `TYPE-002` is an executable comparison of consumer
  sets; a citation in a decision note or contract document, in or out of a table, is
  checked (R-145).
- M-85: a query predicate is served by an index led by its column. `DB-004` reports a
  `WHERE` equality with no such index, and a `JOIN` statement is `NOT VERIFIED` (R-146).
- M-86: a permission grant the approved requirement does not name is a human gate.
  `AUTH-002` reports each grant added to an authorization file (R-147).
- M-87: a proposed commit carries only the commits of the approved scope. `COMMIT-002`
  reports each foreign commit with its path (R-148).
- M-88: a performance criterion is `PASS` only against a metric and a threshold, and
  alignment asks for both before implementation (R-149). The alignment gate is a
  conversational procedure; this check verifies its wording, not its execution.
- M-89: a teardown the requirement asks for is listed in the phase plan with its target
  path and authorization status (R-150).
- M-80: a red gate attributed `BASELINE_FAILURE` states the count of its failing files, so
  the attribution is not a bare word (R-137). The plan is a conversational artifact; this
  check verifies its wording.

The static part reads the repository. The behavioral part extracts the `sh` blocks of the
detectors from `workflows/output-compliance.md`, runs them on fixtures in a temporary
directory outside the repository, which is removed, and uses a git fixture for `AUTH-002`
and `COMMIT-002`. It needs `awk`, `grep`, `sed`, `tr`, `perl`, `git`, and `mktemp`; without
them the behavioral part is `NOT VERIFIED`, never passed.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
ap="references/audit-pass-criteria.md"
db="skills/database-engineering/SKILL.md"
cm="workflows/commit.md"
pv="workflows/project-validation.md"
tc="workflows/task-to-code.md"
vs="workflows/system-validation.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.83: $1 is missing $3"; failures=1; }
}

# The canonical wording of M-85 to M-89 is in AGENTS.md, and every required surface carries its anchor.
need "$ag" "is served by an index whose leading column is the predicate's column" "the M-85 canonical wording"
need "$ag" "a permission, role, scope, or access rule that the approved requirement does not name" "the M-86 canonical wording"
need "$ag" "carries only the commits of the approved scope" "the M-87 canonical wording"
need "$ag" "a measurable metric and a threshold" "the M-88 canonical wording"
need "$ag" "is listed in the phase plan with its target path" "the M-89 canonical wording"
need "$db" "is served by an index whose leading column is the predicate's column" "the M-85 database instruction"
need "$ap" "is served by an index whose leading column is the predicate's column" "the M-85 criterion of Dimension 5"
need "$ap" "a permission, role, scope, or access rule that the approved requirement does not name" "the M-86 criterion of Dimension 6"
need "$ap" "a measurable metric and a threshold" "the M-88 criterion of Dimension 8"
need "$oc" "is served by an index whose leading column is the predicate's column" "the M-85 detector statement"
need "$oc" "a permission, role, scope, or access rule that the approved requirement does not name" "the M-86 detector statement"
need "$oc" "carries only the commits of the approved scope" "the M-87 detector statement"
need "$oc" "is printed as \`finding\`" "the M-47 FINDING wording of TEST-001 and TEST-002"
need "$oc" "a citation outside a table" "the M-78 scope wording of DEC-002"
need "$tc" "a measurable metric and a threshold" "the M-88 alignment step"
need "$tc" "is listed in the phase plan with its target path" "the M-89 plan step"
need "$tc" "a permission, role, scope, or access rule that the approved requirement does not name" "the M-86 alignment step"
need "$cm" "structural compliance is a blocking gate of this workflow" "the M-08 commit gate"
need "$cm" "carries only the commits of the approved scope" "the M-87 commit instruction"
need "$pv" "a \`FAIL\` of that check blocks every gate that runs it" "the M-08 blocking wording of project validation"

# Registry rows M-85 to M-89 and the M-08 surface; regression rows R-144 to R-150.
for id in M-85 M-86 M-87 M-88 M-89; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.83: no $id row in the registry"; failures=1; }
done
grep "^| M-08 |" "$root/$mr" | grep -qF 'workflows/commit.md' || { echo "FAIL V2.83: the M-08 row does not list workflows/commit.md as a surface"; failures=1; }
for id in R-144 R-145 R-146 R-147 R-148 R-149 R-150; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.83: no $id row in the regression matrix"; failures=1; }
done

# Coverage rows and sections of the new and changed detectors, and the sweep call of DB-004.
for id in DB-004 AUTH-002 COMMIT-002; do
  grep -qF "| \`$id\` |" "$root/$oc" || { echo "FAIL V2.83: the coverage table has no $id row"; failures=1; }
  grep -q "^### $id " "$root/$oc" || { echo "FAIL V2.83: $oc has no section for $id"; failures=1; }
done
grep -qF "| \`TYPE-002\` | contract files named by \`TYPE-001\`; \`consumer_group_check\`" "$root/$oc" \
  || { echo "FAIL V2.83: the coverage table does not name consumer_group_check for TYPE-002"; failures=1; }
grep -qF -- "  index_coverage_check . \$src \$migrations" "$root/$oc" \
  || { echo "FAIL V2.83: the whole-project sweep does not run index_coverage_check"; failures=1; }

# Removal probe: a copy of the sweep without the DB-004 call must no longer run it.
fx0="$(mktemp -d "${TMPDIR:-/tmp}/v283.XXXXXX")"
sed '/^  index_coverage_check \. \$src \$migrations$/d' "$root/$oc" > "$fx0/sweep-removed.md"
grep -qF -- "  index_coverage_check . \$src \$migrations" "$fx0/sweep-removed.md" \
  && { echo "FAIL V2.83: the removal probe did not remove the DB-004 call"; failures=1; }
rm -rf "$fx0"

command -v awk >/dev/null 2>&1 && command -v grep >/dev/null 2>&1 && command -v sed >/dev/null 2>&1 \
  && command -v tr >/dev/null 2>&1 && command -v mktemp >/dev/null 2>&1 \
  && command -v perl >/dev/null 2>&1 && command -v git >/dev/null 2>&1 || {
  echo "NOT VERIFIED V2.83: awk, grep, sed, tr, mktemp, perl, or git is not available, the behavioral fixtures were not run"; exit 2; }

# Extracts the sh block of one section of workflows/output-compliance.md. $1 = the rule identifier.
extract_sec() {
  awk -v h="$1" '
    $0 ~ ("^### " h " ") { on = 1; next }
    /^##/ { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; next }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v283.XXXXXX")"
(
  cd "$fx" || exit 1
  extract_sec DEC-002 > dec2.sh
  extract_sec TYPE-002 > type2.sh
  extract_sec DB-004 > db4.sh
  extract_sec AUTH-002 > auth.sh
  extract_sec COMMIT-002 > commit2.sh
  awk '/^test_api_check\(\) \{/{on=1} on{print} on && /^\}$/{exit}' "$root/$oc" > testapi.sh
  . ./dec2.sh && . ./type2.sh && . ./db4.sh && . ./auth.sh && . ./commit2.sh && . ./testapi.sh \
    || { echo "FAIL V2.83: the detector blocks do not load"; exit 1; }

  # M-47, defect: a modified test file whose it( case exists in the base revision is a FINDING,
  # and an it( case added by the change is a FAIL. The clean file, with only test(), prints nothing.
  mkdir -p tests
  printf '%s\n' 'import { test } from "vitest";' 'it("old case", () => {});' 'it("new case", () => {});' > tests/mod.test.ts
  printf '%s\n' 3 > added.txt
  t1="$(test_api_check tests/mod.test.ts added.txt)"; t1s=$?
  printf '%s\n' "$t1" | grep -qF 'FINDING TEST-002: tests/mod.test.ts:2:' \
    || { echo "FAIL V2.83: TEST-002 did not report the pre-existing it( as a FINDING: $t1"; exit 1; }
  printf '%s\n' "$t1" | grep -qF 'FAIL TEST-002: tests/mod.test.ts:3:' \
    || { echo "FAIL V2.83: TEST-002 did not report the added it( as a FAIL: $t1"; exit 1; }
  [ "$t1s" -ne 0 ] || { echo "FAIL V2.83: TEST-002 passed a file with an added it("; exit 1; }
  printf '%s\n' 'import { test } from "vitest";' 'it("old case", () => {});' > tests/pre.test.ts
  printf '%s\n' 9 > added2.txt
  t2="$(test_api_check tests/pre.test.ts added2.txt)"; t2s=$?
  printf '%s\n' "$t2" | grep -qF 'FINDING TEST-002: tests/pre.test.ts:2:' \
    || { echo "FAIL V2.83: TEST-002 dropped a pre-existing violation: $t2"; exit 1; }
  [ "$t2s" -eq 0 ] || { echo "FAIL V2.83: a pre-existing FINDING blocks the task: status $t2s"; exit 1; }
  printf '%s\n' 'import { test } from "vitest";' 'test("old case", () => {});' 'test("new case", () => {});' > tests/clean.test.ts
  printf '%s\n' 3 > added3.txt
  t3="$(test_api_check tests/clean.test.ts added3.txt)"
  [ -z "$t3" ] || { echo "FAIL V2.83: TEST-002 reports the test() twin: $t3"; exit 1; }

  # TYPE-002, defect: a contract whose types have different consumers; the twin whose types share one consumer set.
  mkdir -p src/contracts src/a src/b
  printf '%s\n' 'export interface Alpha { id: string }' 'export interface Beta { id: number }' > src/contracts/mixed.ts
  printf '%s\n' 'import { Alpha } from "../contracts/mixed";' > src/a/comp.ts
  printf '%s\n' 'import { Beta } from "../contracts/mixed";' > src/b/svc.ts
  y1="$(consumer_group_check . src/contracts/mixed.ts)"
  printf '%s\n' "$y1" | grep -qF 'FAIL TYPE-002: src/contracts/mixed.ts:2: Beta is consumed by a different set of files' \
    || { echo "FAIL V2.83: TYPE-002 did not report the type with different consumers: $y1"; exit 1; }
  printf '%s\n' 'import { Alpha, Beta } from "../contracts/mixed";' > src/a/comp.ts
  rm -f src/b/svc.ts
  y2="$(consumer_group_check . src/contracts/mixed.ts)"
  [ -z "$y2" ] || { echo "FAIL V2.83: TYPE-002 reports the shared-consumer twin: $y2"; exit 1; }
  mkdir -p nosrc
  printf '%s\n' 'export interface Gamma { id: string }' > nosrc/lib.ts
  y3="$(consumer_group_check "$fx/nosrc" lib.ts)"
  printf '%s\n' "$y3" | grep -qF 'NOT VERIFIED TYPE-002: lib.ts: there is no src/ folder' \
    || { echo "FAIL V2.83: TYPE-002 passes a project with no src/ folder: $y3"; exit 1; }

  # DEC-002, defect: a citation in prose that points 6 lines away from the identifier; the twin cites the line.
  mkdir -p docs/decisions
  printf '%s\n' 'export function a() {}' 'const x = 1;' 'export function invalidate() {}' 'const y = 2;' \
    'const z = 3;' 'const w = 4;' 'const v = 5;' 'const u = 6;' 'const t = 7;' 'const s = 8;' > src/store.ts
  printf '%s\n' 'The cache is cleared by `invalidate` at src/store.ts:9.' > docs/decisions/d1.md
  printf '%s\n' 'The cache is cleared by `invalidate` at src/store.ts:3.' > docs/decisions/d2.md
  c1="$(decision_citation_check . docs/decisions/d1.md)"
  printf '%s\n' "$c1" | grep -qF 'FAIL DEC-002: docs/decisions/d1.md:1: src/store.ts:9 does not show invalidate within 5 lines' \
    || { echo "FAIL V2.83: DEC-002 did not report the stale citation in prose: $c1"; exit 1; }
  c2="$(decision_citation_check . docs/decisions/d2.md)"
  [ -z "$c2" ] || { echo "FAIL V2.83: DEC-002 reports the twin that cites the line: $c2"; exit 1; }

  # DB-004, defect: a WHERE equality on user_id with no index led by it; the twin adds the index;
  # a JOIN statement is NOT VERIFIED rather than passed.
  mkdir -p migrations
  printf '%s\n' 'CREATE TABLE saved_methods (id INTEGER PRIMARY KEY, user_id TEXT NOT NULL, label TEXT);' \
    'CREATE INDEX idx_label ON saved_methods (label);' > migrations/001.sql
  mkdir -p src/db
  printf '%s\n' 'export const list = (db, u) => db.prepare("SELECT id, label FROM saved_methods WHERE user_id = ? ORDER BY id").all(u);' > src/db/store.ts
  g1="$(index_coverage_check . src/db/store.ts migrations/001.sql)"
  printf '%s\n' "$g1" | grep -qF 'FAIL DB-004: src/db/store.ts:1: user_id = ? on table saved_methods has no index' \
    || { echo "FAIL V2.83: DB-004 did not report the predicate with no index: $g1"; exit 1; }
  printf '%s\n' 'CREATE INDEX idx_user ON saved_methods (user_id, label);' >> migrations/001.sql
  g2="$(index_coverage_check . src/db/store.ts migrations/001.sql)"
  [ -z "$g2" ] || { echo "FAIL V2.83: DB-004 reports the twin with the index: $g2"; exit 1; }
  printf '%s\n' 'export const join = (db) => db.prepare("SELECT a.id FROM saved_methods a JOIN users b ON b.id = a.user_id WHERE a.label = ?").all();' > src/db/join.ts
  g3="$(index_coverage_check . src/db/join.ts migrations/001.sql)"
  printf '%s\n' "$g3" | grep -qF 'NOT VERIFIED DB-004: src/db/join.ts:1: label = ? is in a statement with a JOIN' \
    || { echo "FAIL V2.83: DB-004 passed or failed a JOIN statement instead of NOT VERIFIED: $g3"; exit 1; }

  # AUTH-002, defect: a permission grant added to an authorization file; the twin adds none.
  rm -rf repo && mkdir repo && cd repo || exit 1
  git init -q && mkdir -p src/auth
  printf '%s\n' 'export const policy = {' '  admin: ["read", "write"],' '};' > src/auth/authorization-policy.ts
  git add -A && git -c user.email=v@v -c user.name=v commit -qm base
  printf '%s\n' 'export const policy = {' '  admin: ["read", "write"],' '  viewer: ["read"],' '};' > src/auth/authorization-policy.ts
  a1="$(authorization_grant_check HEAD src/auth/authorization-policy.ts)"
  printf '%s\n' "$a1" | grep -qF 'FAIL AUTH-002: src/auth/authorization-policy.ts:3: an added permission grant' \
    || { echo "FAIL V2.83: AUTH-002 did not report the added grant: $a1"; exit 1; }
  git checkout -q -- src/auth/authorization-policy.ts
  a2="$(authorization_grant_check HEAD src/auth/authorization-policy.ts)"
  [ -z "$a2" ] || { echo "FAIL V2.83: AUTH-002 reports the twin with no grant: $a2"; exit 1; }

  # COMMIT-002, defect: a foreign commit on top of the base; the twin names its path in the approved scope.
  cd "$fx" && rm -rf repo2 && mkdir repo2 && cd repo2 || exit 1
  git init -q && mkdir -p src docs
  commit_as() { git -c user.email=v@v -c user.name=v commit -qm "$1"; }
  echo base > docs/base.md && git add -A && commit_as base
  base_rev="$(git rev-parse HEAD)"
  echo a > src/a.ts && git add -A && commit_as "in scope"
  echo o > src/other.ts && git add -A && commit_as "foreign task"
  printf '%s\n' src/a.ts > manifest.txt
  k1="$(out_of_scope_commit_check "$base_rev" manifest.txt)"
  printf '%s\n' "$k1" | grep -qF 'FAIL COMMIT-002:' && printf '%s\n' "$k1" | grep -qF 'touches src/other.ts' \
    || { echo "FAIL V2.83: COMMIT-002 did not report the foreign commit with its path: $k1"; exit 1; }
  [ "$(printf '%s\n' "$k1" | grep -c 'FAIL COMMIT-002')" = 1 ] \
    || { echo "FAIL V2.83: COMMIT-002 reported more than the one foreign commit: $k1"; exit 1; }
  printf '%s\n' src/a.ts src/other.ts > manifest2.txt
  k2="$(out_of_scope_commit_check "$base_rev" manifest2.txt)"
  [ -z "$k2" ] || { echo "FAIL V2.83: COMMIT-002 reports the twin whose scope names the path: $k2"; exit 1; }
  cd "$fx" || exit 1

  # REPORT-001, defect: a red gate attributed BASELINE_FAILURE with no count of the failing files; the twin states the count.
  extract_sec REPORT-001 > rep.sh && . ./rep.sh || { echo "FAIL V2.83: the REPORT-001 block does not load"; exit 1; }
  printf '%s\n' '# Gate' 'npm run format:check fails: BASELINE_FAILURE, outside the change set.' > rep1.md
  p1="$(report_matrix_check . rep1.md)"
  printf '%s\n' "$p1" | grep -qF 'FAIL REPORT-001: rep1.md:2: a gate attributed BASELINE_FAILURE states no count' \
    || { echo "FAIL V2.83: REPORT-001 did not report the BASELINE_FAILURE with no count: $p1"; exit 1; }
  printf '%s\n' '# Gate' 'npm run format:check fails: 17 files, BASELINE_FAILURE, first at src/a.ts:1, outside the change set.' > rep2.md
  p2="$(report_matrix_check . rep2.md)"
  printf '%s\n' "$p2" | grep -q 'states no count' && { echo "FAIL V2.83: REPORT-001 reports the counted twin: $p2"; exit 1; }
  exit 0
) || failures=1

echo "V2.83 probe: fixtures in $fx were checked and are removed"
rm -rf "$fx"

exit "$failures"
```

## V2.84 Regression — Scope Manifest, Analysis-Only Requests, Approval Quotes, Teardown Gate, and Verdict Statuses

Verify that the holes the external audit exposed are closed on every surface, and that
each detector reports its defect with `file:line` and passes its compliant twin:

- M-90: the approved scope manifest is recorded at alignment before the first file write,
  and `SCOPE-001` reads tracked and untracked files and fails a change set with no
  recorded manifest (R-151).
- M-91: an analysis, decision, or estimate request authorizes no file change in any
  project, including a scratch project; its empty manifest makes any changed file a
  `FAIL` (R-152).
- M-92: a claimed human approval quotes the verbatim user message and its turn;
  `APPROVAL-001` reports an uncited claim as `NOT VERIFIED` (R-153).
- M-93: a teardown recorded `NOT DONE` in the checkpoint blocks the commit proposal;
  `TEARDOWN-001` reports the open step (R-154).
- M-80: a verdict-matrix row with a qualified status or an unenumerated status is a
  `FAIL` of `REPORT-001`, never skipped (R-155).

The static part reads the repository. The behavioral part extracts the `sh` blocks of
`SCOPE-001`, `REPORT-001`, `APPROVAL-001`, and `TEARDOWN-001` from
`workflows/output-compliance.md`, runs them on fixtures in a temporary directory outside
the repository, which is removed, and uses a git fixture for `SCOPE-001`. It needs `awk`,
`grep`, `sed`, `tr`, `git`, and `mktemp`; without them the behavioral part is
`NOT VERIFIED`, never passed.

```sh
root="$(cd "${1:-.}" && pwd)"
failures=0
ag="AGENTS.md"
mr="references/mandatory-rules.md"
rmx="references/system-workflow-regression-matrix.md"
oc="workflows/output-compliance.md"
tc="workflows/task-to-code.md"
cm="workflows/commit.md"
sr="workflows/session-resilience.md"

flat() { tr '\n' ' ' < "$root/$1" | tr -s ' ' | tr 'A-Z' 'a-z'; }

need() {
  needle="$(printf '%s' "$2" | tr 'A-Z' 'a-z')"
  grep -qF -- "$needle" <<< "$(flat "$1")" || { echo "FAIL V2.84: $1 is missing $3"; failures=1; }
}

# The canonical wording of M-90 to M-93 is in AGENTS.md, and each listed surface carries its anchor.
need "$ag" "is recorded at alignment before the first file write" "the M-90 canonical wording"
need "$ag" "authorizes no file change in any project" "the M-91 canonical wording"
need "$ag" "quotes the verbatim user message and its turn" "the M-92 canonical wording"
need "$ag" "blocks the commit proposal until the user resolves it or accepts it in writing" "the M-93 canonical wording"
need "$tc" "is recorded at alignment before the first file write" "the M-90 alignment step"
need "$tc" "authorizes no file change in any project" "the M-91 alignment step"
need "$tc" "quotes the verbatim user message and its turn" "the M-92 alignment step"
need "$tc" "blocks the commit proposal until the user resolves it or accepts it in writing" "the M-93 alignment step"
need "$cm" "is recorded at alignment before the first file write" "the M-90 commit gate"
need "$cm" "blocks the commit proposal until the user resolves it or accepts it in writing" "the M-93 commit gate"
need "$oc" "is recorded at alignment before the first file write" "the M-90 detector statement"
need "$oc" "blocks the commit proposal until the user resolves it or accepts it in writing" "the M-93 detector statement"
need "$sr" "is recorded at alignment before the first file write" "the M-90 checkpoint field"

# Registry rows M-90 to M-93 and regression rows R-151 to R-155.
for id in M-90 M-91 M-92 M-93; do
  grep -q "^| $id |" "$root/$mr" || { echo "FAIL V2.84: no $id row in the registry"; failures=1; }
done
for id in R-151 R-152 R-153 R-154 R-155; do
  grep -q "^| $id |" "$root/$rmx" || { echo "FAIL V2.84: no $id row in the regression matrix"; failures=1; }
done

# Coverage rows of the new detectors, and the changed SCOPE-001 row.
for rule in APPROVAL-001 TEARDOWN-001; do
  grep -qF "| \`$rule\` |" "$root/$oc" || { echo "FAIL V2.84: the coverage table has no $rule row"; failures=1; }
done
grep -qF 'tracked and untracked' "$root/$oc" || { echo "FAIL V2.84: the SCOPE-001 coverage row does not name untracked files"; failures=1; }

# Behavioral part: the detectors run on fixtures in a temporary directory outside the repository.
for cmd in git awk grep sed tr mktemp; do
  command -v "$cmd" >/dev/null 2>&1 || { echo "NOT VERIFIED V2.84: $cmd is not available, the behavioral fixtures were not run"; exit 2; }
done

extract_sec() {
  awk -v h="$1" '
    $0 ~ ("^### " h " ") { on = 1; next }
    /^##/ { on = 0 }
    on && /^```sh$/ { inb = 1; next }
    inb && /^```$/ { inb = 0; next }
    inb { print }' "$root/$oc"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v284.XXXXXX")"
(
  cd "$fx" || exit 1
  extract_sec SCOPE-001 > scope.sh && . ./scope.sh || { echo "FAIL V2.84: the SCOPE-001 block does not load"; exit 1; }
  extract_sec REPORT-001 > report.sh && . ./report.sh || { echo "FAIL V2.84: the REPORT-001 block does not load"; exit 1; }
  extract_sec APPROVAL-001 > approval.sh && . ./approval.sh || { echo "FAIL V2.84: the APPROVAL-001 block does not load"; exit 1; }
  extract_sec TEARDOWN-001 > teardown.sh && . ./teardown.sh || { echo "FAIL V2.84: the TEARDOWN-001 block does not load"; exit 1; }

  # SCOPE-001, defect: an untracked migration the manifest does not name; the tracked change is named.
  git init -q repo && cd repo || exit 1
  git config user.email v@v && git config user.name v
  mkdir -p src migrations
  printf '%s\n' 'export const a = 1;' > src/a.ts
  printf '%s\n' 'export const b = 1;' > src/b.ts
  git add -A && git commit -qm base
  printf '%s\n' 'export const a = 2;' > src/a.ts
  printf '%s\n' 'CREATE TABLE t (id INT);' > migrations/004.sql
  printf '%s\n' src/a.ts > ../m1.txt
  printf '%s\n' src/a.ts migrations/004.sql > ../m2.txt
  : > ../m0.txt
  s1="$(scope_file_check . ../m1.txt)"
  printf '%s\n' "$s1" | grep -qF 'FAIL SCOPE-001: migrations/004.sql:1:' \
    || { echo "FAIL V2.84: SCOPE-001 did not report the untracked unnamed file: $s1"; exit 1; }
  printf '%s\n' "$s1" | grep -q 'src/a.ts' && { echo "FAIL V2.84: SCOPE-001 reports the file the manifest names: $s1"; exit 1; }
  s2="$(scope_file_check . ../nomanifest.txt)"
  printf '%s\n' "$s2" | grep -qF 'no approved scope manifest was recorded at alignment' \
    || { echo "FAIL V2.84: SCOPE-001 passed a change set with no manifest: $s2"; exit 1; }
  s3="$(scope_file_check . ../m0.txt)"
  printf '%s\n' "$s3" | grep -qF 'FAIL SCOPE-001: src/a.ts:1:' && printf '%s\n' "$s3" | grep -qF 'FAIL SCOPE-001: migrations/004.sql:1:' \
    || { echo "FAIL V2.84: an empty manifest did not fail every changed file (M-91): $s3"; exit 1; }
  s4="$(scope_file_check . ../m2.txt)"
  [ -z "$s4" ] || { echo "FAIL V2.84: SCOPE-001 reports the compliant twin: $s4"; exit 1; }
  cd "$fx" || exit 1

  # REPORT-001, defect: a qualified PASS and a free-text status; the twin has fourteen enumerated verdicts.
  { echo '# Audit'; echo '| Dim. | Estado | Motivo |'; echo '|---|---|---|'
    echo '| 2. Patrones | PASS | record con 0 patrones |'
    echo '| 3. Calidad | PASS, con observación | modificación fuera del alcance |'
    echo '| 1, 4, 6, 8–14 | No ejecutadas en esta pasada | límite declarado |'; } > rep1.md
  r1="$(report_matrix_check . rep1.md)"
  printf '%s\n' "$r1" | grep -qF 'FAIL REPORT-001: rep1.md:5: PASS carries a qualifier' \
    || { echo "FAIL V2.84: REPORT-001 did not report the qualified PASS: $r1"; exit 1; }
  printf '%s\n' "$r1" | grep -qF 'FAIL REPORT-001: rep1.md:6: unrecognized status' \
    || { echo "FAIL V2.84: REPORT-001 skipped the grouped free-text status: $r1"; exit 1; }
  { echo '# Audit'; echo '| Dim. | Estado | Motivo |'; echo '|---|---|---|'
    for i in 1 2 3 4 5 6 7 8 9 10 11 12 13 14; do echo "| $i. Dimensión $i | PASS | evidencia del criterio $i |"; done; } > rep2.md
  r2="$(report_matrix_check . rep2.md)"
  [ -z "$r2" ] || { echo "FAIL V2.84: REPORT-001 reports the compliant twin: $r2"; exit 1; }

  # APPROVAL-001, defect: an approval with no quoted message; the twin quotes it with its turn.
  printf '%s\n' '# Decision' 'Aprobado por el dueño del producto para este alcance.' > ap1.md
  a1="$(approval_citation_check . ap1.md)"
  printf '%s\n' "$a1" | grep -qF 'NOT VERIFIED APPROVAL-001: ap1.md:2:' \
    || { echo "FAIL V2.84: APPROVAL-001 did not report the uncited approval: $a1"; exit 1; }
  printf '%s\n' '# Decision' 'Aprobado por el dueño del producto. Turno 4 del usuario: "sí, adelante con el alcance de la migración".' > ap2.md
  a2="$(approval_citation_check . ap2.md)"
  [ -z "$a2" ] || { echo "FAIL V2.84: APPROVAL-001 reports the quoted twin: $a2"; exit 1; }

  # TEARDOWN-001, defect: an open NOT DONE step; the twin records it as accepted in writing.
  printf '%s\n' '# Checkpoint' '- Teardown scratch changes in demo-backend: NOT DONE, no authorization for that path.' > cp1.md
  t1="$(teardown_gate_check cp1.md)"
  printf '%s\n' "$t1" | grep -qF 'FAIL TEARDOWN-001: cp1.md:2:' \
    || { echo "FAIL V2.84: TEARDOWN-001 did not report the open NOT DONE step: $t1"; exit 1; }
  printf '%s\n' '# Checkpoint' '- Teardown scratch changes: NOT DONE, accepted in writing: "déjalos sin commit".' > cp2.md
  t2="$(teardown_gate_check cp2.md)"
  [ -z "$t2" ] || { echo "FAIL V2.84: TEARDOWN-001 reports the accepted twin: $t2"; exit 1; }
  t3="$(teardown_gate_check missing.md)"
  printf '%s\n' "$t3" | grep -qF 'NOT VERIFIED TEARDOWN-001: missing.md' \
    || { echo "FAIL V2.84: TEARDOWN-001 did not report a missing checkpoint as NOT VERIFIED: $t3"; exit 1; }
  exit 0
) || failures=1

echo "V2.84 probe: fixtures in $fx were checked and are removed"
rm -rf "$fx"

exit "$failures"
```

## V2.85 Regression — Native Task Mirror at Stage Level

Verify that the native task mirror follows the stage (subfase) level as well as
the phase level, so the panel shows where in the phase the work is:

- `AGENTS.md` states that the checklist and its mirror are re-emitted at the
  start and at the end of each stage, and that the rewrite carries the stages of
  each phase with their state and the steps of the stage in progress.
- `workflows/session-resilience.md` carries the stage rows of the cadence table,
  the `Stage:` marker that tells a stage line from a step line, and the rule that
  a rewrite never drops a phase or a stage.
- `workflows/task-to-code.md`, `agents/orchestrator.md`, and `commands/task.md`
  name the stage start and end in their operational instruction.
- `references/mandatory-rules.md` carries the stage transitions in M-35 and
  `references/system-workflow-regression-matrix.md` includes R-156.
- Proof: the panel check below accepts a compliant two-stage panel and rejects a
  phase-only panel, a panel that keeps only the last stage, a stage-less step
  mixed into a staged phase, and a stage in progress with no steps, each with the
  `file:line` of the defect. The check verifies the panel's content only; that the
  rewrite happens at each transition is a limit of the client surface, reported
  in the audit, not proven by this block.

```sh
failures=0
has() { tr '\n' ' ' < "$1" | tr -s ' ' | grep -qF "$2"; }
has AGENTS.md "the start and the end of each stage (subfase)" || { echo "FAIL V2.85: AGENTS.md has no stage transition"; failures=1; }
has AGENTS.md "the stages of the phase with their state" || { echo "FAIL V2.85: AGENTS.md has no stage state in the rewrite"; failures=1; }
has workflows/session-resilience.md "A stage (subfase) starts" || { echo "FAIL V2.85: no stage start row in the cadence table"; failures=1; }
has workflows/session-resilience.md "A stage (subfase) ends" || { echo "FAIL V2.85: no stage end row in the cadence table"; failures=1; }
has workflows/session-resilience.md "the \`Stage:\` word marks the line as a stage" || { echo "FAIL V2.85: no Stage: marker rule"; failures=1; }
has workflows/session-resilience.md "step never uses a phase label" || { echo "FAIL V2.85: no ban on phase labels as step prefixes"; failures=1; }
has workflows/task-to-code.md "the start and the end of each stage" || { echo "FAIL V2.85: no stage transition in task-to-code.md"; failures=1; }
has agents/orchestrator.md "the start and the end of each stage" || { echo "FAIL V2.85: no stage transition in orchestrator.md"; failures=1; }
has commands/task.md "the start and the end of each stage" || { echo "FAIL V2.85: no stage transition in commands/task.md"; failures=1; }
grep -q "^| M-35 |.*the start and the end of each stage" references/mandatory-rules.md || { echo "FAIL V2.85: M-35 does not carry the stage transition"; failures=1; }
grep -q "^| R-156 |" references/system-workflow-regression-matrix.md || { echo "FAIL V2.85: no R-156"; failures=1; }

# Panel check. One item per line: "<status> <content>". A phase is "N. name", a
# stage is "N.S Stage: name", a step of a stage is "N.S.K artifact", and a step
# of a phase without stages is "N.K artifact". Prints FAIL lines with file:line.
panel_check() {
  awk '
  function fail(msg) { print "FAIL V2.85: " FILENAME ":" NR ": " msg; bad = 1 }
  {
    st = $1
    c = $0
    sub(/^[^ ]+ /, "", c)
    if (st != "pending" && st != "in_progress" && st != "completed") fail("unknown status " st)
    if (c ~ /^[0-9]+\. /) {
      p = c; sub(/\..*/, "", p)
      phase[p] = 1; pstat[p] = st; plines[p] = NR
      next
    }
    if (c ~ /^[0-9]+\.[0-9]+ Stage: /) {
      split(c, a, " "); split(a[1], n, "."); p = n[1]; s = n[2]
      stage[p "." s] = 1; sstat[p "." s] = st; sline[p "." s] = NR
      hasStage[p] = 1; haschild[p] = 1
      if (s + 0 > maxs[p] + 0) maxs[p] = s + 0
      next
    }
    if (c ~ /^[0-9]+\.[0-9]+\.[0-9]+ /) {
      split(c, a, " "); split(a[1], n, ".")
      p = n[1]; s = n[2]
      if (!((p "." s) in stage)) fail("step " a[1] " has no stage line " p "." s)
      haschild[p "." s] = 1; haschild[p] = 1
      next
    }
    if (c ~ /^[0-9]+\.[0-9]+ /) {
      split(c, a, " "); split(a[1], n, ".")
      plain[n[1]] = 1; haschild[n[1]] = 1
      next
    }
    fail("item is neither a phase, a stage, nor a step: " c)
  }
  END {
    maxp = 0
    for (p in phase) if (p + 0 > maxp) maxp = p + 0
    for (i = 1; i <= maxp; i++) if (!(i in phase)) { print "FAIL V2.85: " FILENAME ": phase " i " is missing"; bad = 1 }
    for (p in phase) {
      if (pstat[p] == "in_progress" && !haschild[p]) { print "FAIL V2.85: " FILENAME ":" plines[p] ": phase " p " in progress with no stage or step"; bad = 1 }
      if (hasStage[p] && plain[p]) { print "FAIL V2.85: " FILENAME ": phase " p " mixes stage-less steps with stages"; bad = 1 }
    }
    for (p in maxs) for (i = 1; i <= maxs[p]; i++) if (!((p "." i) in stage)) { print "FAIL V2.85: " FILENAME ": missing stage " p "." i " of phase " p; bad = 1 }
    for (k in sstat) if (sstat[k] == "in_progress" && !haschild[k]) { print "FAIL V2.85: " FILENAME ":" sline[k] ": stage " k " in progress with no steps"; bad = 1 }
    exit bad
  }' "$1"
}

fx="$(mktemp -d "${TMPDIR:-/tmp}/v285.XXXXXX")"
printf '%s\n' \
  'completed 1. Detect and analyze the repository' \
  'completed 1.1 Inventory the fixture' \
  'completed 1.2 Locate the orders table definition' \
  'completed 2. Align requirements and approach' \
  'completed 2.1 Stage: partitioning rationale' \
  'completed 2.1.1 Strategy comparison for orders' \
  'in_progress 2.2 Stage: migration plan' \
  'in_progress 2.2.1 Online reversible migration steps' \
  'pending 2.2.2 Scope manifest for orders' \
  'in_progress 3. Implement schema and queries' \
  'pending 3.1 Stage: schema' \
  'pending 3.1.1 orders partition DDL' \
  'pending 4. Review and validate' \
  'pending 4.1 Run the format and lint gates' > "$fx/ok.txt"
grep -v '^completed 2\.1' "$fx/ok.txt" | grep -v '^pending 2\.2\.2' > "$fx/last-stage-only.txt"
grep -E '^[a-z_]+ [0-9]+\. ' "$fx/ok.txt" > "$fx/phase-only.txt"
sed 's/^pending 3\.1 Stage/in_progress 3.1 Stage/' "$fx/ok.txt" | grep -v '^pending 3\.1\.1' > "$fx/stage-no-steps.txt"
cp "$fx/ok.txt" "$fx/mixed.txt"; printf '%s\n' 'pending 3.2 Write the rollback notes' >> "$fx/mixed.txt"

(
  [ -z "$(panel_check "$fx/ok.txt")" ] || { echo "FAIL V2.85: the compliant panel was rejected: $(panel_check "$fx/ok.txt")"; exit 1; }
  p1="$(panel_check "$fx/phase-only.txt")"
  printf '%s\n' "$p1" | grep -qF 'phase 3 in progress with no stage or step' \
    || { echo "FAIL V2.85: a phase-only panel was not detected: $p1"; exit 1; }
  p2="$(panel_check "$fx/last-stage-only.txt")"
  printf '%s\n' "$p2" | grep -qF 'missing stage 2.1 of phase 2' \
    || { echo "FAIL V2.85: a panel without the first stage was not detected: $p2"; exit 1; }
  p3="$(panel_check "$fx/mixed.txt")"
  printf '%s\n' "$p3" | grep -qF 'phase 3 mixes stage-less steps with stages' \
    || { echo "FAIL V2.85: a stage-less step in a staged phase was not detected: $p3"; exit 1; }
  p4="$(panel_check "$fx/stage-no-steps.txt")"
  printf '%s\n' "$p4" | grep -qF 'stage 3.1 in progress with no steps' \
    || { echo "FAIL V2.85: a stage in progress without steps was not detected: $p4"; exit 1; }
  exit 0
) || failures=1

echo "V2.85 probe: fixtures in $fx were checked and are removed"
rm -rf "$fx"
exit "$failures"
```
