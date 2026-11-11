# Bootstrap Workflow

## Purpose

Prepare this machine so every agent can use the system from any project. It runs
without a hook and without any manual step: the first time an agent opens this
repository, `AGENTS.md` makes the agent run the block below once. The block
creates the global links that make the agents, commands, Skills, and workflows
discoverable, then records that this repository is ready here.

The logic is one POSIX shell block. It runs on macOS, Linux, and Windows through
Git Bash, and it is the only copy of the procedure.

## Authorized paths

The block writes only these links, and only when the destination is free. These
are the paths the user authorized:

| Link | Points to |
|---|---|
| `$HOME/.config/opencode/AGENTS.md` | `AGENTS.md` of the repository |
| `$HOME/.config/opencode/commands` | `commands/` |
| `$HOME/.config/opencode/agents` | `agents/` |
| `$HOME/.claude/AGENTS.md` | `AGENTS.md` |
| `$HOME/.claude/CLAUDE.md` | `CLAUDE.md`, which imports `@AGENTS.md` |
| `$HOME/.claude/skills` | `skills/` |
| `$HOME/.codex/AGENTS.md` | `AGENTS.md` |
| `$HOME/.codex/skills` | `skills/` |

A broken link is repaired: a symbolic link whose target no longer exists, for
example after the repository was moved, is pointed at the current repository. A
symbolic link to an existing path elsewhere is not repaired, because that path
belongs to someone else. A destination that already exists and is not a link to
the repository is never changed. It is a `FAIL`: the client cannot discover the system through it, so the
repository is not recorded as ready until the user moves or removes it. A link
that cannot be created (for example on Windows without Developer Mode) is also a
`FAIL`, for the same reason.

## When it runs

- **Agent (automatic):** `AGENTS.md`, the first action of a session in this
  repository, checks the record. If it is missing, the agent runs the block once
  with one shell call, reports one line, and continues with the request.
- **Person (manual, optional):** paste the block in a terminal at the repository
  root.

Links that the first session creates apply to the next session of each client, so
restart the client once after the first run.

## The block

```sh
(
  os=$(uname -s 2>/dev/null)
  case "$os" in
    Darwin) os=macos ;;
    Linux) os=linux ;;
    MINGW*|MSYS*|CYGWIN*) os=windows ;;
    *) os=unknown ;;
  esac
  failures=0
  pass() { echo "PASS  $*"; }
  fail() { echo "FAIL  $*"; failures=$((failures + 1)); }
  note() { echo "NOTE  $*"; }
  echo "AI Engineering System bootstrap (os: $os)"

  for tool in git awk grep dirname mkdir ln readlink cksum; do
    if command -v "$tool" >/dev/null 2>&1; then
      pass "$tool found"
    else
      fail "$tool missing: macOS 'xcode-select --install' or 'brew install $tool'; Linux your package manager; Windows 'winget install --id Git.Git -e'"
    fi
  done

  root=""
  if command -v git >/dev/null 2>&1; then
    root=$(git rev-parse --show-toplevel 2>/dev/null)
  fi
  if ! command -v git >/dev/null 2>&1; then
    fail "repository root not checked: git is missing"
  elif [ -z "$root" ]; then
    fail "run this block from inside the repository"
  else
    pass "repository root $root"
    for required in AGENTS.md CLAUDE.md workflows/self-learning.md workflows/system-validation.md workflows/bootstrap.md; do
      if [ -f "$root/$required" ]; then pass "$required present"; else fail "$required missing"; fi
    done

    state="${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}"
    if mkdir -p "$state/learnings/global" "$state/handoffs" "$state/bootstrap"; then
      pass "state root $state"
    else
      fail "cannot create the state root $state"
    fi

    index="$state/learnings/index.tsv"
    if [ ! -f "$index" ]; then
      printf 'id\tscope\tstatus\tcheck\tsource\tdate\tpath\n' > "$index"
      pass "ledger index created"
    else
      pass "ledger index kept as it is"
    fi
    if awk -F '\t' 'NR == 1 { exit ($1 == "id" && NF == 7) ? 0 : 1 }' "$index"; then
      pass "ledger header valid"
    else
      fail "ledger header is not the expected one: $index"
    fi

    link_one() {
      target="$1"
      dest="$2"
      if [ -L "$dest" ] && [ "$(readlink "$dest")" = "$target" ]; then
        pass "$dest already links to the repository"
      elif [ -L "$dest" ] && [ ! -e "$dest" ]; then
        if ln -sfn "$target" "$dest"; then
          pass "repaired $dest (its old target no longer exists, for example after the repository moved)"
        else
          fail "could not repair the broken link $dest"
        fi
      elif [ -e "$dest" ] || [ -L "$dest" ]; then
        fail "$dest exists and is not a link to this repository; it was not changed. Move or remove it, then run again"
      elif mkdir -p "$(dirname "$dest")" && ln -s "$target" "$dest" 2>/dev/null; then
        pass "linked $dest"
      else
        fail "could not link $dest (on Windows enable Developer Mode, or point the client at the repository path); the client cannot discover the system until it is linked"
      fi
    }
    link_one "$root/AGENTS.md" "$HOME/.config/opencode/AGENTS.md"
    link_one "$root/commands" "$HOME/.config/opencode/commands"
    link_one "$root/agents" "$HOME/.config/opencode/agents"
    link_one "$root/AGENTS.md" "$HOME/.claude/AGENTS.md"
    link_one "$root/CLAUDE.md" "$HOME/.claude/CLAUDE.md"
    link_one "$root/skills" "$HOME/.claude/skills"
    link_one "$root/AGENTS.md" "$HOME/.codex/AGENTS.md"
    link_one "$root/skills" "$HOME/.codex/skills"

    if [ "$failures" -eq 0 ]; then
      key=$(printf '%s' "$root" | cksum | awk '{print $1}')
      printf '%s\n' "$root" > "$state/bootstrap/$key.done"
      pass "repository recorded as ready (key $key)"
    fi
  fi

  if [ "$failures" -eq 0 ]; then
    pass "required checks passed"
    if [ "$os" = "windows" ]; then
      note "OpenCode, Claude, and Codex discovery on Windows is NOT VERIFIED until each client is opened there."
    fi
  else
    echo "FAIL  $failures check(s) failed; fix them and run the block again."
    exit 1
  fi
)
```

