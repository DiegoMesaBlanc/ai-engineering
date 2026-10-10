# Task Provider Contract

## Purpose

Define the provider-agnostic contract required to retrieve development
task information.

The implementation may use:

- MCP
- REST API
- GraphQL
- CLI
- local files
- another integration mechanism

The Core must not depend on a specific transport.

---

# Provider Capabilities

A Task Provider may expose:

- get task
- get comments
- get attachments
- get links
- get related tasks
- get parent task
- get child tasks
- get dependencies
- get related pull requests
- get related commits
- update task
- add comment
- change status

---

# Minimum Read Contract

The provider should be capable of retrieving, when available:

```text
Task
├── id
├── title
├── description
├── status
├── priority
├── assignee
├── reporter
├── acceptance criteria
├── comments
├── links
├── attachments
├── parent
├── children
├── dependencies
├── related pull requests
└── related commits
```