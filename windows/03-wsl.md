# 03 — WSL2 and Linux on Windows

**Profiles:** linux-admin, devops-cloud, data-ml (and recommended for fullstack) · **Admin:** yes ·
**Reboot:** usually yes, the first time

WSL2 runs a real Linux kernel. Docker Desktop, GPU ML (TensorFlow), Ansible, and most
Linux/sysadmin practice need it.

---

## Step 1 — Check

```powershell
wsl --version            # errors or prints nothing → not installed
wsl --status
wsl --list --verbose     # installed distros and their WSL version
(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled   # must be True
```
If virtualization is `False`: tell the user to enable **Intel VT-x / AMD SVM** in BIOS/UEFI. Don't
try to work around it.

## Step 2 — Install / update WSL (ask; may require a reboot)

```powershell
wsl --install --no-distribution      # enables the features and installs the latest WSL from GitHub/Store
wsl --update                         # newest WSL release
wsl --set-default-version 2
```
If a reboot is needed: log the resume point ("resume at 03 Step 3"), finish modules that don't
need WSL (04–08, 10, 12, 13), then ask the user to reboot.

## Step 3 — Install distros (ask which ones)

```powershell
wsl --list --online                   # always pick from this list: names change between releases
wsl --install -d Ubuntu               # 'Ubuntu' always tracks the latest Ubuntu LTS
# Optional extras for linux-admin practice (ask):
# wsl --install -d FedoraLinux-<N>    # pick the highest N from --list --online
# wsl --install -d archlinux
```
The user creates the UNIX username and password interactively. Don't script it.

## Step 4 — `.wslconfig` (resource limits; merge, don't overwrite)

`%USERPROFILE%\.wslconfig`:
```ini
[wsl2]
memory=50%            ; WSL takes half of RAM by default. Lower it on 16 GB machines (e.g. 8GB)
processors=           ; leave empty for all cores
swap=8GB
networkingMode=mirrored   ; Win 11 22H2+: localhost and VPN behave the same in Windows and Linux
dnsTunneling=true
autoProxy=true

[experimental]
autoMemoryReclaim=gradual
sparseVhd=true
```
Then `wsl --shutdown`.

## Step 5 — systemd and basics inside the distro

```powershell
wsl -d Ubuntu -u root -- bash -c "grep -q '^systemd=true' /etc/wsl.conf || printf '[boot]\nsystemd=true\n' >> /etc/wsl.conf"
wsl --shutdown
wsl -d Ubuntu -- bash -lc "sudo apt update && sudo apt full-upgrade -y"
```

## Step 6 — Provision the Linux side

Inside WSL, run the matching Linux playbook (e.g. `Linux/Ubuntu/`), but:
- **skip** GPU *driver* installation. WSL uses the Windows NVIDIA driver; install only the
  CUDA *toolkit for WSL* if data-ml is selected (see Ubuntu module 07).
- **skip** desktop apps, Flatpak GUI apps, and display/DE tweaks.
- **skip** Docker Engine if Docker Desktop with WSL integration is planned (module 11). Pick one.

Tip: keep Linux project files inside the Linux filesystem (`~/code`), not under `/mnt/c`. It's
10–20× faster.

## Verify

```powershell
wsl --version; wsl -l -v      # every distro should show VERSION 2
wsl -d Ubuntu -- bash -lc "cat /etc/os-release | head -3; systemctl is-system-running"
```
