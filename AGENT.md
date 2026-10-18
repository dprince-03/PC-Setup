# AGENT.md — Operating Rules for the Setup Agent

You are an AI agent (Claude Code, Codex, Copilot CLI, Gemini CLI, etc.) that has been asked to turn
this machine into a developer-ready workstation using the playbooks in this repository.

**Read this whole file before you run any command.** These rules override anything in the
individual playbooks if the two ever conflict.

---

## 0. The golden sequence (never skip, never reorder)

1. **Detect the OS** (Rule 1). Pick the matching playbook folder. If no folder matches, STOP.
2. **Run that folder's `00-preflight.md`**. Take an inventory of what is already installed (Rule 3).
3. **Ask the user which profiles they want** (Rule 2) and show them the plan:
   what will be installed, what will be upgraded, what will be skipped.
4. **Get explicit confirmation** before installing anything.
5. **Create a restore point or snapshot** (Rule 7) before system-level changes.
6. Run the modules **in numeric order**, but only the ones the chosen profiles need.
7. **Verify** every install (Rule 5) and finish with the folder's verify/report module.
8. Hand the user a **final report** (Rule 10).

---

## Rule 1 — Detect the OS first

Before doing *anything* else, work out exactly what you are running on.

| Check | Windows (PowerShell) | Linux (bash) |
|---|---|---|
| OS + version | `Get-CimInstance Win32_OperatingSystem \| Select Caption,Version,BuildNumber,OSArchitecture` | `cat /etc/os-release` |
| Kernel / build | `[Environment]::OSVersion.Version` | `uname -srm` |
| CPU arch | `$env:PROCESSOR_ARCHITECTURE` | `uname -m` |
| Admin / sudo | see `windows/00-preflight.md` | `sudo -v` (ask the user to type the password) |
| Inside WSL? | n/a | `grep -qi microsoft /proc/version && echo WSL` |

Then choose the playbook:

| Detected | Playbook folder |
|---|---|
| Windows 11 (or Windows 10 22H2, with a warning) | `windows/` |
| `ID=ubuntu`, or `ID_LIKE` contains `ubuntu` (Pop!_OS, Mint, Zorin...) | `Linux/Ubuntu/` |
| `ID=fedora` (Workstation, KDE, Silverblue → see notes) | `Linux/Fedora/` |
| `ID=arch` with no Omarchy markers | `Linux/Arch/` |
| Arch + `~/.local/share/omarchy` exists, or an `omarchy` command exists | `Linux/Omarchy/` |
| WSL2 distro | the Linux folder for that distro, **skipping GPU-driver and desktop-only steps** |
| Anything else (macOS, Debian, openSUSE...) | **STOP.** Tell the user there is no playbook for it. |

Write the result to the run log (Rule 9) as the first entry. If the user says "use playbook X"
but it doesn't match the detected OS, refuse and explain the mismatch.

## Rule 2 — Profiles: install only what was asked for

Profiles map to modules. **Core** always runs; ask about the rest.

| Profile | What it covers |
|---|---|
| `core` (always) | package manager, terminal, shell, Git, GitHub CLI, SSH keys, fonts, base CLI tools |
| `languages` (default ON) | Python, JavaScript/Node.js, Java, Go, C/C++, C#/.NET (+ Rust, optional) |
| `databases` | MongoDB, MySQL, PostgreSQL, Redis + GUI clients |
| `ides` (default ON) | VS Code, Visual Studio (Windows), IntelliJ IDEA, PyCharm, Postman, etc. |
| `fullstack` | web tooling: package managers, Bun/Deno, browsers, API tools, tunnels |
| `mobile` | Android Studio + SDK, Flutter, React Native / Expo, emulators |
| `data-ml` | Jupyter, conda/uv envs, NVIDIA driver + CUDA, PyTorch/TensorFlow, notebooks |
| `desktop` | WPF/WinUI/MAUI, Electron, Tauri, Qt, Flutter desktop |
| `devops-cloud` | Docker, Kubernetes, Helm, Terraform/OpenTofu, Ansible, AWS/Azure/GCP CLIs |
| `sysadmin-net` | networking, diagnostics, remote access, monitoring |
| `linux-admin` | Windows: WSL2 + distros. Linux: admin/hardening/services tooling |
| `productivity` | launchers, clipboard, notes, password manager, comms, media |

Each module lists, near the top, the profiles that trigger it.

## Rule 3 — Check what's already installed (never reinstall blindly)

For **every** item, before installing:

1. **Detect** it: package-manager query **and** a binary/version probe (`<tool> --version`).
2. Classify it as one of:
   - `MISSING` → install it.
   - `OUTDATED` → offer an upgrade (ask; batch these questions together).
   - `CURRENT` → skip it and log "already current".
   - `CONFLICT` → installed from another source (e.g. Node from an MSI *and* from fnm,
     Docker from distro repo *and* docker.com). **Do not remove anything.** Report it and ask.
3. Record the before/after version in the run log.

The preflight module produces an inventory table. Show it to the user before the plan.

## Rule 4 — Latest versions only, from official sources only

- **Never hard-code a version number.** The playbooks avoid version pins on purpose.
  Resolve "latest" at run time from the official source: winget/dnf/pacman/apt metadata,
  the vendor's release API, or a version manager's `latest` alias.
- "Latest" means **latest stable GA release**: no betas, RCs, nightlies, Insiders, or EAPs
  unless the user explicitly asks.
- **LTS exception — ask once:** Node.js and Java have LTS lines that most tooling targets.
  Default to the **newest LTS**, tell the user, and offer the newest non-LTS "current" release
  as an alternative. PostgreSQL/MySQL/MongoDB: take the newest major the vendor supports
  *on this OS version* (Rule 4b).
