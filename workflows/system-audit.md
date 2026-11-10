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
| The linter configuration | A real project receives only its standard `eslint.config.*` and Prettier configuration: a compliant artifact passes, each injected structural violation is detected with `file:line` by the system's compliance check, and a project that adds a custom rule file fails `CONFIG-001` | A small service per affected ecosystem, configured from `references/linter-rules.md`, with the violation injected and no custom tooling added |
| The compliance checks | Each detection pattern finds the violation it claims to find, including the aliased and relative forms | An artifact that contains each violation in both forms |
| The project creation order | A real artifact per affected project type can be created following the order, passes the structural rules, and the checklist is completable | A service, a frontend, a library, and a mobile app, per the affected types |
| The startup decision catalog | Every project type in scope has its decisions, each with a default, a user-decision condition, and a source | The decision resolution applied to a synthetic scope |
| A workflow or a flow | The phases are ordered, every referenced file exists, and the flow is reachable from its command or from the orchestrator | A trace of the flow: the entry point, the phases in order, and the files each phase reads |
| A database change: schema, migration, startup statement, or deployment order (M-69, M-70) | The DDL detector `DB-001` finds no schema statement outside a migration; the deploy-order detector `DB-002` finds a test that names the legacy or previous schema for every migration; `CLAIM-001` marks each order-safety claim `NOT VERIFIED` until its test runs in both orders | A store that creates its index at open, its migration, and a migration with no legacy-schema test: each is detected with `file:line`, and the repaired store passes |
| A change set with unrelated files (M-71) | `SCOPE-001` finds every changed file the approved scope manifest does not name | A git fixture with one unnamed reformatted file and its manifest-named twin |
| A Skill | The description names the concrete triggers, the canonical source is referenced instead of restated, and the skill is reachable | The skill loaded in the situation its description claims to match |
| A command | It delegates to a workflow and copies no block from it | The command run against its workflow |
| An agent | It references the shared workflows and skills instead of restating them | The agent's task dispatch |
| A reference or a document | Every URL resolves and every referenced file exists | The document's referenced artifact |
| The token efficiency policy | The budget, the stop rules, and the measurement are defined, and the reachability check passes | A session-like workload with a known exploration profile |
| A secret or content scan, or the coverage of any control | Every excluded input is reported as `NOT VERIFIED`, never clean (M-51), and the procedure is wired into the security review and the delivery gate | A repository with a secret marker in an unlisted extension and in an oversize file, scanned with the documented exclusions: each excluded file is listed as `NOT VERIFIED` |
| An entry point that identifies its caller, or a project that enters the system (M-52, M-53) | An identity read from a client-controlled header, query string, or body is a finding; the entry-point sweep runs only when the project has an identity mechanism; the health probe reports a pre-existing failure, a tracked generated artifact, and a missing audit as findings without blocking an unrelated task | A route that reads `x-user-id` from the request and an unrelated project with a tracked `.tsbuildinfo` and no configured audit: the review reports the identity source as `FAIL` and the probe reports both artifacts as findings |
| A delivery-safety detector, or the whole-project sweep of the health probe (M-52, M-54 to M-61) | Each detector finds the violation in the form the external audit found it, passes its compliant twin, and reports `NOT VERIFIED` rather than clean for an input it did not read; the sweep covers every tracked and untracked file of the project and prints its scanned and excluded counts | A fixture repository with one violation per detector and its compliant twin, plus the sweep run over it, with each excluded input listed |
| A comment, a document claim about behavior, or a negative test probe | The statement is compared with the code it describes and a contradiction is cited with both lines; a negative probe is isolated from other rules and has a control probe | A comment that claims the opposite of the code it describes, and a probe placed under an ignored path next to one placed outside it |
| A duplicated contract declaration | `TYPE-004` reports each copy with `file:line`, and a name declared once or in one file is not reported | Two files that declare the same contract, one as an interface and one as a JSDoc typedef, and one file that declares a name once |
| A target version, or a stack's version support | The artifact is built for the **confirmed target version**, not for whatever version the environment happens to have | The toolchain reports the target version, or the check that could not run against it is reported as `NOT VERIFIED` with the reason |
| A framework or language convention | The conventions come from the official documentation of that framework, for the target version | The convention is checked against the official source, and the source is recorded |
| A delivery approval or a gate that consumes an artifact (M-114, M-116) | `DELIVERY-001` finds no approval without a digest comparison and no approver read from `github.actor`; `DELIVERY-002` finds no checkout of a dispatched ref in a file that deploys; the platform reviewers are `NOT VERIFIED` | A workflow with an approval that names no digest and a dispatcher approver, and its compliant twin |
| A claim that an agent or a prompt ran, a module with an effect at import, or an error or exit code with one test (M-115, M-117, M-118) | `EXEC-001` finds a claim that cites only a direct CLI call; `IMPORT-001` finds a column-zero call; `ERR-002` and `ENTRY-001` find a code emitted from several sites or an exit code no test asserts | A report with a direct-call claim, a module with a call at import, and an entry point with exit codes 0, 1, and 2 asserted only for 0 |
| A detector, a sweep, or a verdict PASS (M-75, M-98) | Every function the sweep calls is loaded by its probe, so no sweep line runs as "command not found" and silently checks nothing; every PASS row cites its evidence (`REPORT-001`); a mechanical check of a rule is run, not only its text | A fixture with one defect per detector, the sweep run over it with every function loaded, and a PASS row with no evidence next to its compliant twin |

