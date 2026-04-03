# `.tmux.conf.local` Overrides for Oh My Tmux

Official reference: <https://github.com/gpakosz/.tmux>

Upstream is explicit about this: keep the shipped main config untouched and put your changes in the local override file.

For the reviewed machine, that file is:

```bash
~/.config/tmux/tmux.conf.local
```

## Current plugin setup on this machine

The active plugin setup is TPM-backed and currently enables:

```tmux
set -g @plugin 'omerxx/tmux-floax'
```

Oh My Tmux manages TPM integration itself. Do **not** add:

```tmux
set -g @plugin 'tmux-plugins/tpm'
run '~/.tmux/plugins/tpm/tpm'
```

## Idempotent baseline for this setup

This snippet keeps the existing `tmux.conf.local` if present, creates it from upstream if missing, and appends the core settings used on the reviewed machine only when they are not already present.

```bash
XDG_CONFIG_HOME="${XDG_CONFIG_HOME:-$HOME/.config}"
TMUX_CONFIG_DIR="$XDG_CONFIG_HOME/tmux"
LOCAL_CONF="$TMUX_CONFIG_DIR/tmux.conf.local"
UPSTREAM_LOCAL="$HOME/.local/share/tmux/oh-my-tmux/.tmux.conf.local"

mkdir -p "$TMUX_CONFIG_DIR"
[ -f "$LOCAL_CONF" ] || cp "$UPSTREAM_LOCAL" "$LOCAL_CONF"

grep -q "^tmux_conf_new_pane_retain_current_path=true$" "$LOCAL_CONF" || \
  printf "\ntmux_conf_new_pane_retain_current_path=true\n" >> "$LOCAL_CONF"

grep -q "^tmux_conf_copy_to_os_clipboard=true$" "$LOCAL_CONF" || \
  printf "tmux_conf_copy_to_os_clipboard=true\n" >> "$LOCAL_CONF"

grep -q "^set -g mouse on$" "$LOCAL_CONF" || \
  printf "set -g mouse on\n" >> "$LOCAL_CONF"

grep -q "^tmux_conf_update_plugins_on_launch=true$" "$LOCAL_CONF" || \
  printf "tmux_conf_update_plugins_on_launch=true\n" >> "$LOCAL_CONF"

grep -q "^tmux_conf_update_plugins_on_reload=true$" "$LOCAL_CONF" || \
  printf "tmux_conf_update_plugins_on_reload=true\n" >> "$LOCAL_CONF"

grep -q "^tmux_conf_uninstall_plugins_on_reload=true$" "$LOCAL_CONF" || \
  printf "tmux_conf_uninstall_plugins_on_reload=true\n" >> "$LOCAL_CONF"

grep -q "^set -g @plugin 'omerxx/tmux-floax'$" "$LOCAL_CONF" || \
  printf "set -g @plugin 'omerxx/tmux-floax'\n" >> "$LOCAL_CONF"

grep -q "^set -g visual-bell off$" "$LOCAL_CONF" || \
  printf "set -g visual-bell off\n" >> "$LOCAL_CONF"
```

## What those settings do

- keep pane splits in the current directory
- copy selections to the OS clipboard
- start with mouse mode enabled
- let Oh My Tmux auto-install and update TPM plugins
- enable the `omerxx/tmux-floax` popup/scratchpad plugin
- disable visual bell noise

## Optional terminal color overrides

If your terminal supports 24-bit color and your terminfo has `tmux-256color`, add these lines only if you actually need them:

```tmux
set -g default-terminal "tmux-256color" #!important
set -as terminal-features ',xterm-256color:RGB'
```

If that causes issues in your terminal, remove those two lines and fall back to the upstream defaults.

## Apply changes

Inside tmux:

```bash
tmux source-file ~/.config/tmux/tmux.conf
```

## Verify

```bash
grep -n "tmux_conf_update_plugins_on_launch\|tmux_conf_update_plugins_on_reload\|@plugin 'omerxx/tmux-floax'\|visual-bell off" ~/.config/tmux/tmux.conf.local
find ~/.config/tmux/plugins -maxdepth 1 -mindepth 1 -type d | sort
```
