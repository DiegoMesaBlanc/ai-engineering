# Portability Contract

## Purpose

`ai-engineering` is the source of truth and must work for every client and
every machine where it is installed. This reference defines what that means
in practice, and it is the reference the system consults before assuming
anything about the environment.

## The contract

1. **The repository is the source of truth, not the installation.** Any
   client can be pointed at the canonical path. Linking the repository into
   a client's discovery directory is a convenience for clients that only
   look in a fixed location; it is never a requirement for the rules to
   apply.
2. **The knowledge is client-neutral.** No rule, workflow, or reference
   depends on a specific client's feature. `commands/` and `agents/` are
   adapters for clients that support them; every workflow is also reachable
   without them.
3. **The procedures assume a POSIX shell.** `grep`, `sed`, `tr`, `awk`,
   `printf`, and `git` are the baseline. They are native on macOS and
   Linux, and available on Windows through Git Bash, WSL, or a POSIX layer.
   When a required tool is missing, the check reports `NOT VERIFIED` with the
   reason. It never reports a passed check it could not run.
4. **No operating-system assumption is baked into the rules.** No hardcoded
   `/tmp`, no macOS-only utilities in a shared procedure, no absolute
   system path.
5. **The learning store is resolved by the client**, not assumed:
   `${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}`. A client that
   cannot resolve `$HOME` sets the variable explicitly.

## Path resolution

Every path in the system is resolved from the repository root. The
canonical root is exposed to the agent as `AI_ENGINEERING_HOME` or as the
repository root it is working in.

| Consumer | How it reaches the knowledge |
|---|---|
| Any client, any OS | Read `$AI_ENGINEERING_HOME/AGENTS.md` and follow the workflows by path |
| OpenCode | Global configuration directory, with `AGENTS.md`, `commands/`, and `agents/` linked to the canonical repository |
| Claude Code | `CLAUDE.md` with `@AGENTS.md`, and the canonical `skills/` directory exposed as the global Skill directory |
| Codex CLI | `AGENTS.md` from the repository root, read directly |
| Any client without Skills support | The canonical wording in `AGENTS.md` is enough; the Skills add operational detail |

The last row is why the canonical wording of every mandatory rule lives in
`AGENTS.md` and not in a Skill.

## Setup per environment

Setting the root:

```sh
# macOS
export AI_ENGINEERING_HOME="$HOME/Documents/ai-engineering"

# Linux
export AI_ENGINEERING_HOME="$HOME/ai-engineering"

# Windows PowerShell
$env:AI_ENGINEERING_HOME = "$HOME\ai-engineering"
```

The path is whatever the repository actually is. Verify it before use:

```sh
git -C "$AI_ENGINEERING_HOME" status --short --branch
```

### Linking, when a client only looks in a fixed directory

Inspect every destination before writing to it, and never overwrite an
existing file or link.

```sh
# macOS and Linux
ln -s "$AI_ENGINEERING_HOME/AGENTS.md" "$HOME/.config/opencode/AGENTS.md"
ln -s "$AI_ENGINEERING_HOME/commands" "$HOME/.config/opencode/commands"
ln -s "$AI_ENGINEERING_HOME/agents"   "$AI_ENGINEERING_HOME/agents"
```

On Windows, a symbolic link needs Developer Mode or an elevated shell. When
it is unavailable, point the client at the canonical path instead of
linking; the rules do not depend on the link.

### Skill directory

The canonical Skills live in the repository's `skills/` directory. When the
client expects them in a global directory, expose the canonical directory
there. When the global directory already exists and holds other Skills, link
only the canonical ones that are missing, and never overwrite a name that
already exists.

## Degradation rules

- A tool the environment does not have makes its check `NOT VERIFIED`, with
  the tool named. It is never a pass and never a silent skip.
- A client that cannot discover Skills still receives every mandatory rule,
  because the canonical wording is in `AGENTS.md`.
- A client that cannot read a file by path is pointed at the file with an
  absolute path, not a relative one.
- A procedure that assumes a POSIX shell states the assumption, so a Windows
  user knows to run it through Git Bash or WSL.

## Non-portable constructs

The following are not portable and must not appear in a shared procedure.
This list is the single source the audit check reads.

| Construct | Why | Portable alternative |
|---|---|---|
| An absolute temporary path such as `/tmp/` | Does not exist on every system | Pipe the data instead of writing a temporary file |
| A macOS-only hashing utility | macOS only | Use a command that exists on both, or state the assumption |
| The macOS-only form of an in-place edit | macOS only | Use the portable form, or state the assumption |
| The macOS-only form of a file-stat call | macOS only | Use the portable form, or state the assumption |
| The GNU-only form of a path resolver | Linux only | Use the portable form, or state the assumption |
| A client configuration directory | Client-specific | Resolve the client path at setup time |
| A home-relative absolute path in a shared procedure | Assumes one layout | Resolve it from the repository root or an environment variable |

## What the system must never do

- Assume a client's discovery directory, or that a link exists.
- Hardcode an operating-system path in a shared procedure.
- Report a check that did not execute as passed.
- Depend on a client-specific feature for a rule to apply.
