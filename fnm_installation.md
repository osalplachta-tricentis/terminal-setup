# fnm Installation

Official project: <https://github.com/Schniz/fnm>

This setup uses `fnm` instead of `nvm`.

## Prerequisites

```bash
sudo apt update
sudo apt install -y curl unzip
```

## Install without auto-editing shell files

```bash
if command -v fnm >/dev/null 2>&1; then
  echo "fnm already installed: $(fnm --version)"
else
  curl -fsSL https://fnm.vercel.app/install | bash -s -- --skip-shell
fi
```

The installer defaults to an XDG-style location on Linux, commonly `~/.local/share/fnm`. Some setups still use `~/.fnm`, so the shell snippets below handle both.

## Bash setup

Add this to `~/.bashrc` if the matching lines are not already present:

```bash
export PATH="$HOME/.local/bin:$PATH"

if [ -d "$HOME/.fnm" ]; then
  export PATH="$HOME/.fnm:$PATH"
elif [ -n "$XDG_DATA_HOME" ] && [ -d "$XDG_DATA_HOME/fnm" ]; then
  export PATH="$XDG_DATA_HOME/fnm:$PATH"
elif [ -d "$HOME/.local/share/fnm" ]; then
  export PATH="$HOME/.local/share/fnm:$PATH"
fi

if command -v fnm >/dev/null 2>&1; then
  eval "$(fnm env --use-on-cd --shell bash)"
fi
```

## Zsh setup

Make sure `~/.zshrc` contains these lines once:

```zsh
plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
  zsh-autocomplete
  fnm
)

if [ -d "$HOME/.fnm" ]; then
  export PATH="$HOME/.fnm:$PATH"
elif [ -n "$XDG_DATA_HOME" ] && [ -d "$XDG_DATA_HOME/fnm" ]; then
  export PATH="$XDG_DATA_HOME/fnm:$PATH"
elif [ -d "$HOME/.local/share/fnm" ]; then
  export PATH="$HOME/.local/share/fnm:$PATH"
fi

source $ZSH/oh-my-zsh.sh

export PATH="$HOME/.local/bin:$PATH"

if command -v fnm >/dev/null 2>&1; then
  eval "$(fnm env --use-on-cd --shell zsh)"
fi
```

## Minimal baseline

Install an LTS Node.js release after `fnm` is ready:

```bash
fnm install --lts
fnm default lts-latest
```

## Verify

```bash
fnm --version
fnm current
node --version
```