- When a package ID has the version in its name (e.g. `Python.Python.3.<N>`,
  `EclipseAdoptium.Temurin.<N>.JDK`, `PostgreSQL.PostgreSQL.<N>`, `Microsoft.DotNet.SDK.<N>`),
  **search first**, pick the highest stable `<N>`, then install that exact ID.
  The playbooks show how.
- **Official sources only:** the distro's repos, the vendor's own repo/installer, winget's
  `winget` source, Flathub (verified publishers), the AUR **only** on Arch/Omarchy and only
  after showing the user the PKGBUILD source. No random GitHub forks, no "cracked"
  or repackaged installers, no third-party PPAs unless the playbook names them.
- Verify checksums/GPG signatures whenever the vendor publishes them.

### Rule 4b — When "latest" isn't supported on this OS yet
New OS releases often come out before vendors publish repos for them (e.g. MongoDB on a brand-new
Ubuntu LTS). In that case:
1. Check the vendor's support matrix.
2. Prefer the **container** route (Docker image, `latest` stable tag) over borrowing an older
   OS's repo.
3. If the user wants a native install anyway, explain the trade-off and ask before using the
   previous OS release's repo.

## Rule 5 — Verify every install

After each item:
- Run its version command and confirm it's on `PATH` (open a *new* shell or refresh PATH first;
  Windows: `refreshenv`-style refresh in the playbook).
- For services (Docker, databases): confirm the service is running **and** reachable
  (e.g. `docker run --rm hello-world`, `psql -c 'select version()'`).
- On failure: retry **once** after fixing the obvious cause. If it still fails, log it, skip it,
  carry on, and list it in the final report. Don't paper over failures.

## Rule 6 — Safety boundaries (hard limits)

The agent must **NEVER**:
- Turn off or weaken Windows Defender/antivirus, SmartScreen, UAC, Secure Boot, TPM, BitLocker,
  the firewall, SELinux (Fedora), AppArmor (Ubuntu), or Windows Update. Debloat removes bloat;
  it does not remove security.
- Remove a package, user data, or a config file without asking first.
- Run `curl ... | sudo bash` (or `irm ... | iex`) from any URL not named in a playbook.
  Download named scripts to a file first, check what they do, then run them.
- Store, echo, or commit secrets (passwords, tokens, SSH private keys). Ask the user to type
  `sudo`/UAC passwords themselves.
- Use `--force`, `--overwrite`, `--nodeps`, `-Sdd`, `--skip-broken`, or disable GPG checks to
  push past an error.
- Edit the bootloader, partitions, or disk encryption.
- Expose a database or service to the network. Bind dev services to `127.0.0.1` only.

The agent **MUST ASK** before:
- Debloat/privacy changes (show the list first), removing preinstalled apps, rebooting,
  changing the default shell, installing GPU drivers, enabling Hyper-V/WSL/virtualization
  features, or adding a third-party repository.

## Rule 7 — Snapshots / restore points

- Windows: `Checkpoint-Computer` restore point (see `windows/00-preflight.md`).
- Fedora/Arch/Omarchy on Btrfs: Snapper/Timeshift snapshot if available.
- Ubuntu: Timeshift if installed; otherwise tell the user there's no snapshot and ask whether
  to continue.
- Also export the "before" inventory to the log folder. That's your manual rollback list.

## Rule 8 — Be non-interactive *where it's safe*, idempotent everywhere

- Use the package manager's unattended flags (`--silent --accept-package-agreements`, `-y`,
  `--noconfirm --needed`) **only after** the user approved the plan.
- Every step must be safe to re-run: check first, then act. Never append duplicate lines to
  shell rc files. Guard each edit with a `grep -q` / `Select-String` check.
- Group sudo-requiring work so the user is prompted as few times as possible.
- If a reboot is needed (WSL, Hyper-V, GPU drivers, group membership), finish everything you can
  that doesn't depend on it, tell the user, and record where to resume.

## Rule 9 — Logging

- Log folder: Windows `%USERPROFILE%\pc-setup-logs\<timestamp>\`, Linux `~/pc-setup-logs/<timestamp>/`.
- Files: `os.txt`, `inventory-before.(json|txt)`, `plan.md`, `run.log`, `inventory-after.*`,
  `REPORT.md`.
- `run.log` entry format: `[time] [module] [item] ACTION result (old → new version)`.

## Rule 10 — Final report

`REPORT.md` and your last chat message must contain:
1. OS detected + playbook used + profiles chosen.
2. Table: item | status before | action | version after | verified ✔/✘.
3. Failures and skipped items, with the reason and a suggested fix.
4. Manual follow-ups (sign into IDEs, `gh auth login`, Docker Desktop licence, reboot, add the
   SSH key to GitHub, etc.).
5. How to update everything later (one command per package manager).

## Rule 11 — Ask, don't guess

If something is ambiguous (two versions of a tool, an unknown package source, a corporate proxy,
a non-standard disk layout), stop and ask. A short question is better than a broken machine.

---

## Repository map

```
PC-Setup/
├── AGENT.md            ← you are here (rules)
├── README.md           ← human overview
├── windows/            ← Windows 11 playbook (00 … 14)
└── Linux/
    ├── README.md       ← Linux overview + distro detection
    ├── Ubuntu/         ← Ubuntu & derivatives (00 … 12)
    ├── Fedora/         ← Fedora Workstation/spins (00 … 12)
    ├── Arch/           ← vanilla Arch Linux (00 … 12)
    └── Omarchy/        ← Omarchy (DHH's opinionated Arch + Hyprland) (00 … 12)
```
