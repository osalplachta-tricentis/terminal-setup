# Midnight Commander Installation

Official project: <https://midnight-commander.org/>

## Install

```bash
sudo apt update
sudo apt install -y mc
```

## Minimal baseline

The first launch creates the user config under `~/.config/mc/`. After that you can adjust options from the built-in menus instead of editing files by hand.

Useful defaults to review after first launch:

- `Options -> Configuration`
- `Options -> Layout`
- `Options -> Panel options`

## Launch

```bash
mc
```

## Verify

```bash
mc --version | head -n 1
```
