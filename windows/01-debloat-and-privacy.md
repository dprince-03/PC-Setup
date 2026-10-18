# 01 — Debloat, Privacy and Developer Tweaks (Windows)

**Profiles:** debloat (menu item; **always ask** before running this module) · **Admin:** yes ·
**Reboot:** recommended at the end

Tools used (official upstream only):
- **Win11Debloat** by Raphire: <https://github.com/Raphire/Win11Debloat> (main tool)
- **WinUtil** by Chris Titus Tech: <https://github.com/ChrisTitusTech/winutil> (optional GUI,
  only if the user asks for it)

> ⚠️ Both are PowerShell scripts that change system settings. Per AGENT.md Rule 6:
> show the user the exact list of changes, get a yes, and make sure the restore point from
> module 00 exists. **Never** use any option that disables Defender, SmartScreen, UAC, the
> firewall, Windows Update, or BitLocker.

---

## Step 1 — Check the restore point exists

```powershell
Get-ComputerRestorePoint | Where-Object Description -like 'PC-Setup*' | Select-Object -Last 1
```
If there isn't one, go back to 00-preflight Step 4.

## Step 2 — Show the user the plan and ask

Present this default "developer-safe" set and let the user add or remove items:

| Change | Win11Debloat switch | Default |
|---|---|---|
| Remove preinstalled bloat apps (Candy Crush, Clipchamp, News, etc.) | `-RemoveApps` | ✔ |
| Disable telemetry, diagnostic data, targeted ads | `-DisableTelemetry` | ✔ |
| Disable tips/suggestions/ads in Start, Settings, Explorer | `-DisableSuggestions` | ✔ |
| Disable lock-screen tips | `-DisableLockscreenTips` | ✔ |
| Disable Bing/web results in Start search | `-DisableBing` | ✔ |
| Disable Edge ads and MSN feed | `-DisableEdgeAds` | ✔ |
| Disable Microsoft 365 ads in Settings | `-DisableSettings365Ads` | ✔ |
| Stop Windows Update silently adding driver companion apps | `-DisableDeviceAutoAppDownload` | ✔ |
| Disable Widgets | `-DisableWidgets` | ✔ |
| Hide Chat icon | `-HideChat` | ✔ |
| Show file extensions | `-ShowKnownFileExt` | ✔ (devs need this) |
| Show hidden files | `-ShowHiddenFolders` | ✔ |
| Add "End Task" to taskbar right-click | `-EnableEndTask` | ✔ |
| Explorer opens to This PC | `-ExplorerToThisPC` | ✔ |
| Disable Recall | `-DisableRecall` | ✔ |
| Disable Copilot | `-DisableCopilot` | ask |
| Disable Click to Do | `-DisableClickToDo` | ask |
| Disable AI in Edge / Paint / Notepad | `-DisableEdgeAI -DisablePaintAI -DisableNotepadAI` | ask |
| Remove Xbox app + Game Bar | `-RemoveGamingApps -DisableDVR` | ask (gamers say no) |
| Remove OEM (HP) apps | `-RemoveHPApps` | ask, HP machines only |
| Classic (Win10) right-click menu | `-RevertContextMenu` | ask |
| Align taskbar left | `-TaskbarAlignLeft` | ask |
| Dark mode | `-EnableDarkMode` | ask |
| Disable Fast Startup (helps dual-boot, WSL and USB issues) | `-DisableFastStartup` | ask (yes if dual-booting) |

**Never pass:** `-ForceRemoveEdge` (breaks WebView2-based apps and dev tools),
`-DisableBitlockerAutoEncryption` (weakens security; only if the user explicitly insists),
`-Sysprep`, `-User` (not needed here).

**Keep these apps:** do NOT remove Windows Terminal, Microsoft Store, App Installer (winget),
Windows Calculator, Snipping Tool, Photos, Notepad, Paint, Your Phone/Phone Link (Android
devs may use it), or the Xbox Identity Provider if the user games.

## Step 3 — Download the script, look at it, then run it

Following Rule 6, don't pipe it straight into `iex` blind. Download it, glance at it, then run it:

```powershell
$dl = "$env:TEMP\Win11Debloat-Get.ps1"
Invoke-RestMethod "https://debloat.raphi.re/" -OutFile $dl
Get-Content $dl -TotalCount 40          # sanity check: it should fetch from github.com/Raphire/Win11Debloat
Get-AuthenticodeSignature $dl | Select-Object Status   # informational; the script is unsigned upstream
```

