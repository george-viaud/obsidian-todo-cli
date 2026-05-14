# obsidian-todo-cli

A tiny, deterministic CLI for appending tasks to an Obsidian vault in the
syntax of the [Obsidian Tasks plugin](https://publish.obsidian.md/tasks/).

## Why

The Tasks plugin is the dominant task-management plugin in the Obsidian
ecosystem, but it has no built-in CLI — task creation happens through an
in-app modal. If you want to add tasks from a terminal, a shell script,
a hotkey, or an AI agent, you end up either:

- writing inconsistently-formatted lines by hand (`echo "- [ ] ..." >>
  TODOs.md`), or
- learning the official Obsidian CLI's narrow task surface, which requires
  Obsidian to be running and is read-mostly today.

`todo` is the missing write-side: ~200 lines of Python stdlib, no
dependencies, produces byte-identical Tasks-plugin syntax every time.

## Install

```bash
git clone https://github.com/george-viaud/obsidian-todo-cli.git ~/src/obsidian-todo-cli
ln -s ~/src/obsidian-todo-cli/todo ~/.local/bin/todo
# make sure ~/.local/bin is on your PATH
```

Then set the vault path in your shell config (`~/.bashrc`, `~/.zshrc`, etc.):

```bash
export OBSIDIAN_VAULT="$HOME/path/to/your/vault"
```

In your vault, create `TODOs.md` containing — at minimum — the marker line:

```markdown
## Raw (CLI appends here)
```

Anything above the marker is left untouched (this is where you put your
Tasks-plugin query blocks). New tasks are appended below it.

Optionally, create an empty `inbox.md` for freeform mobile capture.

A starter template is included in [`vault-template/`](./vault-template/).

## Usage

```bash
todo add "Pick up dry cleaning" --tag life --due tomorrow
todo add "Review PR" --tag work --priority high --due +2d
todo add "Cancel that subscription" --tag life --tag finance
todo list                       # open tasks
todo list --tag life            # filter (AND across multiple --tag)
todo list --state done
todo list --state all
todo done 3                     # mark task #3 (index from `todo list`) done
todo inbox                      # interactively promote inbox.md bullets
todo --help
```

### Convention: first tag is the "domain"

The CLI does not enforce domains, but a useful convention is to make the
**first tag** the broad category (`#life`, `#work`, `#home`, `#side/xyz`,
etc.). Tasks-plugin query blocks at the top of `TODOs.md` can then surface
per-domain views without splitting your tasks across multiple files.

**Date forms** accepted by `--due`: `YYYY-MM-DD`, `today`, `tomorrow`, `+Nd`.

**Priorities:** `high` → ⏫, `medium` → 🔼, `low` → 🔽.

**Tags:** letters, digits, `-`, `_`, `/` only (the leading `#` is optional).

## Output format

Tasks are written as a single line per task, with a fixed token order for
byte-determinism across runs:

```
- [ ] <description> #task <other-tags...> <priority-emoji?> 📅 <due-date?>
```

When marked done, the line is rewritten with `[x]` and a `✅ YYYY-MM-DD`
suffix:

```
- [x] <description> #task <tags...> <priority?> 📅 <due?> ✅ 2026-05-14
```

## Configuration

| Env var | Default | Purpose |
|---|---|---|
| `OBSIDIAN_VAULT` | *(required)* | Absolute path to your vault |
| `OBSIDIAN_TODO_FILE` | `TODOs.md` | Filename inside the vault |
| `OBSIDIAN_INBOX_FILE` | `inbox.md` | Filename inside the vault |
| `OBSIDIAN_TASKS_FILTER` | `#task` | Tasks-plugin global filter tag |

The default `#task` filter matches the Tasks plugin's recommended setup. If
you don't use a global filter, set `OBSIDIAN_TASKS_FILTER=""` — but the
plugin will then treat every checkbox in your vault as a task.

## Inbox workflow

Mobile devices can't easily run a Python CLI, but they can append a line
to a markdown file from inside Obsidian. The convention here:

1. On phone: open `inbox.md`, append `- whatever you want to remember`.
   No tags, no format — just a bullet.
2. On desktop: run `todo inbox`. For each bullet you're prompted to
   `[p]romote / [k]eep / [d]rop / [q]uit`. Promotion asks for tags,
   due date, priority, then writes a structured line into `TODOs.md`
   and removes the bullet from `inbox.md`.

This preserves byte-determinism in `TODOs.md` (only the CLI writes there)
while keeping the mobile capture path frictionless.

## Companion: official Obsidian CLI for reads

If you have [Obsidian 1.12+ with the official CLI](https://obsidian.md/cli)
enabled (currently a Catalyst feature), it pairs well with `todo` as the
read-side counterpart:

```bash
obsidian tasks todo                                      # list open tasks (Obsidian-rendered)
obsidian tasks todo format=json | jq '.[] | select(.text | contains("#life"))'
obsidian tag name=#apex verbose                          # find file:line of every #apex tag
```

The official CLI doesn't add the Tasks-plugin `✅ YYYY-MM-DD` done-date
stamp when marking tasks done, and it requires the Obsidian GUI to be
running. `todo done <n>` does add the stamp and works headlessly — use it
for completion.

## License

MIT — see [LICENSE](./LICENSE).
