# tmux Installation

Official project: <https://github.com/tmux/tmux>

## Install or confirm

```bash
sudo apt update
sudo apt install -y tmux
```

## Minimal baseline

`tmux` itself does not need a custom config before first use. This repository keeps the theming, local overrides, and plugin setup in the separate Oh My Tmux flow:

- [Oh My Tmux installation](oh_my_tmux_installation.md)
- [`.tmux.conf.local` overrides](tmux_conf_local_overrides.md)

The reviewed machine uses the XDG layout:

- `~/.config/tmux/tmux.conf`
- `~/.config/tmux/tmux.conf.local`
- `~/.config/tmux/plugins/`

## Verify

```bash
tmux -V
```

## First launch

```bash
tmux new -As main
```

Detach with `Ctrl+b` then `d`.
