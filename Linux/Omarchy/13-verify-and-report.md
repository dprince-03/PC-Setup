# 13 — Verify and Report (Omarchy)

Open a new terminal (Super + Enter).

```bash
LOG=$(ls -d ~/pc-setup-logs/*/ | sort | tail -1)
pacman -Qe > "$LOG/inventory-after-explicit.txt"; pacman -Qm > "$LOG/inventory-after-aur.txt"
mise ls --current > "$LOG/mise-tools.txt"; uv tool list > "$LOG/uv-tools.txt" 2>/dev/null
omarchy-version > "$LOG/omarchy-version.txt" 2>/dev/null
```
Run the `check` smoke-test table from `Linux/Ubuntu/13-verify-and-report.md`.

`$LOG/REPORT.md` (AGENT.md Rule 10). Omarchy follow-ups: log out/in for group changes (kvm, wireshark,
libvirt, docker), sign in to the IDEs, `gh auth login`, and remind the user that **`omarchy update`**
is the only update command they need (it covers pacman, AUR, mise and migrations, and takes a
snapshot first).
