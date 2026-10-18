# 01 — Base System (Fedora)

**Profiles:** core

## Step 1 — dnf tuning and full update

```bash
# Faster downloads (idempotent)
grep -q '^max_parallel_downloads' /etc/dnf/dnf.conf || echo 'max_parallel_downloads=10' | sudo tee -a /etc/dnf/dnf.conf
sudo dnf upgrade --refresh -y
sudo fwupdmgr refresh --force 2>/dev/null; sudo fwupdmgr get-updates 2>/dev/null   # firmware (ask before fwupdmgr update)
```
If the kernel was updated, note "reboot pending".

## Step 2 — RPM Fusion and Flathub (ask; needed for NVIDIA drivers and codecs)

```bash
dnf_install "https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm" \
            "https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm"
sudo dnf group upgrade -y core 2>/dev/null
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak remote-modify --enable flathub      # Fedora may ship a filtered/disabled Flathub
# Multimedia codecs (optional, ask):
sudo dnf swap -y ffmpeg-free ffmpeg --allowerasing 2>/dev/null
```

## Step 3 — Essentials

```bash
dnf_install @development-tools git git-lfs gh curl wget jq ripgrep fd-find fzf bat eza zoxide tree \
  htop btop ncdu tmux neovim direnv ShellCheck unzip p7zip p7zip-plugins starship
git lfs install
```
(If `starship` or `eza` isn't in the repos for this release, install Starship with the official
script as in Ubuntu 01 Step 5.)

## Step 4 — Git config, SSH key, shell

Same as Ubuntu 01 Steps 4–5: ask for name/email, set `init.defaultBranch main`, create the
ed25519 key if missing, have the user run `gh auth login`, and add starship/zoxide/direnv init
lines to `~/.bashrc` with `grep -qxF` guards.

Nerd Font: `dnf_install jetbrains-mono-fonts-all` (plain), or download the Nerd Font tarball as in
Ubuntu 01.

## Step 5 — "Debloat" (ask; optional)

Fedora Workstation is lean. Offer these:
```bash
sudo dnf remove -y gnome-tour gnome-maps gnome-weather gnome-contacts 2>/dev/null   # only items the user ticks
sudo systemctl disable --now abrtd 2>/dev/null       # crash reporting (optional)
```
Never remove `firewalld`, `selinux-policy`, or `PackageKit` without asking (GNOME Software
depends on PackageKit).

## Step 6 — Security baseline (keep it ON)

```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --get-default-zone      # FedoraWorkstation: it opens high ports; consider 'public'
getenforce                                # keep Enforcing
dnf_install dnf-automatic && sudo systemctl enable --now dnf-automatic.timer   # optional: auto security updates
```
SELinux denials during dev → read `sudo ausearch -m avc -ts recent` and fix labels
(`restorecon`, `:Z` on container volumes). **Never** `setenforce 0` as a fix.

## Step 7 — NVIDIA driver (desktop + NVIDIA only; ask)

```bash
dnf_install akmod-nvidia xorg-x11-drv-nvidia-cuda      # from RPM Fusion nonfree
# Wait for the kmod to build (≈5 min), then check:
modinfo -F version nvidia
```
**Secure Boot on:** the akmod must be signed. Run `sudo kmodgenca -a && sudo mokutil --import /etc/pki/akmods/certs/public_key.der`
(the user picks a one-time password, then confirms in the blue MOK screen at reboot). Reboot afterwards.

## Step 8 — Developer sysctl

```bash
echo 'fs.inotify.max_user_watches=524288' | sudo tee /etc/sysctl.d/90-dev-inotify.conf && sudo sysctl --system
```

## Verify

```bash
dnf repolist | grep -Ei 'rpmfusion|fedora'; flatpak remotes; gh --version | head -1; getenforce; nvidia-smi -L 2>/dev/null
```
