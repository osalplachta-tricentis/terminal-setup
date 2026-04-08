# Tmux Copilot Status Integration

This guide documents the current Oh My Tmux integration that shows GitHub Copilot CLI context usage and monthly premium quota in the right status bar.

It is implemented without modifying the upstream Oh My Tmux config:

- live main config: `~/.config/tmux/tmux.conf`
- live local overrides: `~/.config/tmux/tmux.conf.local`
- live helper script: `~/.local/bin/tmux-copilot-status`
- tracked script copy: `~/dev/terminal-setup/scripts/tmux-copilot/tmux-copilot-status`

## Current behavior

The status bar shows a Copilot segment only when the **focused tmux pane** is running GitHub Copilot CLI.

When active, the segment format is:

```text
CTX 78k/305k ▃ 26% | Quota 57%
```

Behavior details:

- `CTX ...` comes from the focused pane's active Copilot session log
- the display total adds a 32k headroom to the compaction budget so it tracks `/context` more closely
- the context percentage includes an 8-step vertical meter: `▁▂▃▄▅▆▇█`
- meter colors are thresholded: under 50% green, 50-70% orange, 70%+ red
- `Quota ...` comes from `gh api /copilot_internal/user`
- quota is only shown when a matching Copilot session is active in the focused pane
- switching panes or windows updates the displayed Copilot context to follow the focused pane
- if the focused pane is not running Copilot CLI, the entire Copilot segment is hidden
- the Copilot segment uses the black status background, while time/date keep the red background

## Why pane-aware matching is needed

When multiple Copilot CLI sessions are open at the same time, a global "latest log wins" strategy is misleading. The current helper instead accepts tmux `#{pane_pid}` and resolves the active Copilot process for that pane by checking:

1. whether the Copilot PID shares the pane shell's session leader
2. or whether the Copilot process is a descendant of the pane process

That makes the status line follow the Copilot instance in the currently focused pane instead of some unrelated pane.

## Data sources

### Context usage

The helper reads:

- lock files from `~/.copilot/session-state/*/inuse.*.lock`
- Copilot logs from `~/.copilot/logs/process-*-PID.log`

It extracts the latest line matching:

```text
CompactionProcessor: Utilization X% (used/total tokens)
```

### Monthly quota

The helper reads:

```bash
gh api /copilot_internal/user
```

It uses `quota_snapshots.premium_interactions` and renders only the used percentage as `Quota xx%`.

The API response is cached for 5 minutes in:

```bash
~/.cache/tmux-copilot-status/quota.json
```

This avoids hitting `gh api` on every tmux status redraw.

## Required auth

If the quota endpoint is unavailable, refresh GitHub CLI auth with:

```bash
gh auth refresh -h github.com -s user
```

## Current `tmux.conf.local` integration

The active override on this machine is:

```tmux
tmux_conf_theme_status_right=" #{prefix}#{mouse}#{pairing}#{synchronized}#{?battery_status,#{battery_status},}#{?battery_vbar, #{battery_vbar},}#{?battery_percentage, #{battery_percentage},} | #(/home/ondra/.local/bin/tmux-copilot-status '#{pane_pid}') | #(/home/ondra/.local/bin/tmux-claude-status '#{pane_pid}') | %R , %d %b | #{username}#{root} @ #{hostname} "

tmux_conf_theme_status_right_fg="$tmux_conf_theme_colour_12,$tmux_conf_theme_colour_13,$tmux_conf_theme_colour_13,$tmux_conf_theme_colour_13,$tmux_conf_theme_colour_14"
tmux_conf_theme_status_right_bg="$tmux_conf_theme_colour_15,$tmux_conf_theme_colour_15,$tmux_conf_theme_colour_15,$tmux_conf_theme_colour_16,$tmux_conf_theme_colour_17"
tmux_conf_theme_status_right_attr="none,none,none,none,bold"
```

Note: the Claude status segment was added alongside Copilot (see `tmux_claude_status_integration.md`). The fg/bg arrays now have 5 entries: battery, Copilot, Claude, time/date, username.

Important detail: in Oh My Tmux, major right-side theme segments are split with `|`, not commas. That is why the Copilot and Claude blocks must each be their own `| ... |` segment to keep the black background separate from the red time/date segment.

## Install or restore from the tracked script copy

```bash
mkdir -p ~/.local/bin
install -m 755 ~/dev/terminal-setup/scripts/tmux-copilot/tmux-copilot-status ~/.local/bin/tmux-copilot-status
tmux source-file ~/.config/tmux/tmux.conf
```

## Verify

```bash
~/.local/bin/tmux-copilot-status "$(tmux display-message -p '#{pane_pid}')"
tmux show -gv status-right
```
