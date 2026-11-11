---
description: Prepare this machine for ai-engineering on macOS, Linux, or Windows (Git Bash): detect the operating system, verify the tools, create the local state root, and confirm the client links, without overwriting anything.
agent: orchestrator
---

Run the Bootstrap Workflow:

`workflows/bootstrap.md`

Read the fenced block in "The block" of that workflow and run it as one shell
call from the repository root. Report its output as printed.

The block creates only the links listed in `workflows/bootstrap.md`, and it
never overwrites a destination it does not own. Do not install tools; report the
install command the block prints.
