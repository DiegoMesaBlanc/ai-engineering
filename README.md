# AI Engineering System

A provider-agnostic, free-first engineering workflow for OpenCode and compatible AI coding clients.

## Design principles

- One canonical repository for Skills, agents, commands, contracts, and workflows.
- Load Skills on demand; do not load the entire knowledge base for every task.
- Prefer local Git and supplied artifacts when sufficient; MCP is optional.
- Detect project stack, architecture, repository topology, and conventions from evidence.
- Ask the user for an exact branch name before implementation if one was not supplied.
- Preserve user changes and use isolated worktrees for PR inspection when needed.
- `/pr` generates copy-ready PR text; it does not create a PR.
- `/review-pr` creates copy-ready feedback; it does not publish comments or modify remote repositories.
- Prefer free/open-source options, but use company-provided licenses when authorized.

## Current top-level structure

- `agents/`: OpenCode agents and orchestration.
- `commands/`: commands such as `/task`, `/pr`, and `/review-pr`.
- `contracts/`: provider-neutral task, design, and Git contracts.
- `integrations/`: provider access, security, cost, and compatibility policy.
- `skills/`: reusable domain and engineering knowledge.
- `workflows/`: end-to-end engineering processes.
- `CLAUDE.md`: canonical engineering rules shared through compatible configuration.

Do not create `core/` or `external/` solely to match a previous folder proposal. Add a directory only when it owns real, non-duplicated content.

## OpenCode setup (macOS)

OpenCode supports global Skills, commands, agents, and global rules. See the official documentation:

- https://docs.opencode.ai/docs/skills/
- https://docs.opencode.ai/docs/commands/
- https://docs.opencode.ai/docs/agents/
- https://docs.opencode.ai/docs/rules/

Assuming this repository is at `~/ai-engineering`, OpenCode can discover Skills from `~/.claude/skills` when that link already points to `~/ai-engineering/skills`.

For global rules, commands, and agents, first inspect existing configuration:

```sh
ls -ld "$HOME/.config/opencode" \
  "$HOME/.config/opencode/AGENTS.md" \
  "$HOME/.config/opencode/commands" \
  "$HOME/.config/opencode/agents" \
  "$HOME/.claude/skills" 2>/dev/null
```

Do not replace any existing file or directory. If the corresponding destination does not exist, create the link:

```sh
mkdir -p "$HOME/.config/opencode"

# Run each ln only if its destination does not already exist.
ln -s "$HOME/ai-engineering/CLAUDE.md" "$HOME/.config/opencode/AGENTS.md"
ln -s "$HOME/ai-engineering/commands" "$HOME/.config/opencode/commands"
ln -s "$HOME/ai-engineering/agents" "$HOME/.config/opencode/agents"
```

If `~/.claude/skills` does not already point to this repository and no existing destination would be overwritten, link it too:

```sh
mkdir -p "$HOME/.claude"
ln -s "$HOME/ai-engineering/skills" "$HOME/.claude/skills"
```

These are examples for a clean configuration. Inspect the paths first and execute only the links whose destinations are absent. If any destination already contains your own files, integrate the references manually instead of replacing it.

Restart OpenCode after setting up discovery. Validate with `/help`, then run the demo tasks in the supplied demo project. The current repository has not yet been validated on the user's machine.

## Branch policy

For a task, the assistant must ask for the exact branch name before changing application code. It must validate the name, inspect branch collisions and working-tree state, preserve uncommitted changes, create/prepare the branch or an isolated worktree, and verify the working location. It must not push unless separately requested.

## PR policy

The assistant prepares copy-ready titles, descriptions, checks, and feedback. The user manually creates the PR and pastes review comments. Current workflows must not publish comments, create PRs, approve, merge, or update remote tasks.
