# PC-Setup

Agent-readable playbooks that take a fresh **Windows** or **Linux** machine to a fully
**developer-ready** workstation for:

- Full-stack software engineering
- Mobile development (Android, Flutter, React Native)
- Data science, ML and deep learning (GPU included)
- Desktop app development (.NET, Electron, Tauri, Qt)
- DevOps and cloud (Docker, Kubernetes, IaC, AWS/Azure/GCP)
- Sysadmin, networking and Linux administration
- Plus everyday productivity tools

Each playbook is a set of numbered Markdown files written **for an AI agent to read and run**.
A human can follow them too.

## How to use it

1. Clone the repo onto the target machine:
   ```bash
   git clone https://github.com/dprince-03/PC-Setup.git
   cd PC-Setup
   ```
2. Open your coding agent (Claude Code, Codex CLI, Gemini CLI, Copilot CLI, ...) in that folder.
3. Give it this prompt:

   > Read `AGENT.md` and follow its rules exactly. Detect my OS, run the matching playbook's
   > preflight, show me what's already installed and your plan, ask which profiles I want,
   > then set up my machine.

The agent will:
- **detect the OS first** and refuse to run the wrong playbook
- **inventory what's already installed** and skip or upgrade it, never blindly reinstall
- install **only the latest stable versions**, from **official sources**
- ask before debloating, rebooting, adding repos, or installing drivers
- verify every tool and write a final report to `~/pc-setup-logs/`

## Layout

| Path | Target |
|---|---|
| [`AGENT.md`](AGENT.md) | Rules every agent must follow (read first) |
| [`windows/`](windows/) | Windows 11: debloat, winget, WSL2, the whole stack |
| [`Linux/Ubuntu/`](Linux/Ubuntu/) | Ubuntu LTS/interim + derivatives (Mint, Pop!_OS, Zorin) |
| [`Linux/Fedora/`](Linux/Fedora/) | Fedora Workstation and spins |
| [`Linux/Arch/`](Linux/Arch/) | Vanilla Arch Linux |
| [`Linux/Omarchy/`](Linux/Omarchy/) | [Omarchy](https://omarchy.org): DHH's Arch + Hyprland setup |

Every playbook uses the same module numbering, so you can find the same topic in any OS folder:

| # | Windows | Linux |
|---|---|---|
| 00 | Preflight (OS check, inventory, restore point) | Preflight |
| 01 | Debloat and privacy | Base system (updates, repos, cleanup) |
| 02 | Foundation (winget, terminal, Git, shell) | Languages |
| 03 | WSL2 | Databases |
| 04 | Languages | IDEs and editors |
| 05 | Databases | Full-stack web |
| 06 | IDEs and editors | Mobile |
| 07 | Full-stack web | Data science / ML / DL |
| 08 | Mobile | Desktop app dev |
| 09 | Data science / ML / DL | DevOps and cloud |
| 10 | Desktop app dev | Sysadmin, networking, Linux admin |
| 11 | DevOps and cloud | Productivity |
| 12 | Sysadmin and networking | Verify and report |
| 13 | Productivity | |
| 14 | Verify and report | |

## Stack covered

**Languages:** Python, JavaScript/TypeScript (Node.js, Bun, Deno), Java (+ Maven/Gradle),
Go, C/C++ (GCC/Clang/MSVC, CMake, Ninja), C#/.NET, plus optional Rust and Kotlin.
**Databases:** PostgreSQL, MySQL, MongoDB, Redis (native or Docker) + DBeaver, pgAdmin,
Compass, Workbench, Redis Insight.
**DevOps:** Docker, Kubernetes (kubectl, kind, minikube, k9s), Helm, Terraform/OpenTofu, Ansible,
AWS/Azure/GCP CLIs.
**IDEs/tools:** VS Code, Visual Studio 2026, IntelliJ IDEA, PyCharm, Android Studio, Postman,
Bruno, Neovim, JetBrains Toolbox.
**Productivity:** PowerToys, Everything, Flow Launcher, ShareX, Obsidian, Bitwarden, and the
Linux equivalents.

## Updating everything later

| OS | Command |
|---|---|
| Windows | `winget upgrade --all --include-unknown` |
| Ubuntu | `sudo apt update && sudo apt full-upgrade && flatpak update && snap refresh` |
| Fedora | `sudo dnf upgrade --refresh && flatpak update` |
| Arch | `sudo pacman -Syu && yay -Sua` |
| Omarchy | `omarchy update` |
| Version managers | `mise upgrade`, `uv self update`, `rustup update`, `fnm install --lts` |

## Disclaimer

These playbooks change system settings. The agent creates a restore point or snapshot first,
but you are responsible for your machine. Read the debloat module before approving it.
