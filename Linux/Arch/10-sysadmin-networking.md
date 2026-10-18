# 10 — Sysadmin, Networking, Linux Admin (Arch)

**Profiles:** sysadmin-net, linux-admin

```bash
pac net-tools iproute2 iputils traceroute mtr bind whois nmap gnu-netcat socat tcpdump iperf3 ethtool \
  nethogs iftop bmon openssl ipcalc arp-scan wireguard-tools wireshark-qt
sudo usermod -aG wireshark "$USER"
pac openssh mosh rsync sshfs remmina freerdp
pac sysstat iotop lsof strace ltrace smartmontools lm_sensors nvme-cli inxi hwinfo acl audit logrotate cronie fail2ban
# sshd stays off unless the user wants inbound SSH (ask): sudo systemctl enable --now sshd
# Virtualization lab (ask)
pac qemu-full libvirt virt-manager dnsmasq edk2-ovmf swtpm && sudo systemctl enable --now libvirtd && sudo usermod -aG libvirt "$USER"
# Others (ask)
pac tailscale && sudo systemctl enable --now tailscaled
pac cockpit 2>/dev/null; pac incus 2>/dev/null
```

**Arch-specific admin practice:** the Arch Wiki is the manual. Practise systemd units/timers,
`journalctl`, `mkinitcpio`, bootloaders (in a VM only), `pacman -Qkk` integrity checks, `pacdiff`
(`pac pacman-contrib`), and writing PKGBUILDs.

## Verify

`nmap --version | head -1; tshark --version | head -1; virsh --version`
