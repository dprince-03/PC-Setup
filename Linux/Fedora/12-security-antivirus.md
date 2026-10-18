# 12 — Security and Antivirus (Fedora)

**Profiles:** security (default ON) · Turn protections on and audit them; never weaken anything.
**Never read secrets** (LUKS passphrases, tokens, keys). The user types them.

## Step 1 — Audit (read-only)

```bash
{
echo "== firewalld"; sudo firewall-cmd --state; sudo firewall-cmd --get-active-zones
echo "== selinux";   getenforce; sestatus | head -5
echo "== secure boot"; mokutil --sb-state 2>/dev/null
echo "== luks";      lsblk -o NAME,TYPE,FSTYPE,MOUNTPOINTS | grep -E 'crypt|LUKS' || echo "no LUKS volumes"
echo "== auto updates"; systemctl is-enabled dnf-automatic.timer 2>/dev/null || echo "dnf-automatic off"
echo "== listening"; sudo ss -tulpn | grep -v '127.0.0\|::1' | head -30
echo "== firmware";  fwupdmgr get-updates 2>/dev/null | head -5
} | tee "$LOG/security-before.txt"
```

## Step 2 — Antivirus and scanners

```bash
dnf_install clamav clamav-update clamd rkhunter lynis
sudo systemctl enable --now clamav-freshclam      # signature updates
# clamd: enable the scanner service (Fedora ships the clamd@scan instance; check it exists first)
sudo sed -i 's/^#LocalSocket /LocalSocket /' /etc/clamd.d/scan.conf
sudo systemctl enable --now clamd@scan
clamdscan --multiscan --fdpass ~/Downloads 2>/dev/null || clamscan -ri ~/Downloads
sudo rkhunter --update; sudo rkhunter --propupd; sudo rkhunter --check --sk --rwo
sudo rpm -Va --nomtime --nosize --nouser --nogroup 2>/dev/null | grep -v '^.......T' | head -20   # package file integrity (review the output, don't panic over config files)
```
Weekly home scan: create the same `clamscan-home` systemd **user** timer as `Linux/Ubuntu/12-security-antivirus.md` Step 2.
SELinux may label `~/.local/state` differently. If the scan fails, check with `ausearch -m avc -ts recent`.

## Step 3 — Hardening

```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --set-default-zone=public     # FedoraWorkstation zone opens ports 1025-65535; public is stricter (ask)
getenforce                                      # Enforcing: never setenforce 0
dnf_install dnf-automatic && sudo sed -i 's/^upgrade_type.*/upgrade_type = security/; s/^apply_updates.*/apply_updates = yes/' /etc/dnf/automatic.conf && sudo systemctl enable --now dnf-automatic.timer
fwupdmgr refresh --force && fwupdmgr get-updates     # ask before: fwupdmgr update
sudo tee /etc/sysctl.d/90-desktop-hardening.conf >/dev/null <<'EOF'
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.tcp_syncookies = 1
EOF
sudo sysctl --system >/dev/null
```

| Control | Action |
|---|---|
| LUKS | if missing, tell the user it needs a reinstall with encryption enabled. Never try it automatically |
| Secure Boot | if on with NVIDIA, the akmod MOK enrolment from module 01 is needed |
| sshd | off unless needed (`sudo systemctl disable --now sshd`, ask) |
| Sandboxing | Flatpak apps + Flatseal (`flat com.github.tchx84.Flatseal`) |
| App firewall (optional) | OpenSnitch: `dnf_install opensnitch` if packaged for this release, otherwise the upstream RPM from github.com/evilsocket/opensnitch/releases |
| USBGuard (optional) | `dnf_install usbguard`, then `sudo sh -c 'usbguard generate-policy > /etc/usbguard/rules.conf'` **with your devices plugged in**, then enable |
| Encrypted DNS | systemd-resolved is the default: add the `DNSOverTLS=yes` drop-in (see Ubuntu 12) |

## Step 4 — Developer security

Same as `Linux/Ubuntu/12-security-antivirus.md` Step 4: no `.env` files with real values; use
`op run`/`bws run`/`infisical run`; local dev passwords in the keyring with
`dnf_install libsecret` + `secret-tool store ...` (the user types them); gitleaks, osv-scanner and
pre-commit via mise/uv; SSH commit signing; GitHub 2FA and push protection.

## Step 5 — Verify

```bash
sudo lynis audit system --quick | tee "$LOG/lynis.txt" | tail -25
getenforce; sudo firewall-cmd --get-default-zone; systemctl is-active clamav-freshclam clamd@scan
```
Add a **Security** section to `REPORT.md` (before → after, Lynis index, and the user's to-dos).
