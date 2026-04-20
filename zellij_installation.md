# Zellij Installation

Official project: <https://github.com/zellij-org/zellij>

## Install or confirm

Ubuntu/Debian does not reliably provide a current `zellij` package, so use the upstream release binary.

```bash
arch="$(uname -m)"
case "$arch" in
  x86_64) target="x86_64-unknown-linux-musl" ;;
  aarch64|arm64) target="aarch64-unknown-linux-musl" ;;
  *) echo "Unsupported architecture: $arch" >&2; exit 1 ;;
esac

tmpdir="$(mktemp -d)"
trap 'rm -rf "$tmpdir"' EXIT

tag="$(curl -fsSL https://api.github.com/repos/zellij-org/zellij/releases/latest | sed -n 's/.*"tag_name": "\([^"]*\)".*/\1/p' | head -n1)"
asset="zellij-${target}.tar.gz"
curl -fL "https://github.com/zellij-org/zellij/releases/download/${tag}/${asset}" -o "$tmpdir/$asset"
tar -xzf "$tmpdir/$asset" -C "$tmpdir"

install -d "$HOME/.local/bin"
install -m 0755 "$tmpdir/zellij" "$HOME/.local/bin/zellij"
```

If you prefer a system-wide install and have passwordless `sudo` configured, replace the last two lines with:

```bash
sudo install -m 0755 "$tmpdir/zellij" /usr/local/bin/zellij
```

## Minimal baseline

`zellij` works without any extra theme or plugin framework. Start with the default layout and add configuration only after the user has used it enough to know what they want to change.

This repository keeps Zellij setup intentionally small:

- install the upstream binary
- keep it reachable through `~/.local/bin`
- avoid writing config until there is a clear user preference

## Verify

```bash
zellij --version
```

## First launch

```bash
zellij
```

Detach from a session with `Ctrl+o` then `d`.
