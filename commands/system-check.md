---
description: Read-only audit of ai-engineering structure, OpenCode discovery, client configuration, policy consistency, and provider status.
agent: orchestrator
---

Run the System Validation Workflow:

`workflows/system-validation.md`

Resolve the canonical system root first.

Use read-only filesystem and Git inspection.

Check that required files exist, references resolve, Skills are not
duplicated, command agents exist, and PR/review restrictions remain
consistent.

For OpenCode, inspect the actual global configuration instead of assuming
the repository's setup instructions have already been applied.

Use shell commands only when necessary and request permission when
required by OpenCode.

Do not print credentials or secret values.

Do not install tools, change configuration, create links, modify files,
create commits, or access remote write operations.

Return a concise report with PASS, WARN, FAIL, or NOT VERIFIED for each
check.

Recommend corrective changes without applying them.