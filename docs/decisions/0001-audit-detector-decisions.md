# 0001. Detector scope and stated limits for M-132 and M-133

**Status:** proposed, pending the user's approval of the commit.
**Scope:** shell detectors in `workflows/output-compliance.md` and validation blocks in `workflows/system-validation.md`.

## Context

An external audit found gaps in the system's own detectors: a runtime API used under an older floor, a contract
not recorded as deviating, a decision record claiming an order the history did not show, an undocumented exit
status, a CSV unit not listed, a local type, an uncommitted IIFE generator without a stated reason, and an
undisclosed fixture path. A later audit of the detector claims found prose that overstated what two detectors
read: a shell operand and a `$` path for ARTIFACT-001, and the word `head`, a sentence-final period, and a closed
branch type list for GIT-001; it also found a GIT-001 run with no stated cost limit and a record number that did
not reproduce. Decisions 1 to 6 answer the first audit, and decisions 7 and 8 the second and third: the second found
prose that overstated what the two claim detectors read, and the third found that a URL hid a real citation, that a
URL path segment could still be claimed as a branch, and that two rule anchors were missing from their surfaces.
Restoring the anchors exposed a fourth copy of them: the V3.03 and V3.04 probes held their own copy, which had
drifted from the registry and failed the tree that V2.32 accepts. The probes now read the anchor from the registry
(decision 9).

## Decisions

1. **FLOOR-001 uses a table with an official source.** Rejected: inferring runtime APIs from the code. The table
   records only the version that adds each API. An API outside the table is a reviewer check, and a floor above the
   highest listed major is not judged by the table and is checked against the official documentation for that major
   (stated limit).
2. **DEC-003 checks that a Deviations section exists; DEC-005 checks that it names each field of a cited JSON contract
   present in the repository.** Rejected: comparing values, which needs a per-format parser. Values and code-only
   fields stay a reviewer decision (stated limit). The DEC-003 trigger is a word list, and the field scan skips keys
   that are JSON Schema keywords, so a data field named `type`, `title`, `format`, `default`, or `description` is not
   checked (stated limit).
3. **DEC-004 checks the commit order with `merge-base --is-ancestor`.** The trigger is a phrase list (stated limit).
4. **DEVIATION-001 checks paths that climb with `../`.** A path written without `../` is not read, and guides are
   selected by name (`guias/`, `guide`, `scenario`), so a guide with another name is not selected (stated limit).
5. **M-133 is partly mechanized.** CONVERGE-001 (V3.04) checks that sweep calls with an underscore name are defined
   and that fixture directories have a trap. The batch, class-search, and no-interruption clauses are reviewer obligations.
6. **Git calls in the changed detectors run under a perl alarm (`GIT_TIMEOUT_SECONDS`, 60 s).** Every git call in the
   detectors this change set touches (`func001_scan`, `decision_order_check`, `git_claim_check`) is wrapped with
   `bounded`, and V3.05 asserts the bound with a dependency that never resolves: the call ends in a terminal state and
   the dependency's own completion marker is absent (an event, not a wall-clock measurement). Pre-existing git calls
   in other detectors are a finding under ORIGIN-001 (stated limit), not part of this record.
