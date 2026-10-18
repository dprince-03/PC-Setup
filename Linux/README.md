# Linux Playbooks

> Agent: read [`../AGENT.md`](../AGENT.md) first. Then detect the distro (below) and work **only**
> inside the matching folder.

## Distro detection

```bash
. /etc/os-release
echo "ID=$ID ID_LIKE=$ID_LIKE VERSION_ID=$VERSION_ID CODENAME=${VERSION_CODENAME:-} UBUNTU_CODENAME=${UBUNTU_CODENAME:-}"
uname -srm
grep -qi microsoft /proc/version && echo "Running inside WSL"
{ [ -d "$HOME/.local/share/omarchy" ] || command -v omarchy >/dev/null; } && echo "Omarchy detected"
```

| Result | Folder |
|---|---|
| `ID=ubuntu`, or `ID_LIKE` contains `ubuntu` (Mint, Pop!_OS, Zorin, elementary, KDE neon) | [`Ubuntu/`](Ubuntu/) |
| `ID=fedora` | [`Fedora/`](Fedora/) (for Atomic desktops, e.g. Silverblue/Kinoite, see its README) |
| `ID=arch` and Omarchy **not** detected | [`Arch/`](Arch/) |
| `ID=arch` **and** Omarchy detected | [`Omarchy/`](Omarchy/) |
| `ID=debian`, `ID_LIKE=arch` (Manjaro, EndeavourOS, CachyOS) or anything else | **STOP.** No playbook. Arch derivatives *can* use `Arch/` if the user explicitly agrees. Warn them first |

## Module numbering (the same in every distro folder)

| # | Module | Profiles |
|---|---|---|
| 00 | Preflight: OS check, inventory, snapshot, helpers | all |
| 01 | Base system: update, repos, essentials, cleanup, firewall, drivers | core |
| 02 | Languages: Python, Node, Java, Go, C/C++, .NET, Rust | languages |
| 03 | Databases: PostgreSQL, MySQL, MongoDB, Redis + GUIs | databases |
| 04 | IDEs and editors: VS Code, IntelliJ, PyCharm, Postman... | ides |
| 05 | Full-stack web | fullstack |
| 06 | Mobile: Android Studio, Flutter, React Native | mobile |
| 07 | Data science / ML / DL (+ NVIDIA/CUDA) | data-ml |
| 08 | Desktop app dev: GTK, Qt, Electron, Tauri, .NET | desktop |
| 09 | DevOps and cloud: Docker, K8s, IaC, cloud CLIs | devops-cloud |
| 10 | Sysadmin, networking, Linux admin | sysadmin-net, linux-admin |
| 11 | Productivity | productivity |
| 12 | Security and antivirus: ClamAV, rootkit checks, Lynis, firewall, MAC, encryption | security |
| 13 | Verify and report | all |

## Shared conventions across distros

- **Runtime version manager: [mise](https://mise.jdx.dev)** for Node, Go, Java, Maven, Gradle, and
  others. It gives the same commands on every distro, `@latest` resolves the newest stable,
  and Omarchy ships it by default.
- **Python: [uv](https://docs.astral.sh/uv/)** for interpreters, venvs and tools. The system
  Python belongs to the OS. **Never `sudo pip install`.**
- **GUI apps:** the distro's native package first, then the vendor's own repo, then **Flathub**
  (verified publisher). Snap only on Ubuntu, and only from the official publisher.
- **Dev databases:** Docker containers by default (AGENT.md Rule 4b), native packages if the user
  asks.
- **WSL:** skip GPU drivers, display/desktop tweaks, and GUI-only apps.
