# 14 — Security and Antivirus (Windows)

**Profiles:** security (default ON) · **Admin:** yes · **Reboot:** maybe (memory integrity, offline scan)

Goal: a developer machine that's **hard to compromise, quick to recover, and never leaks secrets**.
This module **turns protections on and checks them**. It never turns anything off (AGENT.md Rule 6).

> **Never read secrets in this module:** BitLocker recovery keys, passwords, tokens. Where a step
> involves one, the **user** does it in the UI or at a prompt they type into.

---

## Step 1 — Security status audit (read-only)

```powershell
# Antivirus (Microsoft Defender)
Get-MpComputerStatus | Select AMServiceEnabled, AntivirusEnabled, RealTimeProtectionEnabled, IsTamperProtected,
  BehaviorMonitorEnabled, AntivirusSignatureLastUpdated, QuickScanAge, FullScanAge
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntiVirusProduct | Select displayName, productState  # any third-party AV?
# Firewall
Get-NetFirewallProfile | Select Name, Enabled, DefaultInboundAction
# Secure Boot / TPM (read only: never change firmware settings)
Confirm-SecureBootUEFI; Get-Tpm | Select TpmPresent, TpmReady, ManufacturerVersion
# Disk encryption status (status only: do NOT run commands that display recovery keys)
Get-BitLockerVolume -MountPoint C: | Select MountPoint, VolumeStatus, ProtectionStatus, EncryptionPercentage
# Virtualization-based security / memory integrity
Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard |
  Select VirtualizationBasedSecurityStatus, SecurityServicesRunning
# LSA protection, SmartScreen, Smart App Control
Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa' -Name RunAsPPL -EA 0 | Select RunAsPPL
Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\CI\Policy' -Name VerifiedAndReputablePolicyState -EA 0  # Smart App Control: 0 off, 1 on, 2 evaluation
# Pending updates
winget upgrade --include-unknown | Select-Object -Last 5
```
Write the results to `$Log\security-before.txt` and show the user a table: **control | state | recommended**.

## Step 2 — Antivirus: which one?

| Option | Recommendation |
|---|---|
| **Microsoft Defender Antivirus** (built in) | ✔ **Default.** It scores consistently well in independent tests (AV-TEST, AV-Comparatives), costs nothing, integrates with SmartScreen, ASR and Dev Drive, and adds no bloat |
| Third-party real-time AV (Bitdefender, ESET, Norton, Kaspersky...) | Only if the user asks. Install **one** real-time AV: it registers with Security Center and Defender's real-time protection steps aside automatically. Never run two real-time engines |
| **Second-opinion, on-demand scanners** (no real-time component) | ✔ optional: **Microsoft Safety Scanner** (MSERT, from microsoft.com), **Malwarebytes** (free version as an on-demand scanner only), ESET Online Scanner |

Second-opinion scanner (ask):
```powershell
Install-App 'Malwarebytes.Malwarebytes'   # tell the user: decline or let the Premium trial lapse, and keep it as an on-demand scanner so it doesn't fight Defender
```

## Step 3 — Harden Microsoft Defender (ask once, apply as a batch)

```powershell
Update-MpSignature
Set-MpPreference -MAPSReporting Advanced -SubmitSamplesConsent SendSafeSamples   # cloud-delivered protection
Set-MpPreference -CloudBlockLevel High -CloudExtendedTimeout 50
Set-MpPreference -PUAProtection Enabled                                            # block potentially unwanted apps (bundleware, adware)
Set-MpPreference -EnableNetworkProtection Enabled                                  # blocks known-bad domains for all apps
Set-MpPreference -DisableRealtimeMonitoring $false -DisableBehaviorMonitoring $false -DisableIOAVProtection $false
Set-MpPreference -ScanAvgCPULoadFactor 30                                          # keep scheduled scans light
```
- **Tamper Protection:** must be **On**. It can only be turned on in the UI: *Windows Security → Virus &
  threat protection → Manage settings*. Ask the user to check it.
- **No folder exclusions** for repos or `node_modules`. For build speed, use a **Dev Drive** (module 01)
  in performance mode instead. That's the supported way.

### Attack Surface Reduction (ASR) rules: Audit first, then Block

