# 10 — Sysadmin, Networking and Linux Admin (Ubuntu)

**Profiles:** sysadmin-net, linux-admin

## Packages

```bash
# Networking and diagnostics
apt_install net-tools iproute2 iputils-ping traceroute mtr-tiny dnsutils whois nmap netcat-openbsd \
  socat tcpdump iperf3 ethtool nethogs iftop bmon curl wget openssl ipcalc arp-scan wireguard-tools
apt_install wireshark            # the user picks "allow non-superusers" in the prompt; then:
sudo usermod -aG wireshark "$USER"
# Remote access
apt_install openssh-client openssh-server mosh rsync sshfs remmina
sudo systemctl disable --now ssh 2>/dev/null  # keep sshd OFF unless the user wants inbound SSH (ask)
# System admin
apt_install sysstat iotop lsof strace ltrace smartmontools lm-sensors nvme-cli inxi hwinfo \
  acl attr auditd logrotate cron at fail2ban needrestart debsums apt-file
sudo apt-file update
# Virtualization for lab VMs (desktop; ask)
apt_install qemu-kvm libvirt-daemon-system libvirt-clients virtinst virt-manager bridge-utils
sudo usermod -aG libvirt,kvm "$USER"
```

## Tools via other sources (ask)

| Tool | Install |
|---|---|
| Tailscale | official script, saved to a file: `curl -fsSL https://tailscale.com/install.sh -o /tmp/ts.sh && sh /tmp/ts.sh` |
| Cockpit (web admin UI) | `apt_install cockpit` (listens on :9090; ask, and firewall it) |
| Multipass (Ubuntu VMs) | `snap_install multipass` |
| LXD / Incus (containers + VMs) | `snap_install lxd` or `apt_install incus` |
| Vagrant | HashiCorp APT repo |
| Termius (optional GUI SSH) | `snap_install termius-app` |

## Linux-admin practice suggestions (tell the user)

- systemd: write a unit + timer, `journalctl -u`, `systemd-analyze blame`
- storage: LVM on loop devices (`losetup`), `mdadm`, filesystems (ext4/xfs/btrfs)
- networking: `ip`, `nmcli`/netplan, `ufw`/`nftables`, network namespaces (`ip netns`)
- security: `auditd`, AppArmor profiles, `fail2ban`, SSH hardening (`/etc/ssh/sshd_config.d/`)
- Lab VMs: Multipass or virt-manager running Ubuntu Server, Debian, Rocky, Alma

## Verify

```bash
nmap --version | head -1; tcpdump --version 2>&1 | head -1; iperf3 --version | head -1; virsh --version 2>/dev/null
```