7. **GIT-001 claims a branch only where one line holds both the branch context and the token, and only a document
   claims.** The prose keywords (`branch`, `rama`, `checkout`) are read case-insensitively; the git ref `HEAD` is read
   in uppercase only, so the word `head` in prose claims nothing (V3.05). A token that names an existing path or a
   document file is a path; a line that is a shell operand and a fenced code block are not claims and are excluded
   from every window; a flag is reported on the line that holds the token, once. The branch type prefixes are a
   closed list (`fs`, `feat`, `fix`, `feature`, `hotfix`, `release`, `bug`, `test`, `task`, `perf`, `revert`, `build`,
   `ci`, `style`, `chore`, `docs`, `refactor`); a branch of an unlisted prefix is not claimed (stated limit).
   Measured by executed runs on the final tree, over the six files that hold the claim classes: three external
   pinned commits (`skills/ui-ux-design/SKILL.md:14`,
   `skills/ui-ux-design/references/web-interface-checklist.md:10`, `workflows/system-validation.md:1281`); no flag
   for the two URL path segments (`references/official-documentation.md:687`, `skills/core-web-vitals/SKILL.md:118`);
   and no flag for the prose word `head` in `workflows/engineering-baseline.md:165`. A URL token is removed from
   the line and its window before the claim is extracted, so a path segment inside a URL is not a branch and a
   real token on the same line is still read (V3.05). The three pinned commits are
   pre-existing findings under ORIGIN-001. The scoped run over the seven files of this change set, on the final
   tree, reports one flag, `workflows/system-validation.md:1281`, the third of those commits, and no other; the
   line holds the same content at `HEAD:1278` and is not added by this change set. A full-repository run with the
   final detector was not executed (cost limit below). A real missing branch in branch
   context still fails (V3.05). No old-detector count is stated: a HEAD archive copy has no real branches, so that
   baseline is methodology-dependent (stated limit). Cost limit (A-6): the check reads a five-line window for every
   line and runs a git call for each candidate, so its run time grows with document size; a scoped run over a handful
   of large documents takes several minutes, and a full-repository run exceeded eight minutes on the development
   machine. The sweep runs it over the change set; a full-repository count is produced deliberately and is not
   repeated for every task. A later audit (A-1) found a `SIN COMMIT` header on a branch that held commits: the
   detector was correct but ran only in the audit, not in the producer of the delivery report. The delivery gate in
   `workflows/task-to-code.md` now runs GIT-001 over the whole report text before delivery. This adds a scope, not a
   detector change; the cost limit above still applies to sweep runs.
8. **ARTIFACT-001 reads a prefix only with a name after it, and a shell operand, a fenced block, a runtime `$` path,
   and a sed expression are not citations** (`> "$dir/x"`, `< "$root/x"`, a command line, code between fences). The
   operand rule is a fixed command vocabulary, so prose that mentions a `$` path or a command is still read
   (V3.05). A URL token is removed from the line and the remainder is read, so a real climbing citation on a URL
   line still fails (V3.05). Measured by executed runs on one input: a `git archive HEAD` export of the base revision (103 Markdown
   files) reports 449 findings with the base detector and 237 with this change's detector; the working tree, over
   its tracked Markdown files and this record, reports 230, none of them on a line added by this change set. Basis:
   the archive list is `find . -name '*.md'` over the export; the working-tree list is `git ls-files '*.md'` plus
   this record. The added-line check maps each finding's `file:line` to the added line numbers of the diff and to
   every line of this new record; no finding maps to one. A sentence-final period is punctuation, not part of the
   token, and a path is reported once (V3.05). Prose and comments are still read; a climbing, home, or absolute
   citation in prose still fails. A relative citation is resolved from the repository root, so a link relative to
   the citing document is a known
   false-positive class and a reviewer decision; the same applies to a violating path quoted as an example, to a
   backticked rule or package identifier with a slash and no file extension on a line that also holds a URL, which
   is read as a directory and reported `NOT VERIFIED`, and to a
   site-root path in web guidance such as the llms.txt proposal at a site root (stated limit).

9. **The reachability probes read the canonical anchor from the registry.** V3.03 and V3.04 do not hold a copy of
   the anchor: a hardcoded copy drifted from `references/mandatory-rules.md` and failed a tree that V2.32 accepts
   (audit A-1). The registry is the single source, and a probe that cannot read it reports a failure.

## Design record

| Pattern | Force inspected | Decision |
|---|---|---|
| (none selected) | Shell detectors in Markdown: no runtime unit, no shared state, no variant behavior, no external client | No pattern applied; the detectors are plain functions in one unit |

| Principle | Status | Unit and reason |
|---|---|---|
| SRP | NOT APPLICABLE | The detectors are documentation-level functions; no runtime unit with more than one reason to change was added |
| OCP | NOT APPLICABLE | No extension point; each check is a fixed function called by the sweep |
| LSP | NOT APPLICABLE | No type hierarchy |
| ISP | NOT APPLICABLE | No interface |
| DIP | NOT APPLICABLE | No concrete module imported by another; the sweep calls functions in the same file |

## Deviations

- None. This record cites no JSON contract document; DEC-005 does not apply to it.

## Consequences

The detectors are as strong as the stated limits. A reader of a report must read the limits before reading a `PASS`.
