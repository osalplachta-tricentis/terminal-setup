# Fresh Editor Installation

Official pages:

- <https://getfresh.dev/>
- <https://github.com/sinelaw/fresh>

Fresh's upstream README advertises the install script as the quick-install path, and that is the best fit for this setup.

## Install with the official script

```bash
if command -v fresh >/dev/null 2>&1; then
  echo "fresh already installed: $(fresh --version)"
else
  curl https://raw.githubusercontent.com/sinelaw/fresh/refs/heads/master/scripts/install.sh | sh
fi
```

## Minimal baseline

Fresh is designed to work with zero configuration. The main requirement for this setup is that `~/.local/bin` stays on `PATH`, because that is where user-level tools commonly land.

The current shell docs in this repository already keep `~/.local/bin` on `PATH`.

## Launch

```bash
fresh
```

Open a file directly:

```bash
fresh path/to/file
```

## Verify

```bash
fresh --version
command -v fresh
```
