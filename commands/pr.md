# /pr

Prepare a Pull Request / Merge Request proposal for manual creation.

This command generates copy-ready content.
It does not create or publish a PR/MR.

## Inputs

Accept any available combination of:

- current local repository
- source branch
- target branch
- task URL or identifier
- design URL or artifact
- PR template or repository contribution guidelines

Infer provider details when reliable evidence is available.

Do not require an MCP if local Git and repository files provide sufficient
information.

## Workflow

1. Inspect repository status and current branch.
2. Identify the intended source and target branches.
3. Inspect commits and relevant changes.
4. Inspect the task when available.
5. Inspect associated design artifacts when relevant.
6. Analyze architecture and affected modules.
7. Identify unrelated or accidental changes.
8. Check available validation evidence.
9. Generate copy-ready PR title and description.
10. Include appropriate testing and risk information.

Do not load unrelated project documentation unless needed.

## Output

### PR TITLE

<Copy-ready title>

### PR DESCRIPTION

#### Summary
<What changed and why>

#### Task / Requirements
<Task reference and acceptance criteria addressed, when available>

#### Changes
<Important implementation details>

#### Design
<Relevant design decisions, when applicable>

#### Architecture
<Architectural impact, if significant>

#### Testing
<Only tests and validations actually performed>

#### Risks and Limitations
<Relevant risks or unresolved issues>

#### Reviewer Notes
<Files or decisions reviewers should focus on>

#### Checklist
<Project-specific checklist, when available>

## Quality Rules

- Follow the repository's existing PR template and conventions.
- Prefer concise, concrete, technically useful descriptions.
- Never claim tests passed unless they were executed or reliable results
  were retrieved.
- Do not invent task IDs, links, requirements, or reviewers.
- Do not include unrelated refactoring.
- Use the language expected by the project or team.

## Strict Read-Only Policy

Never:

- create a PR/MR
- publish comments
- push a branch
- merge
- approve a PR
- change a task
- modify a remote repository

The user copies the generated content and creates the PR manually.