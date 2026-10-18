# 12 — Security and Antivirus (Ubuntu)

**Profiles:** security (default ON) · This module **turns protections on and audits them**. It
never weakens anything (AGENT.md Rule 6). **Never read secrets** here (LUKS passphrases, tokens,
keys). The user types those themselves.

---

## Step 1 — Security audit (read-only)

```bash
{
echo "== firewall";        sudo ufw status verbose
echo "== apparmor";        sudo aa-status --summary 2>/dev/null || sudo aa-status | head -5
echo "== secure boot";     mokutil --sb-state 2>/dev/null
echo "== disk encryption"; lsblk -o NAME,TYPE,FSTYPE,MOUNTPOINTS | grep -E 'crypt|LUKS' || echo "no LUKS volumes found"
echo "== auto updates";    systemctl is-enabled unattended-upgrades 2>/dev/null; apt-config dump | grep -E 'Unattended-Upgrade "1"'
echo "== listening ports"; sudo ss -tulpn | grep -v '127.0.0\|::1' | head -30
echo "== sshd";            systemctl is-active ssh 2>/dev/null || echo "sshd not running"
echo "== firmware";        fwupdmgr get-updates 2>/dev/null | head -5
} | tee "$LOG/security-before.txt"
```
Show the user a table: **control | state | recommended**. Ask before changing anything.

## Step 2 — Antivirus and malware scanning

Linux desktop malware is rare. What matters most is **patching, sandboxing, and not running
untrusted scripts**. An AV scanner is still useful for downloaded files, USB drives, email
attachments, and files you'll pass to Windows users.

| Tool | Role | Install |
|---|---|---|
| **ClamAV** | open-source antivirus engine (on-demand scans; optional on-access) | `apt_install clamav clamav-daemon clamav-freshclam` |
| **rkhunter** / **chkrootkit** | rootkit and suspicious-binary checks | `apt_install rkhunter chkrootkit` |
| **Lynis** | full security audit + hardening suggestions | `apt_install lynis` |
| **debsums** | verifies installed package files against their checksums | `apt_install debsums` |
| AIDE (optional) | file integrity monitoring | `apt_install aide` |

```bash
apt_install clamav clamav-daemon clamav-freshclam rkhunter chkrootkit lynis debsums
sudo systemctl enable --now clamav-freshclam     # automatic signature updates
sudo systemctl enable --now clamav-daemon        # clamd for fast clamdscan
# First scans:
clamdscan --multiscan --fdpass ~/Downloads       # on-demand scan of Downloads
sudo rkhunter --update; sudo rkhunter --propupd  # baseline AFTER a clean install/update
sudo rkhunter --check --sk --rwo                 # report warnings only
sudo chkrootkit -q
sudo debsums -s                                  # silent unless a file has changed
```

Weekly scan of `$HOME` with a systemd user timer (ask):
```bash
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/clamscan-home.service <<'EOF'
[Unit]
Description=Weekly ClamAV scan of home
[Service]
Type=oneshot
ExecStart=/usr/bin/clamdscan --multiscan --fdpass --infected --log=%h/.local/state/clamscan.log %h
Nice=19
IOSchedulingClass=idle
EOF
cat > ~/.config/systemd/user/clamscan-home.timer <<'EOF'
[Unit]
Description=Weekly ClamAV scan
[Timer]
OnCalendar=weekly
Persistent=true
RandomizedDelaySec=2h
[Install]
WantedBy=timers.target
EOF
mkdir -p ~/.local/state && systemctl --user daemon-reload && systemctl --user enable --now clamscan-home.timer
```
**On-access scanning** (`clamonacc`) uses a lot of CPU/RAM and can slow builds. Only enable it if the
user asks, and exclude nothing important.

## Step 3 — System hardening

```bash
# Firewall (module 01 enabled it). Re-assert the policy:
sudo ufw default deny incoming && sudo ufw default allow outgoing && sudo ufw --force enable
# AppArmor must be enforcing
sudo systemctl enable --now apparmor && sudo aa-status | head -3
# Automatic security updates (module 01)
sudo dpkg-reconfigure -f noninteractive unattended-upgrades
# Firmware updates (desktop): review, then apply
fwupdmgr refresh && fwupdmgr get-updates     # ask before: fwupdmgr update
# Kernel/network hardening (safe on desktops; Docker needs ip_forward, which Docker sets itself)
sudo tee /etc/sysctl.d/90-desktop-hardening.conf >/dev/null <<'EOF'
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
kernel.unprivileged_bpf_disabled = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1
EOF
sudo sysctl --system >/dev/null
```