If the adjustment does not fit any row, derive the check from the rule it
implements and state it in the report.

A check of a framework or language convention consults the **version guide** of
the ecosystem first, for the confirmed target version, and records the version
and the date consulted (AGENTS.md, Human Decision Gates).

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

Always executed after the implementation. The structural checks are owned by
`workflows/system-validation.md` §6: the executable checks §6.0 to §6.7 cover
reference integrity, the command catalog, portability, index and numbering,
rule reachability and cross-client reach, retired wordings, and duplication.
This workflow runs them and does not copy them. A change to a check is made in
`workflows/system-validation.md`.

The portability check assumes a **POSIX shell**, the shell the shared procedures
are written for. When the environment does not provide one, the check is
`NOT VERIFIED` with the reason, never a pass.

Every check reports `PASS`, `FAIL`, or `NOT VERIFIED` with the reason and the
evidence. A `FAIL` is a defect of the system's own artifact.

A reinforcement of an engineering discipline adopted from an external
methodology is audited with a behavioral scenario, as `AGENTS.md` requires for
M-42: the case the discipline must catch, the procedure run against it, and the
removal probe that proves the check detects a missing step.

The adjustment adds one check of its own:

### 3.1 Coverage (adjustment-specific)

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

- Build the artifact from the canonical configuration and references of the repository, not from
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
- **Verdict check**: the dimension matrix of this report is written to a file
  in the temporary directory and `REPORT-001` (`workflows/output-compliance.md`)
  runs over it before the report is delivered. A verdict matrix is consistent
  with its own rules (M-80): every dimension has its own row, no `PASS` carries
  an open item, no closure is claimed over an open row, and a red gate outside
  the change set is `BASELINE_FAILURE`. A `FAIL` is corrected in the report
  before it is delivered, and the correction is reported.
- **Claims of the text.** `EVIDENCE-001` and `APPROVAL-001`
  (`workflows/output-compliance.md`) run over the whole report text, not only over
  the matrix. A count of a set (files, lines) is checked against its cited list, a
  claim that a tool covers files is checked against the tool's own file resolution
  (`prettier --file-info`, `ruff check --show-files`), and a claimed time window
  cites its timestamp. A report that states "the formatter covered the 7 new files"
  without that resolution is `NOT VERIFIED` for that claim until it is cited (M-109,
  M-92).
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
