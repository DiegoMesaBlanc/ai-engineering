---
name: knowledge-graph
description: On-demand, client-agnostic protocol for building a lightweight knowledge graph of a codebase or documentation set and querying it before reading files whole. Uses only the client's local tools, with no external service, no API key and no always-on instructions.
---

# Knowledge Graph

## Purpose

Answer "where does this live and what depends on it" without reading an entire
corpus. A graph here is a plain index of nodes and edges with citations back to
the source. It is an aid to navigation, not a replacement for reading the code
or document that answers the question.

This Skill is a procedure. It installs no package, contacts no service, sends
no data off the machine, writes no hook, and is not active on its own.

## When to apply

Invoke it on demand in any client (OpenCode, Claude Code, Codex, or another) by
naming the Skill and the target directory or question, for example: "Apply
`knowledge-graph` to `src/` for the billing flow". There is no client-specific
command and no automatic trigger. It is useful when the same corpus will be
queried several times, or when a broad question would otherwise require many
full file reads. It is not useful for a single file or a one-shot grep.

## Build

Use only the read and search tools the client already has.

1. **Fix the scope.** Record the target root, the file extensions included, the
   paths excluded, and the rule used to decide both. State the count and its
   definition; do not carry a count over from an earlier run.
2. **Record the baseline.** Note the branch, HEAD or revision, and the working
   tree state. A graph built from a dirty tree says so explicitly.
3. **Collect nodes.** A node is a file, a document section, a symbol, a
   contract, a schema object, or a decision. Give each node a stable identifier
   based on its path and, where applicable, its symbol name. Record one
   `file:line` citation per node.
4. **Collect edges.** An edge carries a kind — `imports`, `calls`, `defines`,
   `implements`, `references`, `owns`, `depends-on` — and the `file:line` where
   the relationship is established. Do not infer an edge from a name alone.
5. **Write the index.** Keep it as plain text or Markdown in the project's
   existing notes convention. If the project has no tracked location for such
   notes, keep it in the user-level handoff directory outside the source tree.
   Do not add a generated file to the repository just to hold the graph.
6. **Stamp it.** Record the tool used, the build date, the scope definition,
   and the baseline from step 2 so a later reader can tell whether it is stale.

## Query

1. Read the index, not the corpus. Find candidate nodes by identifier, path, or
   edge kind.
2. Follow the edges needed to answer the question. Stop when the answer is
   supported by citations rather than by breadth.
3. Open only the cited spans to confirm what the index claims. The graph is an
   index: a cited source line wins over a stale edge.
4. Answer with the citations used and state the graph's baseline. If the
   working tree or HEAD has changed since the build, say the graph may be stale
   and re-run the affected part of the build before relying on it.

## Never

- Do not treat the graph as proof. Every claim that reaches a decision or a
  report needs a source citation checked at query time.
- Do not record secrets, tokens, credentials, or personal data in a node.
- Do not send the corpus, the index, or a prompt to an external graph service or
  LLM backend. Everything runs with the client's local read and search tools.
- Do not install a dependency or add an MCP server to make this Skill work.
- Do not write always-on instructions into `AGENTS.md`, `CLAUDE.md`, or any
  client configuration. The caller invokes this Skill each time it is needed.
- Do not claim a token saving without a measurement of the compared paths.

## Verification

- [ ] Scope, exclusions, count and its definition are recorded.
- [ ] Baseline (branch, HEAD, working tree) is recorded and checked at query time.
- [ ] Every node and edge has a `file:line` citation.
- [ ] Every edge kind is one of the declared kinds.
- [ ] The answer cites source lines that were actually opened.
- [ ] No secret or personal datum appears in the index.
- [ ] Nothing was installed, sent off-machine, or added to always-on config.

## Relation to other components

- `skills/context-compression/SKILL.md` is the companion for shrinking a text
  that still has to be read; this Skill avoids reading it in the first place.
- `workflows/session-resilience.md` owns the handoff directory and the minimum
  sufficient context; the graph uses that directory, it does not replace the
  checkpoint.
- `skills/architecture-advisor/SKILL.md` owns architectural decisions; the
  graph supplies navigation evidence to that analysis.