Preview what it would do (the script supports `-WhatIf` from release 2026.07.11 onward):

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force   # this session only
& $dl -WhatIf -RemoveApps -DisableTelemetry -DisableSuggestions
```

Then run it with the switches the user approved, for example:

```powershell
& $dl -Silent -CreateRestorePoint -LogPath "$Log" `
  -RemoveApps -DisableTelemetry -DisableSuggestions -DisableLockscreenTips -DisableBing `
  -DisableEdgeAds -DisableSettings365Ads -DisableDeviceAutoAppDownload -DisableWidgets -HideChat `
  -ShowKnownFileExt -ShowHiddenFolders -EnableEndTask -ExplorerToThisPC -DisableRecall
```

If `debloat.raphi.re` can't be reached, download the latest release ZIP from
`https://github.com/Raphire/Win11Debloat/releases/latest`, extract it, and run
`Win11Debloat.ps1` with the same switches (after `Set-ExecutionPolicy Bypass -Scope Process -Force`).

## Step 4 — (Optional) WinUtil GUI

Only if the user asks for it. It's a GUI for tweaks, installs and Windows Update policy.

```powershell
irm "https://christitus.com/win" | iex
# fallback if the domain is down:
# irm "https://github.com/ChrisTitusTech/winutil/releases/latest/download/winutil.ps1" | iex
```
Tell the user: **use only the "Standard" tweak preset**. Don't use the "Updates → Disable"
option, the "Remove Edge" option, or anything that disables Defender.

## Step 5 — Developer-oriented Windows settings (always safe; ask once as a batch)

```powershell
# 1. Developer Mode: symlinks without admin, sideloading, dev settings
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\AppModelUnlock" /t REG_DWORD /f /v AllowDevelopmentWithoutDevLicense /d 1

# 2. Long paths (node_modules, Java, Android builds)
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem' -Name LongPathsEnabled -Value 1 -PropertyType DWord -Force

# 3. Git long paths too (after Git is installed in module 02):  git config --system core.longpaths true

# 4. Power plan: High performance on desktops; leave Balanced on laptops on battery
#    powercfg /list ; powercfg /setactive SCHEME_MIN   # only on desktops / when plugged in, ask

# 5. PowerShell script execution for the current user (needed for profiles, fnm, oh-my-posh)
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned -Force

# 6. Turn off the Microsoft Store "python.exe" aliases (they hijack `python`)
#    Settings > Apps > Advanced app settings > App execution aliases > turn off python.exe and python3.exe
#    Ask the user to do this in Settings: it's a per-user UI toggle with no supported CLI.

# 7. Enable the built-in `sudo` for Windows (Windows 11 24H2+), inline mode
#    Settings > System > For developers > Enable sudo, or:
sudo config --enable normal 2>$null
```

### Dev Drive (recommended for repos and package caches, Windows 11 22H2+, needs ≥50 GB free)

Ask the user. If yes: Settings → System → Storage → Advanced storage settings → Disks & volumes →
**Create Dev Drive**. Use a VHDX if there's no spare partition. Then point caches at it:

```powershell
# Example if the Dev Drive is D:\
setx npm_config_cache D:\packages\npm
setx PIP_CACHE_DIR  D:\packages\pip
setx UV_CACHE_DIR   D:\packages\uv
setx NUGET_PACKAGES D:\packages\nuget
setx GRADLE_USER_HOME D:\packages\gradle
setx MAVEN_OPTS "-Dmaven.repo.local=D:\packages\maven"
setx CARGO_HOME D:\packages\cargo
```
Also: keep Defender on, but make sure the Dev Drive uses **performance mode**
(`fsutil devdrv query D:`). Don't add Defender exclusions.

## Step 6 — Windows Update and drivers

```powershell
# Make sure the OS is current before installing toolchains
Start-Process "ms-settings:windowsupdate"
```
Ask the user to install pending updates (including **Optional → Driver updates**, then reboot if
asked). On laptops, also suggest the OEM support tool (e.g. HP Support Assistant, Lenovo Vantage,
Dell Command Update) for BIOS and firmware. Install the OEM tool only if the user wants it.

## Verify

- File extensions are visible in Explorer; Start shows no ads.
- `Get-ItemPropertyValue 'HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem' LongPathsEnabled` → `1`
- Log any changes to `$Log\run.log`. Tell the user a **reboot** finishes the Explorer/taskbar
  changes. It can wait until the end of module 03.
