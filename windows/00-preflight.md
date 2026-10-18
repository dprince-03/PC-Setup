# 00 — Preflight (Windows)

**Profiles:** all · **Admin:** yes · **Goal:** confirm the OS, confirm admin rights, make sure
winget works, take an inventory of what's already installed, create a restore point, and load
the helper functions.

---

## Step 1 — Confirm this is Windows, and which version

```powershell
$os = Get-CimInstance Win32_OperatingSystem
[pscustomobject]@{
  Caption      = $os.Caption
  Version      = $os.Version
  Build        = [int]$os.BuildNumber
  DisplayVer   = (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion').DisplayVersion
  Arch         = $env:PROCESSOR_ARCHITECTURE      # AMD64 or ARM64
  Edition      = (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion').EditionID
  PSVersion    = $PSVersionTable.PSVersion.ToString()
  RAM_GB       = [math]::Round($os.TotalVisibleMemorySize/1MB,1)
  FreeC_GB     = [math]::Round((Get-PSDrive C).Free/1GB,1)
}
```

Decide:
- `Build -ge 22000` → Windows 11 ✔. Continue.
- `Build -lt 22000` → Windows 10. Warn that it's past end of support, then ask whether to continue.
- `Arch = ARM64` → note it. Some tools (older Android emulator images, some CUDA tooling) aren't
  available. Each module says when that applies.
- `FreeC_GB -lt 60` → warn. A full install across all profiles needs roughly 60–120 GB.
- `Edition` contains `Home` → Hyper-V and Windows Sandbox aren't available. WSL2 and Docker
  Desktop still work.

## Step 2 — Confirm admin rights

```powershell
$IsAdmin = ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()
           ).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
$IsAdmin
```
If `False`: stop and ask the user to reopen the terminal with **Run as administrator**.
Never try to get around UAC.

## Step 3 — Make sure winget works

```powershell
winget --version
winget source update
```
- If `winget` isn't recognised: ask the user to update **App Installer** from the Microsoft Store
  (`ms-windows-store://pdp/?ProductId=9NBLGGH4NNS1`), then try again.
- Accept the source agreements once: `winget list --accept-source-agreements | Out-Null`

## Step 4 — Create the log folder and a restore point

```powershell
$Log = Join-Path $env:USERPROFILE ("pc-setup-logs\" + (Get-Date -Format 'yyyyMMdd-HHmmss'))
New-Item -ItemType Directory -Force -Path $Log | Out-Null
"$(Get-Date -Format s) [00] OS: $($os.Caption) $($os.Version) build $($os.BuildNumber) $env:PROCESSOR_ARCHITECTURE" |
  Tee-Object -FilePath "$Log\os.txt"

# Restore point (needs System Protection on C:)
Enable-ComputerRestore -Drive "C:\" -ErrorAction SilentlyContinue
# Windows allows one restore point per 24 h by default. This lifts that for this run:
New-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SystemRestore' `
  -Name SystemRestorePointCreationFrequency -Value 0 -PropertyType DWord -Force | Out-Null
Checkpoint-Computer -Description "PC-Setup before changes" -RestorePointType MODIFY_SETTINGS
Get-ComputerRestorePoint | Select-Object -Last 1
```
If the restore point fails, tell the user and ask whether to continue anyway.

## Step 5 — Inventory: what's already installed

```powershell
# 1. Everything winget knows about (includes most MSI/EXE installs, not only winget ones)
winget export -o "$Log\inventory-before.json" --include-versions --accept-source-agreements | Out-Null
winget list --accept-source-agreements | Out-File "$Log\inventory-before-winget.txt"
winget upgrade --include-unknown        | Out-File "$Log\upgrades-available.txt"