## Record check

The record is keyed by the repository root with `cksum`, the same POSIX checksum
the block uses. `AGENTS.md` runs this one line, which prints `ready` or `missing`:

```sh
key=$(printf '%s' "$(git rev-parse --show-toplevel)" | cksum | awk '{print $1}'); test -f "${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}/bootstrap/$key.done" && echo ready || echo missing
```

## Steps

1. **Tools.** `git`, `awk`, `grep`, `dirname`, `mkdir`, `ln`, `readlink`, and
   `cksum` must be on the path. A missing tool is a `FAIL` with its install
   command. The block never installs software; that needs approval per package.
2. **Repository.** The root comes from `git rev-parse --show-toplevel`. No path is
   hardcoded. The canonical files must exist.
3. **State root.** `${AI_ENGINEERING_STATE_HOME:-$HOME/.ai-engineering}` gets its
   directories and the ledger header, created only when missing.
4. **Global links.** The eight links above. A link that already points to the
   repository is accepted. Anything else at the destination is a `FAIL`, and it is
   kept.
5. **Record.** Only when every check passes does the block write the record file
   with the repository root in it. While a check fails, no record is written, so
   the next session runs the block again until the failure is fixed.
6. **Result.** Every check prints `PASS`, `FAIL`, or `NOTE`. A failure never closes
   a terminal, because the block runs in a subshell.

## Safety

- The writes are the state root, the ledger header when missing, the record, and
  the eight authorized links. Nothing in the repository is modified.
- The block does not edit any shell profile, does not set environment variables,
  and does not change git configuration.
- A rerun changes nothing that is already in place.
- Existing configuration, links, and Skills are never overwritten.

## Platform notes

- **Windows:** symbolic links need Developer Mode or an elevated shell. Without
  them the block reports a `FAIL` and writes no record, so the first session
  keeps reporting it until Developer Mode is enabled. The client can instead be
  pointed at the repository path, but no rule depends on that workaround.
- **Windows line endings:** with `core.autocrlf=true`, a pasted block can keep a
  carriage return at the end of each line. Before cloning on Windows, run once
  `git config --global core.autocrlf input`. The block does not run this command,
  because it changes global git configuration.

## Relative references

`AGENTS.md` cites its workflows relative to the repository. From another project,
an agent resolves them from the root that the global link points to:
`readlink "$HOME/.config/opencode/AGENTS.md"` gives the path of `AGENTS.md` in the
repository, and its directory is the root.

## Limits

- **macOS:** the block ran natively in a temporary HOME, including the link rules
  for existing and differing destinations.
- **Linux:** run in an Alpine container from a copy of the repository: first run,
  rerun, and run without git. Other distributions are `NOT VERIFIED`.
- **Windows:** simulated with command shims. A real run in Git Bash is `NOT
  VERIFIED`.
- Whether each client follows `AGENTS.md` and runs the first-run check on its own
  is `NOT VERIFIED` until each client is opened in the repository. The check is an
  instruction that the agent executes; it is not enforced by the client.
- The first session may need the shell permission of the client to run the block.
- The record shows the block ran. It does not prove discovery; `/system-check`
  inside each client does.

## Related

- `references/portability.md` — the portability contract this workflow serves.
- `workflows/system-validation.md` — the structural check of the repository.
- `commands/system-check.md` — the discovery check run after bootstrap.
