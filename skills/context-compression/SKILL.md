---
name: context-compression
description: On-demand, client-agnostic protocol for compressing prompts and tool outputs before they are sent or after they are received, while preserving identifiers, evidence and required checklist text. Load when a context window is under pressure or when raw tool output is too large to keep verbatim.
---

# Context Compression

## Purpose

Reduce the size of text exchanged with a model without losing the facts that
make the exchange usable. This Skill is a procedure, not a tool: it installs no
dependency, writes no hook, changes no client configuration, and is not active
on its own.

## When to apply

Invoke it on demand, in any client (OpenCode, Claude Code, Codex, or another),
by naming the Skill in the request and giving it the text or the artifact to
compress, for example: "Apply `context-compression` to the last tool output".
There is no client-specific slash command and no automatic trigger: the caller
decides each time.

It applies to text at either boundary of a model call:

- **Inbound**: raw output from file reads, searches, logs, builds, HTTP fetches,
  or test runs that is about to enter the conversation.
- **Outbound**: a prompt draft or a handoff that is about to be sent and is
  larger than the information it carries.

It does not replace checkpoints or the minimum-sufficient-context rules in
`workflows/session-resilience.md`; it reduces a single artifact, not the state
of a task.

## Never compress

These items are contractual or procedural in this system and must survive
verbatim. If an input contains one of them, keep that span unchanged even when
compressing the rest:

- phase plans, live checklists, and phase-gate or commit-gate questions;
- verbatim user messages and any attribution that quotes them;
- evidence: commands, exit codes, file paths, line ranges, hashes, counts, and
  the scope definition that produces a count;
- identifiers, contract and error names, URLs, and exact error or stack-trace
  text;
- security warnings, irreversible-step explanations, and failure states;
- acceptance criteria, requirements, constraints, and non-goals;
- diff hunks that a reviewer still has to judge.

When a span is too large to keep and also falls in this list, stop and ask the
caller instead of shortening it.

## Procedure

1. **Name the boundary.** State what the text is, where it came from, and
   whether it is inbound or outbound.
2. **Mark the lossless anchors.** Before changing anything, list the paths,
   line numbers, identifiers, commands, exit codes, error names, and URLs that
   must survive. They are the acceptance criteria of the compression.
3. **Remove only removable material.** Drop repeated headers and navigation
   chrome, duplicated lines, decorative separators, progress chatter, and prose
   that restates the same fact. Replace a long repetition with a count and one
   representative line. Replace a long stack trace with its top frame, the
   error text, and a pointer to the full trace.
4. **Keep the structure.** Preserve ordering that carries meaning, such as the
   order of test failures or the sequence of a reproduction. Do not merge two
   different errors into one summary.
5. **Point at the original.** Record where the uncompressed artifact can be
   recovered: a file path, a tool-output identifier, or the command that
   regenerates it. Compression is reversible only through that pointer.
6. **Label the result.** Mark it as compressed, say what classes of material
   were removed, and state the anchor list that was preserved.

## Verification

Before returning the compressed text, check each item:

- [ ] Every anchor from step 2 is present and unchanged.
- [ ] No item from **Never compress** was shortened.
- [ ] Counts state the set they count and match the lines that remain.
- [ ] The original is recoverable from the recorded pointer.
- [ ] No secret, token, or personal datum was introduced into the compressed
      text; do not print a secret value to prove it was found.
- [ ] The result is labelled compressed and the omissions are named.

If an item fails, return the original and report the failure instead of a
partial compression.

## Measurement

This Skill claims no token-reduction percentage. A saving exists only when it
is measured: use the client's tokenizer or counting command if one is
available, state the exact input and output artifacts, and report both counts
with the tool used. A percentage carried over from another corpus is not
evidence for this one.

## Relation to other components

- `AGENTS.md` governs which text is mandatory; this Skill never overrides it.
- `workflows/session-resilience.md` owns checkpoints and minimum sufficient
  context; this Skill handles one artifact at a time.
- `skills/task-context/SKILL.md` owns requirement normalization; compressed
  task text still goes through that Skill before implementation.
- `skills/knowledge-graph/SKILL.md` is the companion on-demand Skill for
  navigating a large corpus without reading it whole.
