# 12 — Verify and Report (Fedora)

Open a new login shell (`exec bash -l`).

```bash
LOG=$(ls -d ~/pc-setup-logs/*/ | sort | tail -1)
rpm -qa --qf '%{NAME} %{VERSION}-%{RELEASE}\n' | sort > "$LOG/inventory-after-rpm.txt"
flatpak list --app > "$LOG/inventory-after-flatpak.txt" 2>/dev/null
mise ls --current > "$LOG/mise-tools.txt" 2>/dev/null; uv tool list > "$LOG/uv-tools.txt" 2>/dev/null
dnf check-update -q > "$LOG/upgrades-remaining.txt" 2>/dev/null
```
Run the `check` smoke-test table from `Linux/Ubuntu/12-verify-and-report.md` (same commands; swap
`redis-cli` for `valkey-cli` if Valkey was chosen, and add `podman --version` and `getenforce`).

Write `$LOG/REPORT.md` (AGENT.md Rule 10). Fedora-specific follow-ups: reboot after the kernel/akmod
update, MOK enrolment screen (if Secure Boot + NVIDIA), log out/in for the docker/kvm/libvirt/wireshark
groups, `gh auth login`, Android Studio first run.

**Keep it updated:** `sudo dnf upgrade --refresh` · `flatpak update` · `mise upgrade` ·
`uv self update && uv tool upgrade --all` · `sudo fwupdmgr update` · release upgrades with
`sudo dnf system-upgrade download --releasever=<N>` (ask).
