# WSL Clipboard Integration for tmux + Claude Code + Copilot CLI

This guide makes mouse-drag copy and paste land on the **Windows clipboard** when you live in:

- WSL2 Ubuntu/Debian
- a host terminal that does **not** speak OSC 52 (e.g. the VS Code Remote-WSL integrated terminal)
- tmux 3.x with mouse mode enabled
- inner TUIs like Claude Code and GitHub Copilot CLI

It also re-enables Copilot CLI's built-in `/copy` action and double-click word selection.

If your host terminal is Windows Terminal, WezTerm, Ghostty, or anything else that already understands OSC 52, the steps below still apply and are harmless — they add a second, independent path to the clipboard via `clip.exe`.

## What's broken without this guide

- tmux's mouse drag-end / double-click / triple-click run `copy-pipe-and-cancel` with no destination, so the selection only lands in tmux's internal paste buffer, never the Windows clipboard.
- `set-clipboard` defaults to `external`, which means tmux **does not** emit OSC 52 itself — it only forwards OSC 52 from inner apps. And even if it did, VS Code's integrated terminal does not consume OSC 52 ([microsoft/vscode#193508](https://github.com/microsoft/vscode/issues/193508)).
- Copilot CLI's WSL clipboard backend shells out to `clip.exe`. The vscode-server shells launched by VS Code Remote-WSL **do not inherit Windows PATH**, so `command -v clip.exe` fails and Copilot CLI's `/copy`, double-click word select, and `Ctrl+Shift+C` actions silently no-op ([github/copilot-cli#2143](https://github.com/github/copilot-cli/issues/2143), [#2082](https://github.com/github/copilot-cli/issues/2082)).
- VS Code's terminal Ctrl+C is "copy if there is a terminal-level selection, else SIGINT". Selections drawn by tmux/Claude/Copilot live inside the screen buffer and xterm.js never sees them, so Ctrl+C falls through to SIGINT, which Claude/Copilot interpret as cancel. The only thing that "works" is **Shift**+drag, because Shift bypasses tmux/inner-app mouse capture and hands the selection to xterm.js natively — that's what makes the Ctrl+C copy path light up.

## Prerequisites

- WSL2 with Windows interop enabled (default)
- `/mnt/c/Windows/System32/clip.exe` exists and is executable from WSL — verify with:

```bash
test -x /mnt/c/Windows/System32/clip.exe && echo "clip.exe reachable"
printf 'wsl-clipboard-probe' | /mnt/c/Windows/System32/clip.exe && echo "clip.exe pipe ok"
```

- tmux installed and using `~/.config/tmux/tmux.conf.local` for local overrides (see [`oh_my_tmux_installation.md`](oh_my_tmux_installation.md))

## Step A — Wire tmux to clip.exe

Append to `~/.config/tmux/tmux.conf.local` in the user-customizations section (after the existing `set -g mouse on` line). The block is idempotent.

```bash
LOCAL_CONF="${XDG_CONFIG_HOME:-$HOME/.config}/tmux/tmux.conf.local"

if ! grep -q '^# WSL: pipe all tmux copies' "$LOCAL_CONF"; then
  cat >> "$LOCAL_CONF" <<'TMUX'

# WSL: pipe all tmux copies (mouse drag-end, double/triple-click, prefix+y)
# to the Windows clipboard via clip.exe. Absolute path so it works even when
# /mnt/c/Windows/System32 is missing from $PATH (vscode-server shells).
if -b 'test -x /mnt/c/Windows/System32/clip.exe' \
  'set -s copy-command "/mnt/c/Windows/System32/clip.exe"'

# Forward OSC 52 from inner apps to the outer terminal. No-op in VS Code's
# integrated terminal (which doesn't consume OSC 52), useful in Windows
# Terminal / WezTerm / Ghostty / etc.
set -g set-clipboard on
set -as terminal-features ',tmux-256color:clipboard'
set -as terminal-features ',xterm-256color:clipboard'
TMUX
fi
```

Why `copy-command` rather than rebinding every mouse key: in tmux 3.2+, `copy-pipe`, `copy-pipe-no-clear`, and `copy-pipe-and-cancel` all default to the `copy-command` server option when invoked without an argument. Setting it once covers the default mouse drag-end, double-click word, triple-click line, and `prefix + y` bindings without touching them.

Reload tmux so the new options take effect:

```bash
tmux source-file ~/.config/tmux/tmux.conf
```

## Step B — Make `clip.exe` discoverable on `$PATH`

Copilot CLI (and any other Linux app that uses `command -v clip.exe`) needs `clip.exe` reachable by name. vscode-server shells start with a clean `$PATH` that omits Windows interop directories, so add it explicitly. Append to `~/.zshrc` (and/or `~/.bashrc` if you also use bash):

```bash
SHELL_RC="$HOME/.zshrc"
if ! grep -q 'WSL: ensure clip.exe' "$SHELL_RC"; then
  cat >> "$SHELL_RC" <<'SH'

# WSL: ensure clip.exe (and other Windows tools) is on PATH so Copilot CLI
# and other Linux apps can shell out to the Windows clipboard. vscode-server
# does not inherit Windows PATH the way a normal wsl.exe shell does.
case ":$PATH:" in
  *":/mnt/c/Windows/System32:"*) ;;
  *) [ -x /mnt/c/Windows/System32/clip.exe ] && export PATH="$PATH:/mnt/c/Windows/System32" ;;
esac
SH
fi
```

Open a new shell (or `exec zsh`) and confirm:

```bash
command -v clip.exe
```

## Step C (optional) — Make Ctrl+V paste in the VS Code terminal

Plain `Ctrl+V` is not a terminal paste convention; the standard is `Ctrl+Shift+V`, which VS Code already handles. If you really want bare `Ctrl+V`, add this to VS Code's `keybindings.json`:

```json
{
  "key": "ctrl+v",
  "command": "workbench.action.terminal.paste",
  "when": "terminalFocus"
}
```

Trade-off: you lose `Ctrl+V` as the literal-insert / word-erase shortcut inside inner apps. `Ctrl+Shift+V` is the safer habit.

## Verify

After reload + new shell:

```bash
# tmux options are set
tmux show-options -s copy-command            # → /mnt/c/Windows/System32/clip.exe
tmux show-options -g set-clipboard           # → on
tmux show-options -gs terminal-features | grep -E 'tmux-256color|xterm-256color'

# clip.exe is on PATH
command -v clip.exe

# direct pipe round-trips
printf 'verify-tmux-clipboard' | /mnt/c/Windows/System32/clip.exe
```

End-to-end test, fully scripted, asserts that tmux's copy path actually reaches Windows:

```bash
tmux new-session -d -s clip-test -x 120 -y 30 'sleep 60'
PID=$(tmux list-panes -t clip-test -F "#{pane_id}")
printf 'PRE_TEST_SENTINEL' | /mnt/c/Windows/System32/clip.exe
tmux send-keys -t "$PID" "echo TMUX_COPY_PROBE_$$" Enter
sleep 0.3
tmux copy-mode -t "$PID"
tmux send-keys -t "$PID" -X cursor-up
tmux send-keys -t "$PID" -X cursor-up
tmux send-keys -t "$PID" -X select-line
tmux send-keys -t "$PID" -X copy-pipe-and-cancel
sleep 0.4
/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -NoProfile -Command "Get-Clipboard"
tmux kill-session -t clip-test
```

Expected: the final `Get-Clipboard` prints the `echo TMUX_COPY_PROBE_…` line, **not** `PRE_TEST_SENTINEL`.

## Usage notes — how to actually copy from each context

The wiring above is necessary but not sufficient — you also need to know which mouse drag actually reaches tmux.

### From a shell prompt pane (no TUI running)

Just drag with the mouse. tmux auto-enters copy mode on drag, copies on release, and `clip.exe` writes to Windows. Works.

### From inside Claude Code / Copilot CLI / vim / less / any TUI that captures the mouse

Plain mouse drag goes to the **inner app**, not tmux — `mouse_any_flag` is set, so tmux's root `MouseDrag1Pane` binding forwards the event with `send-keys -M`. tmux's copy path never fires. Two options:

1. **Enter tmux copy mode first** with `prefix + [`, then mouse-drag normally. Copy mode "freezes" the pane and routes mouse events to tmux instead of the inner app. Drag-end → `clip.exe` → Windows clipboard.
2. **Use the inner app's own copy action**:
   - **Copilot CLI**: its `/copy` slash command and double-click word selection both shell out to `clip.exe`. They start working as soon as Step B is in place. If they don't, check `command -v clip.exe` from a fresh shell.
   - **Claude Code**: no native copy action — use option 1 (enter tmux copy mode) or Shift+drag.

### Shift+drag — the universal escape hatch

Holding Shift while dragging tells xterm.js (the host terminal's terminal layer) to bypass mouse reporting entirely. The selection becomes a host-terminal-level selection, and the host's normal `Ctrl+C` / right-click / "copy on select" handles it. This works everywhere regardless of inner-app mouse capture, but it does **not** go through tmux's `copy-command`, so it does not depend on (and is not affected by) anything in this guide.

## Troubleshooting

- **`clip.exe` round-trip works but mouse drag doesn't change the clipboard.** You're dragging in a pane where an inner TUI captures the mouse. Enter `prefix + [` first, then drag. Confirm with `tmux display-message -p '#{mouse_any_flag}'` — if it prints `1`, mouse is captured by the inner app.
- **Copilot CLI's `/copy` does nothing.** `command -v clip.exe` from inside a Copilot CLI parent shell — if it returns nothing, Step B didn't load. Restart the shell that launched Copilot CLI.
- **`Ctrl+C` in Claude Code cancels instead of copying.** Expected. Use mouse drag (after entering tmux copy mode) or Shift+drag.
- **Garbage like `]52;c;…` printed to the screen after copying.** Your host terminal doesn't understand the OSC 52 sequence tmux now emits because of `set-clipboard on`. Either upgrade the terminal or, if you're stuck on it, revert `set-clipboard on` to `set-clipboard external`. The `clip.exe` path keeps working either way.

## References

- [tmux Clipboard wiki](https://github.com/tmux/tmux/wiki/Clipboard)
- [github/copilot-cli#2143 — clip.exe PATH shadowing](https://github.com/github/copilot-cli/issues/2143)
- [github/copilot-cli#2082 — `ctrl+shift+c` regression](https://github.com/github/copilot-cli/issues/2082)
- [github/copilot-cli#2159 — alt-screen + mouse breaks select-to-copy](https://github.com/github/copilot-cli/issues/2159)
- [microsoft/vscode#193508 — OSC 52 in integrated terminal (still open)](https://github.com/microsoft/vscode/issues/193508)
- [microsoft/terminal#2946 — Windows Terminal OSC 52 support](https://github.com/microsoft/terminal/issues/2946)