These rules stop common malware techniques. Start in **Audit** mode (2), review the events for a
week (*Event Viewer → Microsoft-Windows-Windows Defender/Operational*, IDs 1121/1122), then switch
to **Block** (1). The rules below rarely affect developers. Rules that break builds (prevalence/age
checks, PsExec/WMI blocking) are deliberately left out.
```powershell
$asr = @{
  'BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550' = 'Block executable content from email client and webmail'
  'D4F940AB-401B-4EFC-AADC-AD5F3C50688A' = 'Block Office apps from creating child processes'
  '3B576869-A4EC-4529-8536-B80A7769E899' = 'Block Office apps from creating executable content'
  '75668C1F-73B5-4CF0-BB93-3ECF5CB7CC84' = 'Block Office apps from injecting code into other processes'
  'D3E037E1-3EB8-44C8-A917-57927947596D' = 'Block JS/VBScript from launching downloaded executables'
  '5BEB7EFE-FD9A-4556-801D-275E5FFC04CC' = 'Block execution of potentially obfuscated scripts'
  '92E97FA1-2EDF-4476-BDD6-9DD0B4DDDC7B' = 'Block Win32 API calls from Office macros'
  '9E6C4E1F-7D60-472F-BA1A-A39EF669E4B2' = 'Block credential stealing from LSASS'
  'B2B3F03D-6A65-4F7B-A9C7-1C7EF74A9BA4' = 'Block untrusted/unsigned processes from USB'
  '56A863A9-875E-4185-98A7-B882C64B5CE5' = 'Block abuse of exploited vulnerable signed drivers'
  'E6DB77E5-3DF2-4CF1-B95A-636979351E5B' = 'Block persistence through WMI event subscription'
  'C1DB55AB-C21A-4637-BB3F-A12568109D35' = 'Use advanced protection against ransomware'
}
$mode = 2   # 2 = Audit (start here), 1 = Block (after review), 0 = Off
Add-MpPreference -AttackSurfaceReductionRules_Ids @($asr.Keys) -AttackSurfaceReductionRules_Actions (@($mode) * $asr.Count)
Get-MpPreference | Select -Expand AttackSurfaceReductionRules_Ids
```

### Controlled Folder Access (ransomware protection): ask

It protects Documents/Desktop/Pictures from untrusted writers. It **can block dev tools** writing to
protected folders, so keep code outside them (e.g. `C:\dev` or the Dev Drive). Start in audit mode:
```powershell
Set-MpPreference -EnableControlledFolderAccess AuditMode    # later: Enabled
```

## Step 4 — Windows platform protections

| Control | How | Notes |
|---|---|---|
| **Firewall** on for all profiles | `Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled True -DefaultInboundAction Block` | mark unknown Wi-Fi as **Public** |
| **SmartScreen** (apps, Edge, Store) | Windows Security → App & browser control → Reputation-based protection: all On | also turn on "Potentially unwanted app blocking" there |
| **Memory integrity (HVCI)** | Windows Security → Device security → Core isolation → Memory integrity: On (reboot) | if an old driver blocks it, update or remove that driver. Don't force it |
| **LSA protection** | Core isolation → Local Security Authority protection: On (reboot) | protects credentials in memory |
| **Smart App Control** | only available after a clean install, and it can block unsigned dev tools | explain this, and let the user decide. Once off, it can't be turned back on without a reset |
| **Secure Boot + TPM 2.0** | check them (Step 1). If off, *tell the user* to enable them in UEFI | never change firmware from the agent |
| **Disk encryption** | Pro: *Settings → Privacy & security → Device encryption / BitLocker* → turn on. Home: Device encryption | **The user** saves the recovery key (Microsoft account, or printed / in their password manager). The agent never displays or reads it |
| **Windows Update** | automatic, including **Optional → Driver updates** monthly | never disable it |
| **Account security** | Windows Hello PIN/biometrics; MFA on the Microsoft account; consider a separate **standard** user for daily browsing (ask) | UAC stays on (default level or "Always notify") |
| **Encrypted DNS** | Settings → Network → (adapter) → DNS server assignment → Manual → e.g. Quad9 `9.9.9.9` / Cloudflare `1.1.1.1` with **DNS over HTTPS: On (automatic template)** | ask, since corporate networks may need their own DNS |
| **Remote Desktop** | keep **off** unless needed. If on: Network Level Authentication and allowed users only | |
| **SMBv1** | must be disabled: `Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol` → `Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart` if enabled | legacy, a big ransomware vector |

