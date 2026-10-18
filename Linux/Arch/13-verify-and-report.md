# 13 — Verify and Report (Arch)

Open a new login shell (`exec bash -l`).

```bash
LOG=$(ls -d ~/pc-setup-logs/*/ | sort | tail -1)
pacman -Q > "$LOG/inventory-after-pacman.txt"; pacman -Qm > "$LOG/inventory-after-aur.txt"
flatpak list --app > "$LOG/inventory-after-flatpak.txt" 2>/dev/null
mise ls --current > "$LOG/mise-tools.txt" 2>/dev/null; uv tool list > "$LOG/uv-tools.txt" 2>/dev/null
checkupdates > "$LOG/upgrades-remaining.txt" 2>/dev/null   # from pacman-contrib
```
Run the `check` smoke-test table from `Linux/Ubuntu/13-verify-and-report.md` (swap `redis-cli` for
`valkey-cli` if Valkey was chosen).

Write `$LOG/REPORT.md` (AGENT.md Rule 10). Arch follow-ups: reboot after kernel/NVIDIA updates, log
out/in for group changes, list every **AUR** package installed (they need `yay -Sua` to update),
and `pacdiff` for any `.pacnew` files.

**Keep it updated:** `sudo pacman -Syu` (read Arch news first) · `yay -Sua` · `flatpak update` ·
`mise upgrade` · `uv self update && uv tool upgrade --all` · `rustup update`
