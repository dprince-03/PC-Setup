# 00 — Preflight (Omarchy)

## Step 1 — Confirm Omarchy

```bash
. /etc/os-release; echo "$PRETTY_NAME | ID=$ID"           # ID=arch
export OMARCHY_PATH="${OMARCHY_PATH:-$HOME/.local/share/omarchy}"
[ -d "$OMARCHY_PATH" ] && command -v omarchy >/dev/null && echo "Omarchy OK" || echo "NOT Omarchy: use Linux/Arch/ and STOP"
omarchy-version 2>/dev/null || git -C "$OMARCHY_PATH" describe --tags 2>/dev/null
uname -srm; echo "Session: ${XDG_SESSION_TYPE:-?} ${XDG_CURRENT_DESKTOP:-?}"   # Hyprland / wayland
omarchy-update-available 2>/dev/null && echo "An Omarchy update is available"
```

## Step 2 — sudo, logs, snapshot

```bash
sudo -v
export LOG="$HOME/pc-setup-logs/$(date +%Y%m%d-%H%M%S)"; mkdir -p "$LOG"
echo "$(date -Is) [00] Omarchy $(omarchy-version 2>/dev/null) $(uname -r)" | tee "$LOG/os.txt"
omarchy-snapshot create || echo "Snapshot failed/unconfigured: tell the user and ask whether to continue"
```

## Step 3 — Inventory (Omarchy ships a lot, so this matters)

```bash
pacman -Qe > "$LOG/inventory-before-explicit.txt"; pacman -Qm > "$LOG/inventory-before-aur.txt"
mise ls > "$LOG/inventory-before-mise.txt" 2>/dev/null
docker --version; docker compose version; systemctl is-active docker
ls ~/.local/share/applications | grep -i -E 'webapp|\.desktop' | head -50 > "$LOG/webapps.txt"
```
Run the `probe` block from `Linux/Fedora/00-preflight.md` Step 3 → `$LOG/inventory-before-probes.txt`.
Expect many tools to show as present already. Classify them CURRENT and skip them.

## Step 4 — Helpers

```bash
log()  { echo "$(date -Is) [$1] $2" | tee -a "$LOG/run.log"; }
have() { command -v "$1" >/dev/null 2>&1; }
pkg()  { if omarchy-pkg-add "$@"; then log pkg "ok: $*"; else log pkg "FAILED: $*"; return 1; fi; }     # official repos
aurpkg(){ if omarchy-pkg-aur-add "$@"; then log aur "ok: $*"; else log aur "FAILED: $*"; return 1; fi; } # AUR (ask first)
flat() { flatpak info "$1" >/dev/null 2>&1 || { flatpak install -y flathub "$1" && log flatpak "installed $1"; }; }
```

## Step 5 — Plan → approval

`$LOG/plan.md`: mark items as *already in Omarchy*, *install via omarchy command*, or *AUR (needs a yes)*.