```powershell
Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled True -DefaultInboundAction Block -DefaultOutboundAction Allow
if ((Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol).State -eq 'Enabled') { Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart }
```

## Step 5 — Developer security (secrets, code and supply chain)

### Secrets on a dev machine: no `.env` files with real values
- **Password manager** (Bitwarden / 1Password / KeePassXC from module 13) for personal secrets.
- **Project secrets** come from a secret manager at runtime: `op run -- npm start` (1Password),
  `bws run -- npm start` (Bitwarden Secrets Manager), or `infisical run -- npm start` (Infisical). Commit only
  templates with **references** (e.g. `DATABASE_URL=op://dev/myapp/db-url`), never values.
- **Local service passwords** (dev databases, etc.) go in the **Windows Credential Manager** via
  PowerShell SecretManagement. The user types the value:
  ```powershell
  pwsh -c "Install-Module Microsoft.PowerShell.SecretManagement, Microsoft.PowerShell.SecretStore -Scope CurrentUser -Force"
  pwsh -c "Register-SecretVault -Name LocalStore -ModuleName Microsoft.PowerShell.SecretStore -DefaultVault"
  pwsh -c "Set-Secret -Name pg_password"        # prompts the USER for the value (hidden input)
  ```
  Scripts read it with `Get-Secret pg_password -AsPlainText` only to pass it to the process that needs it.
  **The agent never prints it.**

### Code and supply-chain hygiene
```powershell
Install-App 'Gitleaks.Gitleaks'                       # scan repos/commits for leaked secrets
uv tool install pre-commit                            # run gitleaks + linters on every commit
# Sign commits with your SSH key (shows "Verified" on GitHub):
git config --global gpg.format ssh
git config --global user.signingkey "$HOME\.ssh\id_ed25519.pub"
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```
Then ask the user to add the same public key on GitHub as a **Signing key**
(`gh ssh-key add $HOME\.ssh\id_ed25519.pub --type signing`) and to turn on **push protection** and
**2FA (passkey)** for their GitHub account.
Dependency scanning per project: `npm audit`, `pnpm audit`, `uv tool run pip-audit`, `dotnet list package --vulnerable`,
`govulncheck ./...` (`go install golang.org/x/vuln/cmd/govulncheck@latest`), and `osv-scanner` (`Install-App 'Google.OSVScanner'`).
Container/IaC scanning: `trivy` (module 11).

## Step 6 — Browser security (ask)

- Keep the browser auto-updating. Turn on **Enhanced protection** (Chrome Safe Browsing / Edge
  SmartScreen enhanced / Firefox HTTPS-Only mode).
- Ad/tracker blocking: **uBlock Origin** (Firefox), uBlock Origin Lite (Chromium). Install from the
  official browser store only.
- The password manager's browser extension, instead of the browser's own password store, if the user prefers.

## Step 7 — Backups (ransomware recovery)

- **Windows Backup** app (files, settings, apps → OneDrive) and/or **File History** to an external drive.
- Follow 3-2-1: one copy local, one external, one off-site/cloud. Code lives on GitHub/GitLab too.
- The restore point from module 00 is not a backup.

## Step 8 — Scan and verify

```powershell
Start-MpScan -ScanType QuickScan
Get-MpThreatDetection | Select -First 10        # recent detections (empty is good)
Get-MpComputerStatus | Select RealTimeProtectionEnabled, IsTamperProtected, AntivirusSignatureLastUpdated
# Full scan (long), or an offline scan for suspected rootkits (REBOOTS, so ask first):
# Start-MpScan -ScanType FullScan
# Start-MpWDOScan
```
Optional autorun audit: `Install-App 'Microsoft.Sysinternals.Autoruns'`, then run Autoruns → *Options → Scan
Options → Check VirusTotal.com* and review the startup entries with the user.

Write `$Log\security-after.txt` and add a **Security** section to `REPORT.md`: every control's
before → after state, the items the user still has to do in the UI (Tamper Protection, memory
integrity, BitLocker key backup, GitHub signing key/2FA), and the ASR audit → block reminder date.
