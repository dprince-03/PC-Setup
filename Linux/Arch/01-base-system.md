# 01 — Base System (Arch)

**Profiles:** core

## Step 1 — Mirrors, keyring, full upgrade

```bash
pac reflector
sudo cp -n /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
sudo reflector --latest 20 --protocol https --sort rate --save /etc/pacman.d/mirrorlist   # ask for the country (--country XX)
sudo pacman -Sy --needed --noconfirm archlinux-keyring && sudo pacman -Syu --noconfirm
```
pacman.conf QoL (idempotent; ask): enable `ParallelDownloads = 10`, `Color`, and `[multilib]`
(needed for Steam/Wine/some Android tools):
```bash
sudo sed -i 's/^#ParallelDownloads.*/ParallelDownloads = 10/; s/^#Color/Color/' /etc/pacman.conf
grep -q '^\[multilib\]' /etc/pacman.conf || sudo sed -i '/^#\[multilib\]/,/^#Include/ s/^#//' /etc/pacman.conf
sudo pacman -Syu --noconfirm
```

## Step 2 — Essentials

```bash
pac base-devel git git-lfs github-cli openssh curl wget jq ripgrep fd fzf bat eza zoxide tree \
  htop btop ncdu tmux neovim direnv shellcheck unzip zip p7zip man-db man-pages bash-completion \
  starship ttf-jetbrains-mono-nerd pacman-contrib
git lfs install
sudo systemctl enable --now paccache.timer     # keeps the pacman cache trimmed
```

## Step 3 — AUR helper (ask; skip if yay or paru already exists)

```bash
have yay || have paru || {
  tmp=$(mktemp -d) && git clone https://aur.archlinux.org/yay-bin.git "$tmp/yay-bin" &&
  (cd "$tmp/yay-bin" && cat PKGBUILD && makepkg -si --noconfirm); }   # show the PKGBUILD before building
yay --version
```

## Step 4 — Git config, SSH, shell

Same as `Linux/Ubuntu/01-base-system.md` Steps 4–5 (ask for name/email, ed25519 key, `gh auth login` by
the user, guarded rc lines for starship/zoxide/direnv).

## Step 5 — Flatpak (ask)

`pac flatpak && flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo`

## Step 6 — Security baseline

```bash
pac ufw && sudo systemctl enable --now ufw && sudo ufw default deny incoming && sudo ufw --force enable
# (or firewalld: pac firewalld && sudo systemctl enable --now firewalld). Pick one
pac apparmor 2>/dev/null   # optional; needs kernel params. Ask, and don't touch the bootloader automatically
```

## Step 7 — Drivers (desktop; ask)

```bash
# CPU microcode (bootloader picks it up via the microcode hook on recent mkinitcpio)
grep -q GenuineIntel /proc/cpuinfo && pac intel-ucode || pac amd-ucode
# GPU
pac mesa vulkan-icd-loader                         # all GPUs
# Intel: pac vulkan-intel intel-media-driver  | AMD: pac vulkan-radeon
# NVIDIA (Turing/RTX 20 and newer):
#   stock 'linux' kernel only →  pac nvidia-open nvidia-utils nvidia-settings
#   other/multiple kernels     →  pac nvidia-open-dkms nvidia-utils nvidia-settings <kernel>-headers
# Older NVIDIA GPUs: see the Arch Wiki "NVIDIA" page (legacy drivers come from the AUR)
```
After an NVIDIA install: check that `nvidia_drm.modeset=1` is in effect (current packages default to it),
rebuild the initramfs if the Wiki says so (`sudo mkinitcpio -P`), and **reboot**. With Secure Boot, you
need signed modules (sbctl + DKMS signing). Explain that and ask; don't change keys automatically.

## Step 8 — Developer sysctl

```bash
echo 'fs.inotify.max_user_watches=524288' | sudo tee /etc/sysctl.d/90-dev-inotify.conf && sudo sysctl --system
```

## Verify

`pacman -Qu | wc -l` (should be 0) · `yay --version` · `gh --version | head -1` · `nvidia-smi -L 2>/dev/null`
