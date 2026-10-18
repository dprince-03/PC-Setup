# Omarchy Playbook

Target: **[Omarchy](https://omarchy.org)** (<https://github.com/basecamp/omarchy>), DHH's
opinionated Arch Linux + Hyprland setup.

Omarchy is **Arch underneath**, so pacman/yay work, but it has its own management layer. The agent
must use it:

| Need | Use the Omarchy command (not raw tools) |
|---|---|
| Update everything | `omarchy update` (snapshot + pacman + AUR + mise + migrations), **never** a bare `pacman -Syu` |
| Add Arch packages | `omarchy pkg add <pkgs>` (wraps `pacman -S --needed`, then verifies) |
| Add AUR packages | `omarchy pkg aur add <pkgs>` (ask first, same as Arch) |
| Programming languages | `omarchy install dev-env <lang>` (installs via **mise**, `@latest`) |
| Databases (Docker) | `omarchy install docker-dbs` (or `omarchy-install-docker-dbs MySQL PostgreSQL Redis MongoDB`) |
| Editors | `omarchy-install-editor-vscode` / `-zed` / `-helix` / `-emacs` |
| Remove bundled apps ("debloat") | `omarchy-remove-preinstalls` (asks for confirmation) |
| Snapshots | `omarchy-snapshot create` / `restore` (snapper) |
| Re-apply defaults if something breaks | `omarchy-refresh-config <file>`, `omarchy reinstall` |

Discover commands: `ls $(dirname "$(command -v omarchy)") | grep omarchy-`, or `omarchy --help`.
Read a script before running it: `cat "$(command -v omarchy-<name>)"`.

**Already included in Omarchy (check, don't reinstall):** Docker + Compose + Buildx, mise, git,
lazygit, Neovim (LazyVim), Chromium, tmux, ripgrep, fd, fzf, eza, zoxide, bat, btop, jq, starship,
ufw (+ ufw-docker), yay, base-devel, Ruby, .NET runtime, Obsidian, LibreOffice, LocalSend, mpv,
Nautilus, JetBrains Mono Nerd font, plus web apps and TUIs. The base package list lives in
`$OMARCHY_PATH/install/omarchy-base.packages`.

| # | Module | Profiles |
|---|---|---|
| 00 | [Preflight](00-preflight.md) | all |
| 01 | [Base system](01-base-system.md) | core |
| 02 | [Languages](02-languages.md) | languages |
| 03 | [Databases](03-databases.md) | databases |
| 04 | [IDEs and editors](04-ides-editors.md) | ides |
| 05 | [Full-stack web](05-fullstack-web.md) | fullstack |
| 06 | [Mobile](06-mobile.md) | mobile |
| 07 | [Data science / ML](07-data-science-ml.md) | data-ml |
| 08 | [Desktop apps](08-desktop-apps.md) | desktop |
| 09 | [DevOps and cloud](09-devops-cloud.md) | devops-cloud |
| 10 | [Sysadmin and networking](10-sysadmin-networking.md) | sysadmin-net, linux-admin |
| 11 | [Productivity](11-productivity.md) | productivity |
| 12 | [Verify and report](12-verify-and-report.md) | all |

Omarchy's own dotfiles live in `~/.config` and get **refreshed by migrations**. Put user
customisations where Omarchy expects them (e.g. `~/.config/hypr/*.conf` overrides,
`~/.config/omarchy/`). Don't edit files under `$OMARCHY_PATH`, because updates overwrite them.
