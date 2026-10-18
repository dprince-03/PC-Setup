# 10 — Sysadmin, Networking, Linux Admin (Fedora)

**Profiles:** sysadmin-net, linux-admin

```bash
dnf_install net-tools iproute iputils traceroute mtr bind-utils whois nmap nmap-ncat socat tcpdump \
  iperf3 ethtool nethogs iftop bmon openssl ipcalc arp-scan wireguard-tools wireshark
sudo usermod -aG wireshark "$USER"
dnf_install openssh-clients openssh-server mosh rsync fuse-sshfs remmina
dnf_install sysstat iotop lsof strace ltrace smartmontools lm_sensors nvme-cli inxi hwinfo acl attr \
  audit logrotate cronie fail2ban setroubleshoot-server policycoreutils-python-utils
sudo systemctl disable --now sshd 2>/dev/null   # leave it off unless the user wants inbound SSH (ask)
# Virtualization lab (ask)
sudo dnf group install -y virtualization && sudo systemctl enable --now libvirtd && sudo usermod -aG libvirt "$USER"
# Cockpit web console (ask; Fedora Server has it by default)
dnf_install cockpit && sudo systemctl enable --now cockpit.socket && sudo firewall-cmd --add-service=cockpit --permanent && sudo firewall-cmd --reload
```

**Other tools (ask):** Tailscale (official repo `https://pkgs.tailscale.com/stable/fedora/tailscale.repo`
→ `dnf_install tailscale`), Vagrant (HashiCorp repo), Incus (`dnf_install incus`).

**Fedora-specific admin practice:** SELinux (`semanage`, `audit2allow`, booleans), `firewalld` zones,
`nmcli`, `systemd` units/timers, `dnf history undo`, Btrfs subvolumes and snapper.

## Verify

`nmap --version | head -1; tshark --version | head -1; virsh --version; sestatus | head -1`
