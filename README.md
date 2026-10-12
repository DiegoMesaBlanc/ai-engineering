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
- Use official documentation on demand for the detected technology and version.
- Evaluate external Skills across the full supported stack without installing one per framework by default.
- Require a configured formatter and linter for every project, with reproducible commands and honest validation results.

## Current top-level structure

- `agents/`: OpenCode agents and orchestration.
- `commands/`: commands such as `/task`, `/pr`, `/review-pr`, `/system-check`, and `/validate`.
- `contracts/`: provider-neutral task, design, and Git contracts.
- `integrations/`: provider access, security, cost, and compatibility policy.
- `references/`: official documentation indexes and technical references.
- `integrations/skill-catalog.md`: evaluation decisions for external Skills.
- `integrations/external-skill-registry.md`: provenance and license records for Skills actually installed.
- `skills/`: reusable domain and engineering knowledge.
- `workflows/`: end-to-end engineering processes.
- `AGENTS.md`: canonical shared engineering instructions.
- `CLAUDE.md`: Claude Code compatibility entry point importing `AGENTS.md`.

Do not create `core/` or `external/` solely to match a previous folder proposal. Add a directory only when it owns real, non-duplicated content.

## Commands

- `/task`: start or continue an engineering task.
- `/pr`: generate copy-ready PR/MR content.
- `/review-pr`: generate copy-ready review feedback.
- `/system-check`: read-only validation of repository consistency and
  OpenCode integration.
- `/validate`: validate project changes using existing tools and
  risk-based checks without modifying application source code.

OpenCode supports Markdown commands and agents with permissions defined in
frontmatter. This diagnostic therefore integrates without adding execution
code or a new runtime dependency.

### System validation versus project validation

- `/system-check` audits the AI Engineering System, file structure,
  Skills, workflows, references, and client configuration.
- `/validate` audits the current software project's changes using
  detected tooling, relevant tests, static checks, and build validation,
  and configures the mandatory formatter and linter when they are missing.

## OpenCode setup (macOS)

Set the path to the canonical repository:

```sh
export AI_ENGINEERING_HOME="$HOME/Documents/ai-engineering"
```

Verify the repository:

```sh
git -C "$AI_ENGINEERING_HOME" status --short --branch
```

OpenCode's global configuration directory is:

```sh
"$HOME/.config/opencode"
```

Before creating links, inspect every destination:

```sh
ls -ld "$HOME/.config/opencode/AGENTS.md" \
  "$HOME/.config/opencode/commands" \
  "$HOME/.config/opencode/agents" \
  "$HOME/.claude/skills" 2>/dev/null
```

If a destination does not exist, link it to the canonical repository:

```sh
ln -s "$AI_ENGINEERING_HOME/AGENTS.md" \
  "$HOME/.config/opencode/AGENTS.md"

ln -s "$AI_ENGINEERING_HOME/commands" \
  "$HOME/.config/opencode/commands"

ln -s "$AI_ENGINEERING_HOME/agents" \
  "$HOME/.config/opencode/agents"
```

Never overwrite an existing configuration or symlink without inspecting
its target.

Skills are stored in the canonical repository's `skills/` directory.
Reuse an existing compatible global Skill link when it already targets
this directory.

Restart OpenCode after modifying discovery configuration.

The installation must use one canonical repository. Client adapters must
not duplicate Skills, contracts, or workflows.

## Branch policy

For a task, the assistant must ask for the exact branch name before changing application code. It must validate the name, inspect branch collisions and working-tree state, preserve uncommitted changes, create/prepare the branch or an isolated worktree, and verify the working location. It must not push unless separately requested.

## PR policy

The assistant prepares copy-ready titles, descriptions, checks, and feedback. The user manually creates the PR and pastes review comments. Current workflows must not publish comments, create PRs, approve, merge, or update remote tasks.
