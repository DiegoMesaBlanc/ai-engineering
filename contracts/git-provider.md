# Git Provider Contract

## Purpose

Define provider-agnostic operations for repository hosting and
Pull Request / Merge Request workflows.

Possible providers:

- GitHub
- GitLab
- Azure Repos
- Bitbucket
- other compatible providers

---

# Read Capabilities

A provider may expose:

- get repository
- get branches
- get commits
- get Pull Request / Merge Request
- get changed files
- get diff
- get comments
- get reviews
- get reviewers
- get linked tasks
- get linked designs
- get CI status
- get check status

---

# Write Capabilities

A provider may expose:

- create Pull Request / Merge Request
- add comment
- add inline comment
- create review
- request reviewers
- add labels
- update title
- update description
- update status
- link task

All write operations require explicit authorization.

---

# Pull Request Model

Normalize provider-specific PR/MR data into:

```text
PullRequest
├── id
├── title
├── description
├── source branch
├── target branch
├── source commit
├── target commit
├── author
├── reviewers
├── changed files
├── commits
├── comments
├── reviews
├── linked tasks
├── linked designs
└── status
```

## Current Operating Scope

The current engineering workflows use provider integrations primarily
for read-only inspection.

The following operations are not enabled in the current workflow:

- Create or publish Pull Requests / Merge Requests.
- Publish review comments.
- Approve or merge PRs/MRs.
- Push source branches.
- Update remote task status.

The pull-request Skill prepares copy-ready text.

The pull-request-review Skill produces copy-ready feedback.

The human user performs the actual publication.

Future provider capabilities documented in this contract must not be
treated as permission to execute them.