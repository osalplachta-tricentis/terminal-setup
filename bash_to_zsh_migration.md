# Bash to Zsh Migration Guide (WSL)

Idempotent migration instructions — each step checks before acting.

## 1. Install Zsh

```bash
if ! command -v zsh &>/dev/null; then
  sudo apt update && sudo apt install -y zsh
else
  echo "zsh already installed: $(zsh --version)"
fi
```

## 2. Set Zsh as Default Shell

```bash
if [ "$(basename "$SHELL")" != "zsh" ]; then
  chsh -s "$(which zsh)"
  echo "Default shell changed to zsh. Log out and back in to take effect."
else
  echo "zsh is already the default shell"
fi
```

## 3. Install Oh My Zsh

```bash
if [ ! -d "$HOME/.oh-my-zsh" ]; then
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
else
  echo "Oh My Zsh already installed"
fi
```

## 4. Set Theme (Optional — Agnoster)

```bash
if ! grep -q '^ZSH_THEME="agnoster"' ~/.zshrc; then
  sed -i 's/^ZSH_THEME=.*/ZSH_THEME="agnoster"/' ~/.zshrc
  echo "Theme set to agnoster"
else
  echo "Theme already set to agnoster"
fi
```

> Agnoster requires a Powerline-patched font in your terminal emulator.

## 5. Migrate Key Config from .bashrc

Transfer NVM, custom PATH entries, and other environment setup from `~/.bashrc` to `~/.zshrc`.

### NVM

```bash
if ! grep -q 'NVM_DIR' ~/.zshrc; then
  cat >> ~/.zshrc << 'EOF'

# NVM
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
EOF
  echo "NVM config added to .zshrc"
else
  echo "NVM config already present in .zshrc"
fi
```

### Custom PATH entries

```bash
# Review what .bashrc adds to PATH and replicate as needed:
grep 'PATH=' ~/.bashrc
# Then add any missing entries to ~/.zshrc
```

### Aliases and functions

```bash
# Check for custom aliases/functions in .bashrc and copy relevant ones:
grep -n 'alias\|function ' ~/.bashrc
```

## 6. Install Plugins

### zsh-autosuggestions (fish-like history suggestions)

```bash
ZSH_CUSTOM="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}"
if [ ! -d "$ZSH_CUSTOM/plugins/zsh-autosuggestions" ]; then
  git clone https://github.com/zsh-users/zsh-autosuggestions "$ZSH_CUSTOM/plugins/zsh-autosuggestions"
else
  echo "zsh-autosuggestions already installed"
fi
```

### zsh-syntax-highlighting (command validation coloring)

```bash
ZSH_CUSTOM="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}"
if [ ! -d "$ZSH_CUSTOM/plugins/zsh-syntax-highlighting" ]; then
  git clone https://github.com/zsh-users/zsh-syntax-highlighting "$ZSH_CUSTOM/plugins/zsh-syntax-highlighting"
else
  echo "zsh-syntax-highlighting already installed"
fi
```

### zsh-autocomplete (real-time tab-completion menu)

```bash
ZSH_CUSTOM="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}"
if [ ! -d "$ZSH_CUSTOM/plugins/zsh-autocomplete" ]; then
  git clone --depth 1 https://github.com/marlonrichert/zsh-autocomplete.git "$ZSH_CUSTOM/plugins/zsh-autocomplete"
else
  echo "zsh-autocomplete already installed"
fi
```

### Enable plugins in .zshrc

```bash
if ! grep -q 'zsh-autosuggestions' ~/.zshrc; then
  sed -i 's/^plugins=(\(.*\))/plugins=(\1 zsh-autosuggestions zsh-syntax-highlighting zsh-autocomplete)/' ~/.zshrc
  echo "Plugins added to .zshrc"
else
  echo "Plugins already configured"
fi
```

## 7. Apply Changes

```bash
source ~/.zshrc
```

## Verification

```bash
echo "Shell: $SHELL"
echo "Zsh version: $(zsh --version)"
echo "Oh My Zsh: $([ -d ~/.oh-my-zsh ] && echo 'installed' || echo 'missing')"
echo "Autosuggestions: $([ -d ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions ] && echo 'installed' || echo 'missing')"
echo "Syntax highlighting: $([ -d ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting ] && echo 'installed' || echo 'missing')"
echo "Autocomplete: $([ -d ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autocomplete ] && echo 'installed' || echo 'missing')"
echo "NVM: $(command -v nvm &>/dev/null && echo 'loaded' || echo 'not loaded')"
```
