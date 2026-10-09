---
description: Prepare copy-ready Pull Request or Merge Request title, description, testing summary, risks, and reviewer notes without publishing anything.
agent: reviewer
---

Prepare a Pull Request / Merge Request proposal for manual creation.

This command generates copy-ready content. It does not create or publish a PR/MR.

## Workflow

1. Inspect repository status, current branch, commits, and changed files.
2. Identify intended source and target branches.
3. Inspect the task and relevant design when available.
4. Analyze architecture impact, unrelated changes, and risks.
5. Report validation actually performed; identify checks not run.
6. Follow an existing repository PR template when available.
7. Generate the copy-ready proposal.

## Output

### PR TITLE

<Concise title following repository conventions>

### PR DESCRIPTION

#### Summary
<What changed and why>

#### Task / Requirements
<Actual task reference and acceptance criteria, when available>

#### Changes
<Meaningful implementation details>

#### Design / Architecture
<Only relevant decisions>

#### Testing
<Executed checks and results, plus important checks not run>

#### Risks and Limitations
<Relevant risks>

#### Reviewer Notes
<Areas requiring attention>

#### Checklist
<Use project-specific checklist if present>

## Rules

- Never invent task IDs, links, requirements, reviewers, or test results.
- Avoid unrelated refactoring.
- Use local Git first; MCP is optional.
- Never create a PR/MR, publish comments, push, merge, approve, or update a remote task.
- The user copies the generated content and creates the PR manually.
