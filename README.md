- Project: https://github.com/linuxcaffe/tw-hook-ctrl
- Issues:  https://github.com/linuxcaffe/tw-hook-ctrl/issues

# hook-ctrl

A visual dashboard for seeing which Taskwarrior hooks are active and toggling them on or off.

## TL;DR

- Full-screen terminal UI showing every hook in your hooks directory at a glance
- Toggle any hook on or off with a single keypress — no `chmod` commands needed
- Hooks grouped by type in procedural order: launch → add → modify → exit
- Color-coded by hook type; enabled/disabled status shown with a clear indicator
- Enable or disable all hooks at once with `e` / `d`
- `--dir PATH` for any custom hooks directory
- Requires Taskwarrior 2.6.0+, Python 3.6+

## Why this exists

Taskwarrior hooks are enabled by the filesystem executable bit. To disable one, you
`chmod -x` the file; to re-enable it, you `chmod +x`. That works fine for one hook.
When you have a dozen, if you're troubleshooting, it stops being fine.

The `task diag` command lists hooks and shows their status, but doesn't provide
an easy way to engage or disengage them, beyond dropping to a`chmod` one-liner 
with `+x` or `-x` composed from scratch each time. 
This python curses tui makes it clear and simple to turn hooks on and off.

## Installation

### Option 1 — Install script

```bash
curl -fsSL https://raw.githubusercontent.com/linuxcaffe/tw-hook-ctrl/main/hook-ctrl.install | bash
```

Installs `hook-ctrl` to `~/.task/scripts/` and makes it executable.

### Option 2 — Via [awesome-taskwarrior](https://github.com/linuxcaffe/awesome-taskwarrior)

```bash
tw -I hook-ctrl
```

### Option 3 — Manual

```bash
# Download the script
curl -fsSL https://raw.githubusercontent.com/linuxcaffe/tw-hook-ctrl/main/hook-ctrl.py \
  -o ~/.task/scripts/hook-ctrl

# Make it executable
chmod +x ~/.task/scripts/hook-ctrl

# Verify
hook-ctrl --version
```

## Usage

```bash
hook-ctrl              # hooks from active TASKRC
hook-ctrl --dir PATH   # any hooks directory
td hook-ctrl           # dev hooks via TW_TASK_DIR — no flag needed
```

**Keys**

| Key | Action |
|-----|--------|
| `↑` `↓` or `j` `k` | Navigate |
| `Space` or `Enter` | Toggle selected hook on/off |
| `e` | Enable all hooks |
| `d` | Disable all hooks |
| `r` | Refresh (picks up external changes) |
| `g` / `G` | Jump to top / bottom |
| `q` or `Esc` | Quit |

**Display**

Each row shows hook type (color-coded), status (`●` enabled / `○` disabled),
and filename. Symlinked hooks are marked with `→`. Hooks are grouped by type
in procedural order — launch, add, modify, exit — so the sequence of execution
is visible at a glance. Unrecognized filenames appear dimmed at the bottom.

The header shows the active hooks directory and the enabled/total count.

## Example workflow

Diagnosing a slow `task add`:

```
1.  hook-ctrl                  # open the dev hooks dashboard
2.  Navigate to on-add hooks
3.  Space                      # disable the suspect hook
4.  q                          # quit
5.  task add "test task"       # time the operation
6.  hook-ctrl                  # re-open
7.  Space                      # re-enable when done
```

## Project status

⚠️  Early release (v0.1.0). Core toggle and display functionality is stable.
The following may change in future releases:

- Keyboard shortcuts
- Color scheme and display layout
- Configuration file support

## Further reading

- [awesome-taskwarrior](https://github.com/linuxcaffe/awesome-taskwarrior) — the
  ecosystem this tool belongs to, including the `tw` package manager
- [Taskwarrior hook documentation](https://taskwarrior.org/docs/hooks/) — the
  official hook API reference

## Metadata

- License: MIT
- Language: Python 3
- Requires: Taskwarrior 2.6.0+, Python 3.6+
- Platforms: Linux
- Version: 0.1.0
