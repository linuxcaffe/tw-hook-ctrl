#!/usr/bin/env python3
"""
hook-ctrl — ncurses TUI for toggling Taskwarrior hook permissions
Part of the awesome-taskwarrior suite.

Usage:
  hook-ctrl              # use hooks dir from active TASKRC
  hook-ctrl --dir PATH   # specify hooks directory
  hook-ctrl --help
"""

import curses
import os
import stat
import subprocess
import sys
from pathlib import Path

VERSION = "0.1.0"

HOOK_TYPES = {
    'on-add':    ('on-add   ', 1),
    'on-modify': ('on-modify', 2),
    'on-exit':   ('on-exit  ', 3),
    'on-launch': ('on-launch', 4),
}

# Color pair indices
CP_ENABLED  = 1   # green  — active hook
CP_DISABLED = 2   # red    — inactive hook
CP_ADD      = 3   # cyan   — on-add
CP_MODIFY   = 4   # yellow — on-modify
CP_EXIT     = 5   # magenta — on-exit
CP_LAUNCH   = 6   # blue   — on-launch
CP_SELECTED = 7   # reverse — cursor row
CP_HEADER   = 8   # bold white
CP_STATUS   = 9   # status bar
CP_SYMLINK  = 10  # dim — symlink indicator
CP_UNRECOG  = 11  # dim — unrecognized hook name


def get_hooks_dir(override=None):
    """Return the hooks directory Path."""
    if override:
        return Path(override).expanduser()
    try:
        result = subprocess.run(
            ['task', '_get', 'rc.data.location'],
            capture_output=True, text=True, timeout=5
        )
        loc = result.stdout.strip()
        if loc:
            return Path(os.path.expanduser(loc)) / 'hooks'
    except Exception:
        pass
    return Path.home() / '.task' / 'hooks'


def hook_type(name):
    """Return (label, color_pair_index, is_recognized) for a hook filename."""
    for prefix, (label, cp) in HOOK_TYPES.items():
        if name.startswith(prefix):
            return label, cp, True
    return '?        ', CP_UNRECOG, False


def load_hooks(hooks_dir):
    """Return sorted list of dicts: {path, name, executable, symlink, recognized}"""
    hooks = []
    if not hooks_dir.exists():
        return hooks
    for f in sorted(hooks_dir.iterdir()):
        if not f.is_file() and not f.is_symlink():
            continue
        if f.name.startswith('.'):
            continue
        is_exec = os.access(f, os.X_OK)
        is_sym  = f.is_symlink()
        _, _, recognized = hook_type(f.name)
        hooks.append({
            'path':       f,
            'name':       f.name,
            'executable': is_exec,
            'symlink':    is_sym,
            'recognized': recognized,
        })
    return hooks


def toggle(hook):
    """Toggle executable bit on a hook file."""
    path = hook['path']
    m = path.stat().st_mode
    if m & stat.S_IXUSR:
        path.chmod(m & ~(stat.S_IXUSR | stat.S_IXGRP | stat.S_IXOTH))
    else:
        path.chmod(m | (stat.S_IXUSR | stat.S_IXGRP | stat.S_IXOTH))


def set_all(hooks, enable):
    """Enable or disable all hooks."""
    for h in hooks:
        m = h['path'].stat().st_mode
        if enable:
            h['path'].chmod(m | (stat.S_IXUSR | stat.S_IXGRP | stat.S_IXOTH))
        else:
            h['path'].chmod(m & ~(stat.S_IXUSR | stat.S_IXGRP | stat.S_IXOTH))


def draw(stdscr, hooks, cursor, scroll, hooks_dir, message=''):
    stdscr.erase()
    h, w = stdscr.getmaxyx()

    active = sum(1 for hk in hooks if hk['executable'])
    total  = len(hooks)

    # ── Header ───────────────────────────────────────────────────────────────
    hdr_left  = f"  hook-ctrl v{VERSION}  {hooks_dir}"
    hdr_right = f"{active}/{total} active  "
    stdscr.addstr(0, 0, hdr_left[:w-1], curses.color_pair(CP_HEADER))
    if len(hdr_right) < w:
        stdscr.addstr(0, w - len(hdr_right), hdr_right, curses.color_pair(CP_HEADER))

    # ── Column labels ─────────────────────────────────────────────────────────
    stdscr.addstr(1, 2, f"  {'TYPE':<11} {'ST':5}  FILENAME", curses.A_DIM)

    # ── Hook list ─────────────────────────────────────────────────────────────
    list_top    = 2
    list_height = h - 4   # header + col labels + status bar + 1 spare

    for i, hk in enumerate(hooks):
        if i < scroll or i >= scroll + list_height:
            continue
        row = list_top + (i - scroll)
        if row >= h - 1:
            break

        selected    = (i == cursor)
        is_exec     = hk['executable']
        label, type_cp, recognized = hook_type(hk['name'])

        # cursor arrow
        arrow = ' ► ' if selected else '   '
        stdscr.addstr(row, 0, arrow,
                      curses.color_pair(CP_SELECTED) if selected else 0)

        # type tag
        type_attr = curses.color_pair(CP_SELECTED) if selected else curses.color_pair(type_cp)
        if not recognized:
            type_attr = curses.color_pair(CP_UNRECOG) | curses.A_DIM
        stdscr.addstr(row, 3, f"[{label}]", type_attr)

        # status dot
        if is_exec:
            sdot, sattr = ' ● ', curses.color_pair(CP_ENABLED) | curses.A_BOLD
        else:
            sdot, sattr = ' ○ ', curses.color_pair(CP_DISABLED)
        if selected:
            sattr = curses.color_pair(CP_SELECTED)
        stdscr.addstr(row, 14, sdot, sattr)

        # filename (+ symlink marker)
        name = hk['name']
        if hk['symlink']:
            name += ' →'
        max_name = w - 19
        name_attr = curses.color_pair(CP_SELECTED) if selected else 0
        if not recognized:
            name_attr = curses.color_pair(CP_UNRECOG) | curses.A_DIM
        stdscr.addstr(row, 18, name[:max_name], name_attr)

    # ── Status bar ────────────────────────────────────────────────────────────
    if message:
        bar = f"  {message}"
    else:
        bar = "  ↑↓/jk navigate   SPACE toggle   e enable-all   d disable-all   r refresh   q quit"
    stdscr.addstr(h - 1, 0, bar[:w - 1].ljust(w - 1), curses.color_pair(CP_STATUS))

    stdscr.refresh()


