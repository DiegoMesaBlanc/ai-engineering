---
description: Prepare copy-ready Pull Request or Merge Request title, description, testing summary, risks, and reviewer notes without publishing anything.
agent: reviewer
---

Prepare a Pull Request / Merge Request proposal for manual creation.

This command generates copy-ready content. It does not create or publish a PR/MR.

## Command

`/generate-pr` prepares a pull request proposal. It never creates, publishes,
or updates a remote pull request.

## Workflow

Apply `skills/pull-request/SKILL.md`. It is the canonical procedure for
preparing a pull request proposal, and it is not restated here.

## Output

Produce the copy-ready proposal defined in `skills/pull-request/SKILL.md`
("Output"). The Skill is the canonical source of the format and it is not
restated here.

## Rules

- Never invent task IDs, links, requirements, reviewers, or test results.
- Avoid unrelated refactoring.
- Use local Git first; MCP is optional.
- Never create a PR/MR, publish comments, push, merge, approve, or update a remote task.
- The user copies the generated content and creates the PR manually.
