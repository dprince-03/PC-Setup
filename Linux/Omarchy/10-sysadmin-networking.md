# 10 — Sysadmin, Networking, Linux Admin (Omarchy)

**Profiles:** sysadmin-net, linux-admin · Same package set as `Linux/Arch/10-sysadmin-networking.md`, with
`pkg` instead of `pac` (Omarchy already has `inetutils`, `whois`, `btop`, `inxi`, `ufw`).

```bash
pkg nmap tcpdump mtr bind iperf3 socat ethtool wireshark-qt wireguard-tools openssh mosh rsync sshfs \
    sysstat lsof strace smartmontools nvme-cli lm_sensors
sudo usermod -aG wireshark "$USER"
pkg qemu-full libvirt virt-manager dnsmasq edk2-ovmf swtpm && sudo systemctl enable --now libvirtd && sudo usermod -aG libvirt "$USER"   # ask
```
Omarchy-specific admin skills: Hyprland config (`~/.config/hypr`), `omarchy-refresh-*` to restore
defaults, snapper rollbacks (`omarchy-snapshot restore`), `journalctl`, systemd user units.
