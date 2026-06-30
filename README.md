# dotfiles

Managed by [chezmoi](https://www.chezmoi.io/). Package management via Brewfile generated from `packages.yaml`. 

## Overview 
- chezmoi managing all dotfiles
- Brewfile generated from .chezmoidata/packages.yaml for system packages
- KeePassXC for secrets
- mise for dev runtime versions (python, node, etc.) and system packages if they are not distributed with brew

## Setup

### Prerequisites

- **macOS** — Apple Silicon (arm64). The setup script hardcodes the Homebrew path at `/opt/homebrew/bin/brew`, so Intel (x86_64) is not supported.
- **Admin access** — the script triggers `sudo` prompts during Xcode Command Line Tools and Homebrew installation
- **KeePassXC** is recommended before running setup; the script will prompt for your database path (install later with `brew install --cask keepassxc` if missing)

### Install

```bash
curl -sfL https://raw.githubusercontent.com/mex7xx/dotfiles/main/.startup.sh | bash
```

The `.startup.sh` script runs end-to-end on a fresh machine. It performs each step in order:

1. **Xcode Command Line Tools** — `xcode-select --install` (skipped if already present)
2. **Homebrew** — installed via the official installer if `brew` is not on `PATH`; on Apple Silicon the script appends `brew shellenv` to `~/.zprofile` so `brew` survives a new shell
3. **chezmoi** — `brew install chezmoi`
4. **`chezmoi init mex7xx`** — clones this repo into `~/.local/share/chezmoi` and renders `.chezmoi.toml.tmpl`, which prompts for:
   - **Is this your work computer?** — `yes`/`no` (drives work/personal config)
   - **Path to KeePassXC database**
5. **`chezmoi apply`** — materializes dotfiles into your home directory and runs the `run_onchange_` scripts, which execute `brew bundle` against the generated `Brewfile`. A KeePassXC database test script (`_disabled_test-keepassxc.sh.tmpl`) also exists but is **disabled by default** — chezmoi skips any script whose name begins with `_`. To enable it, rename the file to drop the leading underscore.

### Post-setup verification

```bash
chezmoi doctor          # sanity-check chezmoi setup (source/target dirs, shell, etc.)
chezmoi diff            # should be empty after a clean apply
./audit-packages.sh     # confirm brew bundle ran and report any drift
```

If any of the above report unexpected results, see [Troubleshooting](#troubleshooting) or run `chezmoi apply -v` for verbose output.

## Troubleshooting

**Xcode CLT install prompt never appears.** Some shells swallow the GUI prompt. Trigger it manually:
```bash
xcode-select --install
```
Then re-run `.startup.sh`.

**`brew: command not found` after install.** The shellenv wasn't evaluated in the current shell. Run it (and ensure it's in `~/.zprofile`):
```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
```

**`chezmoi apply` fails on first run.** Usually `.chezmoi.toml.tmpl` didn't render into `~/.config/chezmoi/chezmoi.toml`. Check the config exists and re-init:
```bash
ls ~/.config/chezmoi/chezmoi.toml
chezmoi init --apply mex7xx
```

**KeePassXC not found / database test fails.** Install the cask and point chezmoi at the database:
```bash
brew install --cask keepassxc
chezmoi edit-config   # set the keepassxc database path
chezmoi apply
```

**`brew bundle` didn't run / packages missing.** `run_onchange_` scripts only fire when the underlying `packages.yaml` hash changes. Force a re-run by editing `packages.yaml`, or run `./audit-packages.sh` to see what's drifted.

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
          └── renders _disabled_test-keepassxc.sh.tmpl → skipped by default (rename to enable)

Ongoing use:
  chezmoi apply
    ├── run_onchange_ → only if packages.yaml or brew state changed
    ├── installs declared packages and warns about drift
    └── _disabled_test-keepassxc.sh.tmpl → skipped by default (rename to enable)

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

