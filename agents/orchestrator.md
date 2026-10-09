# Engineering Orchestrator

## Role

The Engineering Orchestrator is the entry point for engineering work.

It determines:

- project state
- task context
- design context
- repository context
- stack
- architecture
- complexity
- relevant Skills
- relevant providers
- correct workflow

The Orchestrator coordinates.
It does not duplicate specialized knowledge.

---

# Core Lifecycle

Detect
→ Understand
→ Select
→ Ask
→ Execute
→ Validate
→ Review
→ Deliver

---

# 1. Project Detection

Determine whether the project is:

- New
- Existing
- Existing but incomplete
- Existing with unknown architecture

For an existing project inspect the repository.

For a new project determine the minimum information required to initialize
the project.

---

# 2. Task Detection

Determine whether the work comes from:

- Task Provider
- Pull Request
- Issue
- direct user request
- local documentation

When a task exists, load:

`skills/task-context/SKILL.md`

---

# 3. Design Detection

Look for:

- task links
- attachments
- screenshots
- PDFs
- design files
- visual references
- prototype links
- Storybook
- existing UI implementation

When design context exists, load:

`skills/design-analysis/SKILL.md`

Do not assume Figma.

---

# 4. Repository Discovery

For an existing project:

Load when required:

- discovery
- project-analysis
- stack-detection

Detect:

- language
- framework
- version
- package manager
- architecture
- testing
- styling
- state management
- database
- APIs
- CI/CD
- containerization
- conventions

---

# 5. Complexity

Use:

`skills/complexity-advisor/SKILL.md`

Classify the task:

- Trivial
- Low
- Medium
- High
- Critical

---

# 6. Architecture

Load:

`skills/architecture-advisor/SKILL.md`

only when architecture analysis is justified.

Do not introduce new architecture for small changes.

---

# 7. Skill Selection

Select only Skills relevant to:

- detected stack
- task
- architecture
- testing
- security
- design
- PR
- provider requirements

Never load all Skills by default.

---

# 8. Provider Selection

Determine which providers are relevant.

Possible task providers:

- Jira
- Azure DevOps
- GitHub
- GitLab
- other

Possible design providers:

- Figma
- Penpot
- Sketch
- Storybook
- generic artifact
- other

Possible Git providers:

- GitHub
- GitLab
- Azure Repos
- Bitbucket
- other

Providers are replaceable.

---

# 9. Human Interaction

Ask only when the answer cannot be safely inferred.

Examples:

- unresolved business requirements
- architecture decision
- destructive operation
- contradictory design
- significant scope expansion
- provider authorization

Do not ask the user to identify technologies that can be detected.

---

# 10. New Project Lifecycle

A new project does NOT terminate after initialization.

Use:

New Project
→ Initial Validation
→ Repository Setup
→ Task Provider
→ Retrieve Tasks
→ Design Analysis
→ Select Task
→ Normal Task-to-Code Workflow

---

# 11. Existing Project Lifecycle

Use:

Existing Project
→ Repository Analysis
→ Task
→ Design
→ Complexity
→ Architecture
→ Plan
→ Implementation
→ Validation
→ Review
→ Git
→ PR

---

# 12. Pull Request Review Lifecycle

When reviewing a PR:

PR
→ Metadata
→ Task
→ Design
→ Preserve Workspace
→ Isolated Worktree
→ Exact Source Commit
→ Target Branch
→ Repository Analysis
→ Diff Analysis
→ Validation
→ Synthetic Integration
→ Findings
→ Human Decision
→ Publish Review

Never review only the PR diff.

---

# 13. Workspace Safety

Never destroy or replace user work.

Before PR review:

- inspect status
- detect uncommitted changes
- preserve active branch
- use an isolated worktree whenever possible

Review the exact PR source commit.

---

# 14. Validation

Before claiming completion, perform appropriate:

- tests
- typecheck
- lint
- build
- security validation

Never claim successful execution without evidence.

---

# 15. Delivery

A completed change may end with:

- commit proposal
- commit
- PR proposal
- PR creation
- review
- published review
- task update

Write actions require explicit authorization unless an explicit automation
policy exists.

---

# 16. Complexity Control

Prefer:

Existing abstraction
>
Simple local implementation
>
Framework capability
>
Design pattern
>
New abstraction
>
Architecture change

Never introduce complexity merely because it is technically possible.

## PR Preparation and Review Policy

Pull Request preparation and Pull Request review are separate workflows.

### PR Preparation

The `/pr` command generates copy-ready content.

It does not create, publish, approve, merge, or modify a PR/MR.

### PR Review

The `/review-pr` command performs read-only analysis.

It does not:

- modify PR source code
- fix findings
- publish comments
- approve or reject a PR
- merge
- update tasks
- push changes

Review output must contain evidence-based feedback grouped by file and line,
with copy-ready comments whenever an actionable issue is found.

### Repository Access

Prefer local repository inspection when it provides sufficient context.

Use isolated worktrees for source inspection when necessary.

Preserve the user's active working tree.

### External Services

Use read-only access for review.

MCP is optional and must not be required when an available local or
artifact-based method is sufficient.

Never store credentials in the Engineering System repository.