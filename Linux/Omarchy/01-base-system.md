# 01 — Base System (Omarchy)

**Profiles:** core

## Step 1 — Update the Omarchy way

```bash
omarchy update        # snapshot → pacman → AUR → mise → Omarchy migrations; the user types the sudo password once
```
Never run `pacman -Syu` by itself on Omarchy. Migrations would be skipped.

## Step 2 — Fill the gaps (most of the base is already there)

```bash
pkg git-lfs github-cli openssh wget tree ncdu direnv shellcheck p7zip man-pages
git lfs install
```

## Step 3 — Git / SSH

Ask for name and email if `git config --global user.email` is empty, set `init.defaultBranch main`,
create the ed25519 key if it's missing, and have the user run `gh auth login`. Same as Ubuntu 01 Step 4.
Shell: Omarchy already configures bash with starship/zoxide/mise. Only add lines that are missing,
in `~/.bashrc` (guarded with `grep -qxF`).

## Step 4 — "Debloat" (ask; show the list first)

```bash
cat "$(command -v omarchy-remove-preinstalls)" | tail -20     # show the user what gets removed
omarchy-remove-preinstalls                                    # interactive confirm
```
It removes Omarchy's web apps and TUIs, its default mise tool shims, and packages like
LibreOffice, Obsidian, Pinta, and some Omarchy apps. Individual web apps can also go via the
Omarchy menu (Super + Alt + Space → Remove).

## Step 5 — Security baseline

Omarchy enables **ufw** with `ufw-docker`. Check it, and don't weaken it:
```bash
sudo ufw status verbose
```

## Step 6 — Drivers

Omarchy's installer detects NVIDIA/Intel/AMD and configures them (`omarchy-hw-nvidia*` scripts).
Check with `nvidia-smi -L`. If the GPU is missing after a kernel update, run `omarchy update`
first. Don't hand-install drivers.

## Step 7 — Flatpak (optional, ask)

`pkg flatpak && flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo`

## Verify

`omarchy-version; gh --version | head -1; sudo ufw status | head -1; mise --version`
