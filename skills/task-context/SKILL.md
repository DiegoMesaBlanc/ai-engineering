---
name: task-context
description: Retrieve, normalize, and analyze development-task information independently of the task-management platform used by the project.
---

# Task Context

## Purpose

Transform task-management information into a normalized engineering context.

The task source may be:

- Jira
- Azure DevOps
- GitHub Issues
- GitLab Issues
- Linear
- YouTrack
- another task management system
- local Markdown
- JSON
- direct user input

Do not assume a specific provider.

Task retrieval is a provider responsibility.
Task interpretation is an engineering responsibility.

---

# Core Principle

A task is not only a title.

A useful engineering task context includes:

- business intent
- functional requirements
- acceptance criteria
- constraints
- dependencies
- design references
- technical references
- related tasks
- comments
- attachments
- status
- priority
- ownership

Never invent missing information.

---

# Automatic Task Interpretation

A short, ordinary task description is a complete valid task input.

Examples of sufficient input:

- "Implement authentication with MSAL."
- "Fix the checkout total when a discount code is applied."
- a Jira or Azure DevOps ticket URL
- a pasted ticket description
- a local task artifact

Never require the user to:

- rewrite the task into an elaborate prompt
- author or update a Markdown task specification
- provide manual implementation instructions
- list technical acceptance criteria that can be derived from evidence
- choose frameworks, libraries, or structures the repository already
  determines

Interpret the task automatically:

1. Retrieve the task when access is authorized and available.
2. Preserve the original wording and explicit acceptance criteria.
3. Inspect the target repository, relevant code, project instructions,
   architecture, dependencies, configuration, tests, and designs.
4. Detect the intended engineering outcome.
5. Infer likely implementation requirements from the task and actual
   project evidence.
6. Identify testable acceptance criteria, affected modules, constraints,
   risks, and non-goals.
7. Resolve details autonomously through existing code, conventions,
   **`references/official-documentation.md`** (for the detected stack), and engineering policy.
8. Identify only the material ambiguities that cannot safely be inferred.

When a provider is unavailable, use the available task text or artifact.
Request only the minimum missing information needed to proceed.

Never claim to have read inaccessible task content.

Never present an inferred requirement as if the task stated it explicitly.
Never invent business rules, security policies, or acceptance criteria
unsupported by available evidence.

---

# Working Interpretation

Construct a concise, temporary working interpretation within the current
agent session. It contains:

- original objective
- explicit requirements from the task
- inferred requirements supported by repository or design evidence
- proposed acceptance criteria
- assumptions and material unknowns
- affected components and architectural implications
- dependency assessment outcome
- proportional rigor level and the plan artifact decision, recorded after
  the planning phase
- security and validation considerations
- decisions that require user input

Rules:

- Do not create a Markdown file for the working interpretation by default.
- Do not ask the user to author or complete it.
- Persist information only through existing checkpoint mechanisms when
  continuity requires it, or when the user requests a durable artifact.
- Present inferred requirements as inferred, never as explicit task text.
- The working interpretation is session state, not a deliverable.

---

# Task Retrieval

Retrieve, when available:

- task identifier
- title
- description
- acceptance criteria
- status
- priority
- assignee
- reporter
- comments
- attachments
- linked issues
- parent task
- child tasks
- dependencies
- related pull requests
- related commits
- design references
- technical references

If information is unavailable, report it as unavailable.

---

# Requirement Extraction

Separate information into:

## Business Intent

Why the task exists.

## Functional Requirements

What the system must do.

## Non-Functional Requirements

Examples:

- performance
- security
- accessibility
- availability
- compatibility
- observability

## Acceptance Criteria

Specific conditions that determine completion.

## Constraints

Technical, business, architectural, regulatory, or operational constraints.

## Dependencies

Other work or systems required for completion.

---

# Link Analysis

Inspect references associated with the task.

Possible references:

- design URLs
- documentation
- repositories
- pull requests
- issues
- API specifications
- diagrams
- architecture documents
- attachments

Identify their type when possible.

Do not assume that every URL is relevant.

---

# Design Detection

Identify design references such as:

- Figma
- Penpot
- Sketch
- Adobe XD
- Storybook
- screenshots
- PDFs
- image files
- HTML prototypes
- diagrams
- wireframes
- other visual artifacts

When design references exist, pass them to the Design Analysis workflow.

Do not perform platform-specific design analysis inside this Skill.

---

# Task Ambiguity

Identify:

- ambiguous requirements
- conflicting requirements
- missing acceptance criteria
- incomplete design references
- unclear dependencies
- unclear expected behavior

Only ask the user when the ambiguity materially affects implementation.

Do not ask questions that can be answered by inspecting the repository,
task history, design, or existing conventions.

---

# Task Scope

Determine:

- intended scope
- affected feature/module
- potential out-of-scope changes

Prevent unrelated work from silently becoming part of the task.

If potentially valuable improvements are discovered, report them separately.

---

# Existing Project Context

When used with an existing project, relate the task to:

- existing modules
- existing components
- existing services
- existing APIs
- existing state management
- existing architecture
- existing tests
- existing design system

Prefer reusing existing abstractions.

---

# New Project Context

For a new project, use task information to help establish:

- initial requirements
- scope
- architecture constraints
- initial stack decisions
- first development milestone

Do not assume the first task defines the entire future architecture.

Use the simplest architecture that supports the known requirements.

---

# Output

Produce:

## TASK CONTEXT

### Identification

- Provider:
- Task ID:
- Title:
- Status:
- Priority:
- Assignee:

### Business Intent

...

### Functional Requirements

1. ...

### Non-Functional Requirements

1. ...

### Acceptance Criteria

1. ...

### Constraints

- ...

### Dependencies

- ...

### Design References

- ...

### Technical References

- ...

### Related Work

- ...

### Scope

#### In Scope

- ...

#### Out of Scope

- ...

### Unknowns

- ...

### Proportional Rigor

- Complexity (definitive):
- Rigor level (R1–R4, recorded after planning):
- Plan artifact (session state or durable, with reason):

### Risks

- ...

### Recommended Next Step

...

---

# Engineering Rules

1. Never invent requirements.
2. Never invent acceptance criteria.
3. Never assume a task provider.
4. Never assume Figma.
5. Separate requirements from implementation decisions.
6. Separate explicit information from inference.
7. Preserve traceability to the original task.
8. Reuse repository context whenever available.
9. Ask only materially necessary questions.
10. Do not expand scope silently.

---

## Direct References and URL Resolution

Task input may be:

- a URL
- a provider-specific identifier
- a local file
- an attachment
- a pasted description
- multiple labeled references

When a URL is supplied:

1. Identify the likely resource type and provider.
2. Attempt retrieval using the available authorized read-only tools.
3. Prefer information from the original task source.
4. Retrieve relevant acceptance criteria, comments, dependencies, and links.
5. Pass associated design references to Design Analysis.
6. Preserve the source URL for traceability.

If the resource is inaccessible:

- do not invent its contents
- identify what could not be retrieved
- use other available task information
- request an export or pasted content only if necessary

A direct URL is a valid task input. A local Markdown file is not mandatory.
