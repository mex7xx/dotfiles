# dotfiles

Managed by [chezmoi](https://www.chezmoi.io/). Package management via Brewfile generated from `packages.yaml`. 

## Overview 
- chezmoi managing all dotfiles
- Brewfile generated from .chezmoidata/packages.yaml for system packages
- KeePassXC for secrets
- mise for dev runtime versions (python, node, etc.) and system packages if they are not distributed with brew

## Setup

```bash
curl -sfL https://raw.githubusercontent.com/mex7xx/dotfiles/main/.startup.sh | bash
```

Prompts: work/personal profile + KeePassXC database path.

## Repo-local scripts (NOT managed by chezmoi)

| Script | When to run |
|---|---|
| `_package-helpers.sh` | Never directly — sourced by the 3 scripts below |
| `audit-packages.sh` | Manually, when you want to check drift |
| `cleanup-packages.sh` | Manually, when you want to remove drift |
| `reconcile-packages.sh` | Manually, when adding new packages |
| `.startup.sh` | Once, on a fresh machine |

## How phases relate

```
Fresh machine:
  .startup.sh
    ├── installs xcode, brew, chezmoi
    ├── runs chezmoi init mex7xx
    │     └── renders .chezmoi.toml.tmpl → prompts for config
    └── runs chezmoi apply
          ├── renders run_onchange_before_install-packages.sh.tmpl → brew bundle
          └── renders run_test-keepassxc.sh.tmpl → tests keepassxc

Ongoing use:
  chezmoi apply
    ├── run_onchange_ → only if packages.yaml or brew state changed
    ├── installs declared packages and warns about drift
    └── run_ → keepassxc test every time

Drift management (manual):
  ./reconcile-packages.sh   → add new packages to yaml
  ./audit-packages.sh       → check what's drifted
  ./cleanup-packages.sh     → remove undeclared packages
```

## Package management

- Edit `.chezmoidata/packages.yaml` then `chezmoi apply`
- `./audit-packages.sh` — report drift
- `./reconcile-packages.sh` — interactively add/remove undeclared packages
- `./cleanup-packages.sh` — remove all undeclared packages

## Personalize chezmoi repo and dotfiles 
```bash
chezmoi add ~/.zshrc          # add your dotfiles
chezmoi add ~/.gitconfig      # managed by chezmoi from now on
chezmoi cd                    # edit source files directly
```

## Managing tool versions with mise

Use `packages.yaml` for system-level tools and apps. Use `mise` for developer and project-specific runtimes like Python, Node.js, Ruby, Go, and other language/tool versions.

Set global defaults with `mise`:

```bash
mise use --global python@latest
mise use --global node@lts
```

Set project-specific versions from inside a project:

```bash
mise use python@3.12
mise use node@22
```

This lets each project pin its own runtime versions independently of the machine-wide defaults.

### Avoid untracked installs like npm -g 
For global CLI tools published to npm, use `mise use --global npm:<package>` instead of `npm i -g`:

```bash
mise use --global npm:command-code@latest
```

