---
description: Produce a durable effort, duration, and optional cost estimation for a complete project from start to finish, from a project artifact supplied to the agent (brief, document, text, link, file, or image), saved in the current working directory, without starting implementation.
agent: orchestrator
---

# /estimate

Run the Estimation Workflow:

`workflows/estimation.md`

Estimate effort, duration, and optional cost for a complete project from
start to finish, from a project artifact the user supplies. This command is a
thin adapter: it does not implement anything and does not start the task
lifecycle.

## Scope

Estimation covers complete projects only. It does not estimate individual
tickets, stories, tasks, or isolated changes; the boundary is defined in
`workflows/estimation.md`.

## Input

`$ARGUMENTS` is the project artifact to estimate. The accepted artifact forms
— a brief, a document, a text or file, a URL, inline text, or an image
supplied as a local path or as an attachment to the session — are defined in
`workflows/estimation.md` (Phase 1).

If no argument is supplied, ask for the artifact instead of guessing one.

## Output

Follow the report defined in `workflows/estimation.md` (Phase 5): the saved
document path, the language and format it was written in, the effort and
duration ranges, the overall confidence, the most material assumptions, and
what could not be determined from the artifact.

Before writing the document, ask the user for the **language** and the
**format**, offering a short list of suggestions, and use exactly what they
choose — do not infer the language from the request. Suggest Markdown
(recommended) or a standalone HTML file (both dependency-free); PDF/DOCX only
with an existing converter, and never add a dependency. If the chosen language
or format is not supported by the current client or model, say so and offer
the closest supported alternative instead of producing it silently.