def init_colors():
    curses.start_color()
    curses.use_default_colors()
    curses.init_pair(CP_ENABLED,  curses.COLOR_GREEN,   -1)
    curses.init_pair(CP_DISABLED, curses.COLOR_RED,     -1)
    curses.init_pair(CP_ADD,      curses.COLOR_CYAN,    -1)
    curses.init_pair(CP_MODIFY,   curses.COLOR_YELLOW,  -1)
    curses.init_pair(CP_EXIT,     curses.COLOR_MAGENTA, -1)
    curses.init_pair(CP_LAUNCH,   curses.COLOR_BLUE,    -1)
    curses.init_pair(CP_SELECTED, curses.COLOR_BLACK,   curses.COLOR_CYAN)
    curses.init_pair(CP_HEADER,   curses.COLOR_WHITE,   curses.COLOR_BLUE)
    curses.init_pair(CP_STATUS,   curses.COLOR_BLACK,   curses.COLOR_GREEN)
    curses.init_pair(CP_SYMLINK,  curses.COLOR_WHITE,   -1)
    curses.init_pair(CP_UNRECOG,  curses.COLOR_WHITE,   -1)


def run(stdscr, hooks_dir):
    curses.curs_set(0)
    init_colors()
    stdscr.keypad(True)

    hooks   = load_hooks(hooks_dir)
    cursor  = 0
    scroll  = 0
    message = ''

    if not hooks:
        stdscr.addstr(0, 0, f"No hooks found in {hooks_dir}")
        stdscr.getch()
        return

    while True:
        h, _ = stdscr.getmaxyx()
        list_height = h - 4

        # Keep scroll window around cursor
        if cursor < scroll:
            scroll = cursor
        elif cursor >= scroll + list_height:
            scroll = cursor - list_height + 1

        draw(stdscr, hooks, cursor, scroll, hooks_dir, message)
        message = ''

        key = stdscr.getch()

        if key in (ord('q'), ord('Q'), 27):
            break

        elif key in (curses.KEY_UP, ord('k')):
            if cursor > 0:
                cursor -= 1

        elif key in (curses.KEY_DOWN, ord('j')):
            if cursor < len(hooks) - 1:
                cursor += 1

        elif key == ord('g'):
            cursor = 0

        elif key == ord('G'):
            cursor = len(hooks) - 1

        elif key in (ord(' '), 10, 13):  # space / enter
            toggle(hooks[cursor])
            hooks = load_hooks(hooks_dir)
            name = hooks[cursor]['name'] if cursor < len(hooks) else ''
            state = 'enabled' if hooks[cursor]['executable'] else 'disabled'
            message = f"{name} → {state}"

        elif key == ord('e'):
            set_all(hooks, True)
            hooks = load_hooks(hooks_dir)
            message = f"All {len(hooks)} hooks enabled"

        elif key == ord('d'):
            set_all(hooks, False)
            hooks = load_hooks(hooks_dir)
            message = f"All {len(hooks)} hooks disabled"

        elif key == ord('r'):
            hooks  = load_hooks(hooks_dir)
            cursor = min(cursor, len(hooks) - 1)
            message = "Refreshed"

        elif key == curses.KEY_RESIZE:
            pass  # redraw on next loop


def main():
    import argparse
    parser = argparse.ArgumentParser(
        description=f"hook-ctrl v{VERSION} — toggle Taskwarrior hook permissions"
    )
    parser.add_argument('--dir', metavar='PATH',
                        help='hooks directory (default: from active TASKRC)')
    parser.add_argument('--version', action='version', version=f'%(prog)s {VERSION}')
    args = parser.parse_args()

    hooks_dir = get_hooks_dir(args.dir)

    if not hooks_dir.exists():
        print(f"Hooks directory not found: {hooks_dir}", file=sys.stderr)
        sys.exit(1)

    try:
        curses.wrapper(run, hooks_dir)
    except KeyboardInterrupt:
        pass


if __name__ == '__main__':
    main()
