# Git Installation

Official project: <https://git-scm.com/>

## Install

```bash
sudo apt update
sudo apt install -y git
```

## Minimal baseline

Set your identity and a few safe global defaults.

Only the placeholder identity values below need manual replacement. The guards keep existing values intact.

```bash
git config --global --get user.name >/dev/null || git config --global user.name "Your Name"
git config --global --get user.email >/dev/null || git config --global user.email "you@example.com"
git config --global --get init.defaultBranch >/dev/null || git config --global init.defaultBranch main
git config --global --get pull.rebase >/dev/null || git config --global pull.rebase false
git config --global --get core.editor >/dev/null || git config --global core.editor nano
```

If you prefer another editor, replace `nano`.

## Verify

```bash
git --version
git config --global --list
```
