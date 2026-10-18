---
description: Produce a durable effort, duration, and optional cost estimation from an artifact supplied to the agent (document, text, link, file, or image), saved in the current working directory, without starting implementation.
agent: orchestrator
---

# /estimate

Run the Estimation Workflow:

`workflows/estimation.md`

Estimate effort, duration, and optional cost from an artifact the user
supplies. This command is a thin adapter: it does not implement anything and
does not start the task lifecycle.

## Input

`$ARGUMENTS` is the artifact to estimate. The accepted artifact forms — a
document, a text or file, a URL, inline text, or an image supplied as a local
path or as an attachment to the session — are defined in
`workflows/estimation.md` (Phase 1).

If no argument is supplied, ask for the artifact instead of guessing one.

## Output

Follow the report defined in `workflows/estimation.md` (Phase 5): the saved
document path, the effort and duration ranges, the overall confidence, the
most material assumptions, and what could not be determined from the artifact.
