# 12 — Security and Antivirus (Arch)

**Profiles:** security (default ON) · Turn protections on and audit them; never weaken anything.
**Never read secrets.** The user types passphrases and tokens.

## Step 1 — Audit (read-only)

```bash
{
echo "== firewall";    sudo ufw status 2>/dev/null || sudo firewall-cmd --state 2>/dev/null || sudo nft list ruleset | head
echo "== apparmor";    aa-enabled 2>/dev/null || echo "AppArmor not enabled"
echo "== secure boot"; bootctl status 2>/dev/null | grep -i 'secure boot'
echo "== luks";        lsblk -o NAME,TYPE,FSTYPE,MOUNTPOINTS | grep -E 'crypt|LUKS' || echo "no LUKS volumes"
echo "== updates";     checkupdates 2>/dev/null | wc -l
echo "== advisories";  arch-audit 2>/dev/null | head -20
echo "== listening";   sudo ss -tulpn | grep -v '127.0.0\|::1' | head -30
} | tee "$LOG/security-before.txt"
```

## Step 2 — Antivirus and scanners

```bash
pac clamav rkhunter lynis arch-audit
sudo freshclam                                        # first signature download
sudo systemctl enable --now clamav-freshclam.service  # automatic updates
sudo systemctl enable --now clamav-daemon.service     # clamd
clamdscan --multiscan --fdpass ~/Downloads
sudo rkhunter --update; sudo rkhunter --propupd; sudo rkhunter --check --sk --rwo
arch-audit                                            # installed packages with known CVEs (Arch Security Tracker)
sudo pacman -Qkk 2>/dev/null | grep -v ' 0 altered' | head -20   # package file integrity
```
Weekly home scan: the `clamscan-home` systemd user timer from `Linux/Ubuntu/12-security-antivirus.md` Step 2.

## Step 3 — Hardening

```bash
sudo ufw default deny incoming && sudo ufw default allow outgoing && sudo ufw --force enable   # from module 01
sudo tee /etc/sysctl.d/90-desktop-hardening.conf >/dev/null <<'EOF'
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.tcp_syncookies = 1
EOF
sudo sysctl --system >/dev/null
pac fwupd && fwupdmgr refresh && fwupdmgr get-updates      # ask before applying
```

| Control | Action |
|---|---|
| **Updates** | the biggest Arch security control: `sudo pacman -Syu` at least weekly (read Arch news); `yay -Sua` for AUR |
| AppArmor | needs kernel parameters (`lsm=landlock,lockdown,yama,integrity,apparmor,bpf`). **Explain this and let the user edit the bootloader themselves.** The agent doesn't touch the bootloader |
| Secure Boot | `sbctl` (`pac sbctl`) can create and enroll keys. That's a firmware-level change, so guide the user and don't do it automatically |
| LUKS | if missing, it needs a reinstall with encryption |
| sshd | off unless needed |
| App firewall (optional) | `pac opensnitch && sudo systemctl enable --now opensnitchd` (ask; it's interactive) |
| USBGuard (optional) | `pac usbguard`; generate the policy with your devices plugged in **before** enabling |
| Sandboxing | Flatpak + Flatseal, or `pac firejail` for specific apps |
| Encrypted DNS | systemd-resolved DoT drop-in (see Ubuntu 12) if resolved is in use |

## Step 4 — Developer security

Same as `Linux/Ubuntu/12-security-antivirus.md` Step 4 (no `.env` files with real values; `op/bws/infisical run`;
`pac libsecret` + `secret-tool` (the user types the values), or `pac pass`; `pac gitleaks` / `mise use -g osv-scanner@latest`;
pre-commit; SSH commit signing; GitHub 2FA and push protection).

## Step 5 — Verify

```bash
sudo lynis audit system --quick | tee "$LOG/lynis.txt" | tail -25
arch-audit | tee "$LOG/arch-audit.txt" | wc -l
systemctl is-active clamav-freshclam clamav-daemon; sudo ufw status | head -1
```
Add a **Security** section to `REPORT.md`.