| Control | Action |
|---|---|
| **Disk encryption (LUKS)** | if Step 1 found none: **tell the user**. Full-disk encryption can't be safely added in place, so it needs a reinstall with "Encrypt the new installation". Never attempt it automatically |
| **Secure Boot** | if disabled, *tell the user* how to enable it in UEFI (the NVIDIA modules from `ubuntu-drivers` are signed) |
| **SSH server** | keep it **off** unless needed. If needed: keys only, no root (see Server-Setup `Servers/files/configs/sshd/99-hardening.conf`) |
| **App sandboxing** | prefer Flatpak/Snap for untrusted GUI apps; `flatpak install flathub com.github.tchx84.Flatseal` to review permissions |
| **Application firewall** (optional) | OpenSnitch (prompts per app for outbound connections): `.deb` from github.com/evilsocket/opensnitch/releases. Ask, because it's interactive |
| **USB device control** (optional) | `apt_install usbguard`. Generate the policy with the current devices plugged in **before** enabling, or the keyboard can be blocked |
| **Encrypted DNS** | systemd-resolved DoT: `/etc/systemd/resolved.conf.d/dot.conf` with `DNS=9.9.9.9#dns.quad9.net` + `DNSOverTLS=yes` → `sudo systemctl restart systemd-resolved` (ask; corporate networks may need their own DNS) |
| **Screen lock** | auto-lock ≤ 5 min: `gsettings set org.gnome.desktop.session idle-delay 300` (GNOME) |

## Step 4 — Developer security

### Secrets: no `.env` files with real values
- Personal secrets: the **password manager** (module 11).
- Project secrets at runtime: `op run -- <cmd>` / `bws run -- <cmd>` / `infisical run -- <cmd>`. Commit only reference templates.
- Local dev passwords (dev databases): the **GNOME keyring** via `secret-tool`. The **user** types the value:
  ```bash
  apt_install libsecret-tools
  secret-tool store --label="dev postgres" service pc-setup key pg_password    # prompts for the value (hidden)
  ```
  Headless/WSL without a keyring: use `pass` (GPG-encrypted, `apt_install pass`), or the secret manager CLI.
  Scripts fetch values only to hand them to the process that needs them. **The agent never prints them.**

### Code and supply chain
```bash
mise use -g gitleaks@latest                       # leaked-secret scanner
uv tool install pre-commit                        # gitleaks/linters on every commit
mise use -g osv-scanner@latest                    # dependency vulnerability scanner (OSV.dev)
go install golang.org/x/vuln/cmd/govulncheck@latest 2>/dev/null
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true && git config --global tag.gpgsign true
```
The user adds the key on GitHub as a **Signing key** (`gh ssh-key add ~/.ssh/id_ed25519.pub --type signing`)
and turns on 2FA (passkey) and push protection. Per project: `npm audit`, `uv tool run pip-audit`,
`govulncheck ./...`, `trivy fs .`.

## Step 5 — Browser and backups

- Browser: auto-updates, HTTPS-only mode, uBlock Origin from the official store, and the password manager extension.
- Backups: **Timeshift** for system snapshots (`apt_install timeshift`) + **Déjà Dup** or **restic** for home
  data to an external drive or cloud (3-2-1). Encrypt the backups. The user sets the password.

## Step 6 — Audit and verify

```bash
sudo lynis audit system --quick | tee "$LOG/lynis.txt" | tail -25    # note the "Hardening index"
sudo ufw status | head -1; sudo aa-status --enabled && echo "AppArmor enforcing"
systemctl is-active clamav-freshclam clamav-daemon
sudo rkhunter --check --sk --rwo | tee "$LOG/rkhunter.txt"
```
Add a **Security** section to `REPORT.md`: before → after per control, the Lynis hardening index,
scan results, and the user's to-dos (LUKS reinstall if missing, Secure Boot, GitHub signing key/2FA, backup password).