# 2. Command probes: catches tools installed outside winget (zip, scoop, choco, fnm...)
$probes = [ordered]@{
  git='git --version'; gh='gh --version'; pwsh='pwsh -v'; python='python --version'; py='py --version';
  uv='uv --version'; node='node -v'; npm='npm -v'; pnpm='pnpm -v'; bun='bun -v'; deno='deno -v';
  java='java -version'; mvn='mvn -v'; gradle='gradle -v'; go='go version'; gcc='gcc --version';
  clang='clang --version'; cl='cl'; cmake='cmake --version'; dotnet='dotnet --version';
  rustc='rustc --version'; docker='docker --version'; kubectl='kubectl version --client';
  helm='helm version --short'; terraform='terraform -version'; tofu='tofu -version';
  aws='aws --version'; az='az version'; gcloud='gcloud --version'; psql='psql --version';
  mysql='mysql --version'; mongod='mongod --version'; mongosh='mongosh --version';
  'redis-cli'='redis-cli --version'; code='code --version'; wsl='wsl --version';
  flutter='flutter --version'; adb='adb version'; conda='conda --version'; nvidia='nvidia-smi'
}
$inv = foreach ($k in $probes.Keys) {
  $cmd = ($probes[$k] -split ' ')[0]
  $path = (Get-Command $cmd -ErrorAction SilentlyContinue).Source
  $ver = if ($path) { (Invoke-Expression "$($probes[$k]) 2>&1" | Select-Object -First 1) -as [string] } else { '' }
  [pscustomobject]@{ Tool=$k; Present=[bool]$path; Version=$ver.Trim(); Path=$path }
}
$inv | Format-Table -AutoSize | Out-String -Width 250 | Tee-Object "$Log\inventory-before-probes.txt"

# 3. Other package managers that may already be in use
foreach ($pm in 'scoop','choco') { if (Get-Command $pm -EA 0) { "$pm present" } }

# 4. Hardware the later modules care about
Get-CimInstance Win32_VideoController | Select-Object Name, DriverVersion
(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled   # needed for WSL2/Docker/emulators
```

**Conflict detection** (report these, don't fix them yet):
- `node` found but not under `fnm`, *and* the user wants fnm → two Nodes on PATH.
- `python` resolves to `...\WindowsApps\python.exe` → that's the Microsoft Store alias stub,
  not a real Python. Module 04 deals with it.
- Both `scoop` and `winget` versions of the same tool.
- Chocolatey present → keep using winget for this setup. Don't uninstall choco without asking.

## Step 6 — Load the helper functions (paste into the session)

```powershell
function Update-SessionPath {
  $env:Path = [Environment]::GetEnvironmentVariable('Path','Machine') + ';' +
              [Environment]::GetEnvironmentVariable('Path','User')
}

function Test-App([string]$Id) {
  $out = winget list --id $Id -e --accept-source-agreements 2>$null | Out-String
  if ($out -notmatch [regex]::Escape($Id)) { return 'MISSING' }
  $up = winget upgrade --id $Id -e --include-unknown 2>$null | Out-String
  if ($up -match [regex]::Escape($Id)) { 'OUTDATED' } else { 'CURRENT' }
}

function Install-App([string]$Id, [string[]]$Extra = @()) {
  $state = Test-App $Id
  switch ($state) {
    'CURRENT'  { "[$Id] already current - skip" }
    'OUTDATED' { winget upgrade --id $Id -e --source winget --silent --accept-package-agreements --accept-source-agreements @Extra }
    'MISSING'  { winget install --id $Id -e --source winget --silent --accept-package-agreements --accept-source-agreements @Extra }
  }
  "$(Get-Date -Format s) [$Id] $state -> done (exit $LASTEXITCODE)" | Add-Content "$Log\run.log"
  Update-SessionPath
}

# For IDs with the major version in the name, e.g. Resolve-LatestId 'Python.Python.3'
# Skips previews/betas. Returns e.g. 'Python.Python.3.14'
function Resolve-LatestId([string]$Prefix, [string]$Suffix = '') {
  $rows = winget search --id "$Prefix." --source winget --accept-source-agreements | Out-String
  $pat  = [regex]::Escape($Prefix) + '\.(\d+)' + [regex]::Escape($Suffix) + '(?=\s)'
  ([regex]::Matches($rows, $pat) | ForEach-Object { [int]$_.Groups[1].Value } |
     Sort-Object -Unique | Select-Object -Last 1) | ForEach-Object { "$Prefix.$_$Suffix" }
}
```

> `winget install` installs the **newest** version in the source by default. That covers the
> "latest only" rule. Never pass `--version` unless the user asks for a specific version.

## Step 7 — Show the user the plan and get approval

Combine the inventory with the selected profiles into `$Log\plan.md`: a table of
*item | current state | planned action*. Show it. **Wait for an explicit yes.**
