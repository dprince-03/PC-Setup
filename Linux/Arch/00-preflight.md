# 00 — Preflight (Arch)

## Step 1 — Confirm the OS (and that it's *not* Omarchy)

```bash
. /etc/os-release; echo "$PRETTY_NAME | ID=$ID | ID_LIKE=${ID_LIKE:-}"
uname -srm
{ [ -d "$HOME/.local/share/omarchy" ] || command -v omarchy >/dev/null; } && echo "OMARCHY: switch to Linux/Omarchy/ and STOP here"
IS_WSL=$(grep -qi microsoft /proc/version && echo 1 || echo 0); echo "WSL=$IS_WSL"
pacman -Q linux linux-lts linux-zen 2>/dev/null     # installed kernels (matters for NVIDIA/dkms)
findmnt -no FSTYPE /; bootctl status 2>/dev/null | head -3; mokutil --sb-state 2>/dev/null
echo "Desktop: ${XDG_CURRENT_DESKTOP:-none} (${XDG_SESSION_TYPE:-tty})"
```
- `ID` must be `arch`. If it's an Arch derivative (Manjaro/EndeavourOS/CachyOS), ask whether to
  proceed with this playbook. Manjaro holds packages back, so warn about AUR breakage.

## Step 2 — sudo, logs, snapshot

```bash
sudo -v
export LOG="$HOME/pc-setup-logs/$(date +%Y%m%d-%H%M%S)"; mkdir -p "$LOG"
echo "$(date -Is) [00] $PRETTY_NAME $(uname -r) WSL=$IS_WSL" | tee "$LOG/os.txt"
if command -v snapper >/dev/null && sudo snapper list-configs | grep -q root; then sudo snapper -c root create -d "PC-Setup before changes"
elif command -v timeshift >/dev/null; then sudo timeshift --create --comments "PC-Setup before changes"
else echo "No snapshot tool. Ask: set up snapper (btrfs) / timeshift, or continue without?"; fi
```

## Step 3 — Inventory

```bash
pacman -Q   > "$LOG/inventory-before-pacman.txt"
pacman -Qm  > "$LOG/inventory-before-aur.txt"        # foreign (AUR/manual) packages
pacman -Qe  > "$LOG/inventory-before-explicit.txt"
flatpak list --app 2>/dev/null > "$LOG/inventory-before-flatpak.txt"
command -v yay paru 2>/dev/null                        # existing AUR helper? Use the one that's there
```
Run the same `probe` block as `Linux/Fedora/00-preflight.md` Step 3 (it's distro-neutral) and save it to
`$LOG/inventory-before-probes.txt`. Also `lspci | grep -Ei 'vga|3d'`.

## Step 4 — Helpers

```bash
log()  { echo "$(date -Is) [$1] $2" | tee -a "$LOG/run.log"; }
have() { command -v "$1" >/dev/null 2>&1; }
pac()  {   # official repos only; --needed skips anything already current
  if sudo pacman -S --needed --noconfirm "$@"; then log pacman "ok: $*"; else log pacman "FAILED: $*"; return 1; fi; }
aur()  {   # AUR via yay (or paru); only after the user has approved each package
  local h; h=$(command -v yay || command -v paru) || { echo "no AUR helper (see 01)"; return 1; }
  if "$h" -S --needed --noconfirm "$@"; then log aur "ok: $*"; else log aur "FAILED: $*"; return 1; fi; }
flat() { flatpak info "$1" >/dev/null 2>&1 || { flatpak install -y flathub "$1" && log flatpak "installed $1"; }; }
```

## Step 5 — Plan → approval

Write `$LOG/plan.md` and mark which items come from the **AUR** (they need an extra yes). Wait for approval.
