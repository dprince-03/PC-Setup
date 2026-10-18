# 13 — Verify and Report (Ubuntu)

Open a **new login shell** first (`exec bash -l`) so PATH and group changes are picked up.

```bash
LOG=$(ls -d ~/pc-setup-logs/*/ | sort | tail -1)
apt list --installed 2>/dev/null > "$LOG/inventory-after-apt.txt"
snap list 2>/dev/null > "$LOG/inventory-after-snap.txt"; flatpak list --app 2>/dev/null > "$LOG/inventory-after-flatpak.txt"
mise ls --current 2>/dev/null > "$LOG/mise-tools.txt"; uv tool list > "$LOG/uv-tools.txt" 2>/dev/null
apt list --upgradable 2>/dev/null > "$LOG/upgrades-remaining.txt"

check() { local n=$1; shift; if out=$("$@" 2>&1 | head -1); then printf '| %-10s | ✔ | %s |\n' "$n" "$out"; else printf '| %-10s | ✘ | %s |\n' "$n" "$out"; fi; }
{
echo '| Item | OK | Detail |'; echo '|---|---|---|'
check git git --version; check gh gh --version; check python uv run --no-project python --version
check node node -v; check pnpm pnpm -v; check java java -version; check maven mvn -v; check gradle gradle -v
check go go version; check gcc gcc --version; check clang clang --version; check dotnet dotnet --version
check docker docker run --rm hello-world; check kubectl kubectl version --client; check helm helm version --short
check terraform terraform -version; check ansible ansible --version; check aws aws --version
check code code --version; check adb adb version; check flutter flutter --version
check postgres psql -h 127.0.0.1 -U postgres -c 'select 1' ; check redis redis-cli -h 127.0.0.1 ping
check nvidia nvidia-smi -L
check clamav systemctl is-active clamav-freshclam; check firewall sudo ufw status; check gitleaks gitleaks version
check apparmor sudo aa-status --enabled
} | tee "$LOG/smoke-tests.md"
```
Remove rows for profiles that weren't selected.

Write `$LOG/REPORT.md` with the AGENT.md Rule 10 structure. Manual follow-ups to list:
log out/in (docker, kvm, wireshark, libvirt groups), reboot if `/var/run/reboot-required` exists,
`gh auth login`, open Android Studio once, sign in to JetBrains/Postman, `aws/az/gcloud` logins.

**Keep it updated:** `sudo apt update && sudo apt full-upgrade` · `snap refresh` · `flatpak update` ·
`mise upgrade` · `uv self update && uv tool upgrade --all` · `rustup update`
