# dotfiles

Managed by [chezmoi](https://www.chezmoi.io/). Package management via Brewfile generated from `packages.yaml`.

## Setup

```bash
curl -sfL https://raw.githubusercontent.com/mex7xx/dotfiles/main/.startup.sh | bash
```

Prompts: work/personal profile + KeePassXC database path.

## Package management

- Edit `.chezmoidata/packages.yaml` then `chezmoi apply`
- `./audit-packages.sh` — report drift
- `./reconcile-packages.sh` — interactively add/remove undeclared packages
- `./cleanup-packages.sh` — remove all undeclared packages

## Personalize

```bash
chezmoi add ~/.zshrc          # add your dotfiles
chezmoi add ~/.gitconfig      # managed by chezmoi from now on
chezmoi cd                    # edit source files directly
```
