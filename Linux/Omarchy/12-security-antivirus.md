# 12 — Security and Antivirus (Omarchy)

**Profiles:** security (default ON) · Omarchy already gives you a good base: **full-disk
encryption (LUKS) by default** from its installer, **ufw + ufw-docker** enabled, snapper snapshots,
and `omarchy update` for patching. This module audits that and adds scanning.
**Never read secrets.** The user types passphrases and tokens.

## Step 1 — Audit (read-only)

```bash
{
echo "== ufw";        sudo ufw status verbose
echo "== luks";       lsblk -o NAME,TYPE,FSTYPE,MOUNTPOINTS | grep -E 'crypt|LUKS' || echo "no LUKS volumes (unusual for Omarchy)"
echo "== snapshots";  sudo snapper list 2>/dev/null | tail -3
echo "== updates";    omarchy-update-available 2>/dev/null && echo "update available" || echo "up to date"
echo "== advisories"; arch-audit 2>/dev/null | head -20
echo "== listening";  sudo ss -tulpn | grep -v '127.0.0\|::1' | head -30
} | tee "$LOG/security-before.txt"
```

## Step 2 — Antivirus and scanners

```bash
pkg clamav rkhunter lynis arch-audit
sudo freshclam
sudo systemctl enable --now clamav-freshclam.service clamav-daemon.service
clamdscan --multiscan --fdpass ~/Downloads
sudo rkhunter --update; sudo rkhunter --propupd; sudo rkhunter --check --sk --rwo
arch-audit
```
Weekly home scan: the `clamscan-home` systemd user timer from `Linux/Ubuntu/12-security-antivirus.md` Step 2.

## Step 3 — Hardening (on top of Omarchy's defaults)

| Control | Action |
|---|---|
| Updates | `omarchy update` (it takes a snapshot first). Weekly |
| Firewall | Omarchy's ufw config: keep it, and add rules only for what you expose |
| Docker | the docker group is root-equivalent. Omarchy's sudoless-docker helper is opt-in, so explain before using it |
| Kernel sysctl | same drop-in as `Linux/Arch/12-security-antivirus.md` Step 3 |
| App firewall / USBGuard (optional) | `pkg opensnitch` / `pkg usbguard` (generate the USBGuard policy with your devices plugged in first) |
| Secure Boot | Omarchy uses Limine. Secure Boot needs `sbctl` key enrolment, which is a firmware change, so guide the user only |
| Screen lock | Omarchy's hyprlock/hypridle are on by default. Check the timeout in `~/.config/hypr/hypridle.conf` |

## Step 4 — Developer security

Same as `Linux/Ubuntu/12-security-antivirus.md` Step 4. gnome-keyring is already part of Omarchy, so
`pkg libsecret` gives you `secret-tool` (the user types the values). Use `op/bws/infisical run` for
project secrets, with no `.env` files holding real values. Add gitleaks (`pkg gitleaks`), osv-scanner,
pre-commit, SSH commit signing, and GitHub 2FA/push protection. Omarchy has a 1Password installer:
`omarchy-install-service-1password`.

## Step 5 — Verify

```bash
sudo lynis audit system --quick | tee "$LOG/lynis.txt" | tail -25
systemctl is-active clamav-freshclam clamav-daemon; sudo ufw status | head -1; arch-audit | wc -l
```
Add a **Security** section to `REPORT.md`.
