# 12 — Sysadmin and Networking (Windows)

**Profiles:** sysadmin-net (and linux-admin uses WSL from module 03) · **Admin:** yes

---

## Step 1 — Built-in Windows features (ask; some need a reboot)

```powershell
# OpenSSH client (usually present) and, optionally, the server
Get-WindowsCapability -Online -Name OpenSSH* | Select Name, State
# Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0     # only if the user wants inbound SSH

# Telnet client (handy for port checks), TFTP client
Enable-WindowsOptionalFeature -Online -FeatureName TelnetClient -NoRestart

# Hyper-V (Pro/Enterprise/Education only) and Windows Sandbox: ask
# Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All -All -NoRestart
# Enable-WindowsOptionalFeature -Online -FeatureName Containers-DisposableClientVM -All -NoRestart

# RSAT (AD, DNS, DHCP, Group Policy admin tools); only if the user manages Windows Server/AD
# Get-WindowsCapability -Online -Name RSAT* | Where State -ne Installed | Add-WindowsCapability -Online
```

## Step 2 — Tools

| Category         | Tool                                                               | winget ID                                                                           |
| ---------------- | ------------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| Diagnostics      | Sysinternals Suite                                                 | `Microsoft.Sysinternals.Suite`                                                    |
|                  | HWiNFO / CPU-Z                                                     | `REALiX.HWiNFO`, `CPUID.CPU-Z`                                                  |
|                  | WizTree (disk usage)                                               | `AntibodySoftware.WizTree`                                                        |
| Packet capture   | Wireshark (+ Npcap, which the installer offers)                    | `WiresharkFoundation.Wireshark`                                                   |
| Scanning         | Nmap (+ Zenmap)                                                    | `Insecure.Nmap`                                                                   |
| Throughput       | iperf3                                                             | `winget search iperf3` (pick the publisher that matches the ESnet/official build) |
| Remote access    | PuTTY                                                              | `PuTTY.PuTTY`                                                                     |
|                  | WinSCP (SFTP/SCP)                                                  | `WinSCP.WinSCP`                                                                   |
|                  | mRemoteNG (RDP/SSH/VNC manager)                                    | `mRemoteNG.mRemoteNG`                                                             |
|                  | Windows App (RDP client)                                           | `Microsoft.WindowsApp` / `Microsoft.RemoteDesktopClient`                        |
|                  | RustDesk (optional)                                                | `RustDesk.RustDesk`                                                               |
| VPN / overlay    | Tailscale                                                          | `Tailscale.Tailscale`                                                             |
|                  | WireGuard                                                          | `WireGuard.WireGuard`                                                             |
|                  | OpenVPN Connect                                                    | `OpenVPNTechnologies.OpenVPNConnect`                                              |
| HTTP debugging   | Fiddler Everywhere (optional)                                      | `Telerik.Fiddler.Everywhere` (check with `winget search Fiddler`)               |
| DNS              | `Resolve-DnsName` (built in), `dig` via WSL or BIND tools      |                                                                                     |
| Virtualization   | VirtualBox (if Hyper-V isn't used) / VMware Workstation Pro (free) | `Oracle.VirtualBox`, `winget search VMware`                                     |
| Serial / console | PuTTY (serial), Termius (optional)                                 |                                                                                     |
| Network calc     | `ipcalc` in WSL                                                  |                                                                                     |

```powershell
$net = 'Microsoft.Sysinternals.Suite','WiresharkFoundation.Wireshark','Insecure.Nmap','PuTTY.PuTTY',
       'WinSCP.WinSCP','Tailscale.Tailscale','WireGuard.WireGuard'
foreach ($id in $net) { Install-App $id }
```

Npcap's licence doesn't allow silent redistribution, so the Wireshark/Nmap installers may ask the
user to click through the Npcap installer. Let them.

## Step 3 — Linux admin practice (profile linux-admin)

Use WSL2 distros (module 03), plus these optional extras:

- A full VM for systemd/networking/boot practice: Hyper-V *Quick Create* or VirtualBox with an
  Ubuntu Server / Rocky / Debian ISO (always the newest release from the official site).
- Inside WSL Ubuntu, run `Linux/Ubuntu/10-sysadmin-networking.md`.

## Step 4 — PowerShell admin modules (ask)

```powershell
pwsh -c "Install-Module Microsoft.WinGet.Client, PSWindowsUpdate -Scope CurrentUser -Force"
```

## Verify

```powershell
nmap --version | Select -First 1
& "C:\Program Files\Wireshark\tshark.exe" --version | Select -First 1
ssh -V; tailscale version
```
