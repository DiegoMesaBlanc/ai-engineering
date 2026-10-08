# /pr

Create or prepare a Pull Request / Merge Request from the current
validated implementation.

---

## Behavior

1. Detect the Git provider.
2. Inspect repository status.
3. Inspect current branch.
4. Determine target branch.
5. Inspect commits.
6. Inspect changed files.
7. Inspect associated task when available.
8. Inspect associated design when available.
9. Validate tests.
10. Validate typecheck when available.
11. Validate lint when available.
12. Validate build when appropriate.
13. Analyze architecture impact.
14. Analyze potential risks.
15. Generate PR title.
16. Generate PR description.
17. Suggest reviewers when evidence exists.
18. Present PR READY summary.
19. Request approval.
20. Create the PR/MR only after approval.

---

## PR READY

Display:

PR READY

Provider:
...

Repository:
...

Source branch:
...

Target branch:
...

Task:
...

Title:
...

Summary:
...

Changes:
...

Architecture:
...

Tests:
...

Typecheck:
...

Lint:
...

Build:
...

Security:
...

Risks:
...

Suggested Reviewers:
...

---

## Safety

Never:

- create a PR containing unrelated changes
- commit secrets
- force push
- rewrite history
- change the target branch silently
- merge automatically
- approve automatically

The command creates the PR only after explicit authorization.

---

## Provider Independence

The command must work with:

- GitHub
- GitLab
- Azure Repos
- Bitbucket
- other supported Git providers

Do not hardcode provider-specific logic in the command.

Use the Git Provider contract and appropriate integration.