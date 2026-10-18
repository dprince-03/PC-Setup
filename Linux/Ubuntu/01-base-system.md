# 01 — Base System (Ubuntu)

**Profiles:** core · Load the helpers from 00 first.

---

## Step 1 — Update everything

```bash
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt full-upgrade -y
sudo apt autoremove -y
snap refresh 2>/dev/null || true
```
Use `full-upgrade`, not `upgrade`. It pulls in new dependencies, including the kernel-matched
NVIDIA modules. A plain `upgrade` can leave the GPU driver behind a new kernel.
If `/var/run/reboot-required` exists, note it for the final report.

## Step 2 — Repositories and essentials

```bash
sudo add-apt-repository -y universe 2>/dev/null; sudo add-apt-repository -y multiverse 2>/dev/null
apt_install build-essential pkg-config curl wget ca-certificates gnupg lsb-release \
  software-properties-common apt-transport-https unzip zip p7zip-full tar xz-utils \
  git git-lfs openssh-client jq ripgrep fd-find fzf bat tree htop btop ncdu tmux \
  neovim zoxide eza direnv shellcheck man-db bash-completion
# Ubuntu ships some tools under other names:
mkdir -p ~/.local/bin
have fdfind && ln -sf "$(command -v fdfind)" ~/.local/bin/fd
have batcat && ln -sf "$(command -v batcat)" ~/.local/bin/bat
```

## Step 3 — GitHub CLI (official repo)

```bash
add_apt_repo github-cli https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  "deb https://cli.github.com/packages stable main"
sudo apt update && apt_install gh
```

## Step 4 — Git config and SSH key

Ask for name and email; set them only if empty.
```bash
[ -z "$(git config --global user.email)" ] && { git config --global user.name "<Full Name>"; git config --global user.email "<email>"; }
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global fetch.prune true
git config --global core.autocrlf input
git lfs install
[ -f ~/.ssh/id_ed25519 ] || ssh-keygen -t ed25519 -C "<email>" -f ~/.ssh/id_ed25519   # user types the passphrase
```
Tell the user to run `gh auth login` (SSH, upload the key) themselves.

## Step 5 — Shell niceties (idempotent)

```bash
# Starship prompt (official installer, downloaded to a file first)
have starship || { curl -fsSL https://starship.rs/install.sh -o /tmp/starship.sh && sh /tmp/starship.sh -y -b ~/.local/bin; }
add_rc() { grep -qxF "$1" ~/.bashrc || echo "$1" >> ~/.bashrc; }
add_rc 'export PATH="$HOME/.local/bin:$PATH"'
add_rc 'eval "$(starship init bash)"'
add_rc 'eval "$(zoxide init bash)"'
add_rc 'eval "$(direnv hook bash)"'
```
If the user prefers **zsh** or **fish**, ask before running `chsh`, and add the matching lines to
that shell's rc file.

Nerd Font for terminal icons:
```bash
mkdir -p ~/.local/share/fonts && cd /tmp &&
curl -fLO https://github.com/ryanoasis/nerd-fonts/releases/latest/download/JetBrainsMono.tar.xz &&
tar -xf JetBrainsMono.tar.xz -C ~/.local/share/fonts && fc-cache -f && cd -
```

## Step 6 — Flatpak + Flathub (desktop only; ask)

```bash
apt_install flatpak
[ "$XDG_CURRENT_DESKTOP" = "ubuntu:GNOME" ] && apt_install gnome-software-plugin-flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

## Step 7 — "Debloat" and privacy (ask; all optional)

Ubuntu is fairly lean. Offer these:
```bash
sudo apt purge -y ubuntu-report popularity-contest 2>/dev/null   # opt-out telemetry tools
sudo systemctl disable --now apport.service 2>/dev/null           # crash reporter (optional)
ubuntu-advantage config set apt_news=false 2>/dev/null || pro config set apt_news=false 2>/dev/null
```
**Don't** remove `snapd` without asking. Firefox, Thunderbird, and the App Center are snaps on
Ubuntu, so removing snapd removes them. If the user asks for it anyway: install the deb
alternatives first (Mozilla's APT repo for Firefox), then `sudo apt purge snapd`, and pin it with
`/etc/apt/preferences.d/nosnap.pref`.

## Step 8 — Security baseline (keep it ON)

```bash
apt_install ufw unattended-upgrades
sudo ufw default deny incoming; sudo ufw default allow outgoing
sudo ufw allow OpenSSH 2>/dev/null   # only if sshd is installed/used
sudo ufw --force enable
sudo dpkg-reconfigure -f noninteractive unattended-upgrades
```
Never disable AppArmor.

## Step 9 — Drivers (desktop only; ask)

```bash
ubuntu-drivers devices                 # shows the recommended driver
sudo ubuntu-drivers install            # installs the recommended (latest tested) driver
```
NVIDIA: the recommended driver comes as signed, prebuilt `linux-modules-nvidia-*` packages, so it
works with Secure Boot. Reboot afterwards. **Skip on WSL.**

## Step 10 — Performance/QoL (ask)

```bash
echo 'fs.inotify.max_user_watches=524288' | sudo tee /etc/sysctl.d/90-dev-inotify.conf && sudo sysctl --system  # big repos / IDE file watchers
apt_install preload 2>/dev/null || true       # optional
```

## Verify

```bash
gh --version | head -1; git --version; rg --version | head -1; starship --version | head -1; sudo ufw status | head -1
```
