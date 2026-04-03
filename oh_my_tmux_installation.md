# Oh My Tmux Installation

Official project: <https://github.com/gpakosz/.tmux>

This guide uses the XDG-style layout that is already present on the reviewed machine:

- upstream checkout: `~/.local/share/tmux/oh-my-tmux`
- main config symlink: `~/.config/tmux/tmux.conf`
- local overrides: `~/.config/tmux/tmux.conf.local`
- plugins: `~/.config/tmux/plugins`

That layout is safe to re-run and keeps local overrides separate from the upstream checkout.

## Prerequisites

```bash
sudo apt update
sudo apt install -y tmux git curl
```

Oh My Tmux expects your terminal outside tmux to use `xterm-256color`. If your terminal still reports `xterm-color`, adjust that in your terminal profile before troubleshooting the theme.

## Install or update

```bash
XDG_CONFIG_HOME="${XDG_CONFIG_HOME:-$HOME/.config}"
TMUX_CONFIG_DIR="$XDG_CONFIG_HOME/tmux"
OH_MY_TMUX_DIR="$HOME/.local/share/tmux/oh-my-tmux"

mkdir -p "$TMUX_CONFIG_DIR" "$(dirname "$OH_MY_TMUX_DIR")"

if [ ! -d "$OH_MY_TMUX_DIR/.git" ]; then
  git clone --single-branch https://github.com/gpakosz/.tmux.git "$OH_MY_TMUX_DIR"
else
  git -C "$OH_MY_TMUX_DIR" pull --ff-only
fi

ln -sfn "$OH_MY_TMUX_DIR/.tmux.conf" "$TMUX_CONFIG_DIR/tmux.conf"

if [ ! -f "$TMUX_CONFIG_DIR/tmux.conf.local" ]; then
  cp "$OH_MY_TMUX_DIR/.tmux.conf.local" "$TMUX_CONFIG_DIR/tmux.conf.local"
else
  echo "$TMUX_CONFIG_DIR/tmux.conf.local already exists; keeping your copy"
fi
```

## Minimal baseline

- Never edit the upstream `.tmux.conf` directly
- Put all custom changes in `~/.config/tmux/tmux.conf.local`
- If Oh My Tmux overrides a binding or option, add `#!important` to that line in `tmux.conf.local`
- Oh My Tmux already integrates TPM support; do **not** add `tmux-plugins/tpm` manually as a regular plugin

## Reload or start tmux

```bash
tmux source-file "${XDG_CONFIG_HOME:-$HOME/.config}/tmux/tmux.conf" 2>/dev/null || tmux new -As main
```

## Verify

```bash
test -d ~/.local/share/tmux/oh-my-tmux && echo "oh-my-tmux checkout present"
test -L ~/.config/tmux/tmux.conf && echo "~/.config/tmux/tmux.conf symlinked"
test -f ~/.config/tmux/tmux.conf.local && echo "~/.config/tmux/tmux.conf.local present"
```
