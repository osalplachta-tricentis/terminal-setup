# Tmux Claude Status Integration

This guide documents the Oh My Tmux integration that shows Claude Code CLI context usage in the right status bar.

It is implemented without modifying the upstream Oh My Tmux config:

- live main config: `~/.config/tmux/tmux.conf`
- live local overrides: `~/.config/tmux/tmux.conf.local`
- live helper script: `~/.local/bin/tmux-claude-status`
- tracked script copy: `~/dev/terminal-setup/scripts/tmux-claude/tmux-claude-status`

## Current behavior

The status bar shows a Claude segment only when the **focused tmux pane** has an active Claude Code CLI session.

When active, the segment format is:

```text
CTX 35k/200k ▂ 18%
```

Behavior details:

- `CTX ...` comes from the latest API usage recorded in the session's JSONL conversation file
- context = `input_tokens + cache_creation_input_tokens + cache_read_input_tokens` from the last API response
- the context window is fixed at 200k tokens (all current Claude models)
- the percentage includes an 8-step vertical meter: `▁▂▃▄▅▆▇█`
- meter colors are thresholded: under 50% green, 50-70% orange, 70%+ red
- switching panes or windows updates the displayed context to follow the focused pane
- if the focused pane is not running Claude Code CLI, the entire segment is hidden
- the segment uses the black status background, matching the Copilot segment style

## Why pane-aware matching is needed

When multiple Claude Code CLI sessions are open at the same time, a global "latest session wins" strategy is misleading. The helper accepts tmux `#{pane_pid}` and resolves the active Claude process for that pane by checking:

1. whether the Claude PID shares the pane shell's session leader
2. or whether the Claude process is a descendant of the pane process

## Data sources

### Context usage

The helper reads:

- `~/.claude/sessions/[pid].json` to find active Claude sessions and their `sessionId` + `cwd`
- `~/.claude/projects/[cwd-as-path]/[sessionId].jsonl` for the conversation history

It extracts the latest API response entry from the JSONL, which contains:

```json
"usage": {
    "input_tokens": ...,
    "cache_creation_input_tokens": ...,
    "cache_read_input_tokens": ...,
    "output_tokens": ...
}
```

Total context = `input_tokens + cache_creation_input_tokens + cache_read_input_tokens`.

## Current `tmux.conf.local` integration

The active override on this machine is:

```tmux
tmux_conf_theme_status_right=" #{prefix}#{mouse}#{pairing}#{synchronized}#{?battery_status,#{battery_status},}#{?battery_vbar, #{battery_vbar},}#{?battery_percentage, #{battery_percentage},} | #(/home/ondra/.local/bin/tmux-copilot-status '#{pane_pid}') | #(/home/ondra/.local/bin/tmux-claude-status '#{pane_pid}') | %R , %d %b | #{username}#{root} @ #{hostname} "

tmux_conf_theme_status_right_fg="$tmux_conf_theme_colour_12,$tmux_conf_theme_colour_13,$tmux_conf_theme_colour_13,$tmux_conf_theme_colour_13,$tmux_conf_theme_colour_14"
tmux_conf_theme_status_right_bg="$tmux_conf_theme_colour_15,$tmux_conf_theme_colour_15,$tmux_conf_theme_colour_15,$tmux_conf_theme_colour_16,$tmux_conf_theme_colour_17"
tmux_conf_theme_status_right_attr="none,none,none,none,bold"
```

Both Copilot and Claude segments are their own `| ... |` segments with the black background (`colour_15`). The fg/bg arrays have 5 entries: battery, Copilot, Claude, time/date, username.

## Install or restore from the tracked script copy

```bash
mkdir -p ~/.local/bin
install -m 755 ~/dev/terminal-setup/scripts/tmux-claude/tmux-claude-status ~/.local/bin/tmux-claude-status
tmux source-file ~/.config/tmux/tmux.conf
```

## Verify

```bash
~/.local/bin/tmux-claude-status "$(tmux display-message -p '#{pane_pid}')"
tmux show -gv status-right
```
