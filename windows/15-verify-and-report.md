# 15 — Verify Everything and Write the Report (Windows)

**Profiles:** all · Run in a **new** PowerShell 7 window so PATH changes are picked up.

---

## Step 1 — After-install inventory

```powershell
$Log = (Get-ChildItem "$env:USERPROFILE\pc-setup-logs" -Directory | Sort-Object Name | Select-Object -Last 1).FullName
winget export -o "$Log\inventory-after.json" --include-versions --accept-source-agreements | Out-Null
winget upgrade --include-unknown | Out-File "$Log\upgrades-remaining.txt"
```

## Step 2 — Smoke tests (only for the selected profiles)

```powershell
$tests = [ordered]@{
  'git'        = { git --version }
  'gh'         = { gh --version | Select -First 1 }
  'pwsh'       = { pwsh -v }
  'python'     = { python --version }
  'uv'         = { uv --version }
  'node'       = { node -v }
  'pnpm'       = { pnpm -v }
  'java'       = { (java -version 2>&1)[0] }
  'maven'      = { (mvn -v)[0] }
  'gradle'     = { (gradle -v | Select-String 'Gradle ').ToString() }
  'go'         = { go version }
  'dotnet'     = { dotnet --version }
  'cmake'      = { cmake --version | Select -First 1 }
  'clang'      = { clang --version | Select -First 1 }
  'rust'       = { rustc --version }
  'docker'     = { docker run --rm hello-world | Select-String 'Hello from Docker' }
  'kubectl'    = { kubectl version --client | Select -First 1 }
  'helm'       = { helm version --short }
  'terraform'  = { terraform -version | Select -First 1 }
  'aws'        = { aws --version }
  'az'         = { (az version | ConvertFrom-Json).'azure-cli' }
  'gcloud'     = { gcloud --version | Select -First 1 }
  'wsl'        = { (wsl --version)[0] }
  'defender'   = { $m = Get-MpComputerStatus; "realtime=$($m.RealTimeProtectionEnabled) tamper=$($m.IsTamperProtected) sigs=$($m.AntivirusSignatureLastUpdated)" }
  'firewall'   = { (Get-NetFirewallProfile | ForEach-Object { "$($_.Name)=$($_.Enabled)" }) -join ' ' }
  'bitlocker'  = { (Get-BitLockerVolume -MountPoint C:).ProtectionStatus }   # status only, never the key
  'gitleaks'   = { gitleaks version }
  'code'       = { code --version | Select -First 1 }
  'adb'        = { adb version | Select -First 1 }
  'flutter'    = { flutter --version | Select -First 1 }
  'torch'      = { uv run --directory "$env:USERPROFILE\dev\ml-sandbox" python -c "import torch;print(torch.__version__, 'cuda' if torch.cuda.is_available() else 'cpu')" }
}
$results = foreach ($k in $tests.Keys) {
  try { $v = & $tests[$k] 2>&1 | Out-String; [pscustomobject]@{Item=$k; OK=($LASTEXITCODE -eq 0 -or $v); Detail=$v.Trim()} }
  catch { [pscustomobject]@{Item=$k; OK=$false; Detail=$_.Exception.Message} }
}
$results | Format-Table -AutoSize -Wrap | Out-String -Width 220 | Tee-Object "$Log\smoke-tests.txt"
```
Drop the rows for profiles that weren't selected. An empty result for an *unselected* tool is fine.

## Step 3 — Write `REPORT.md`

Write `$Log\REPORT.md` using the AGENT.md Rule 10 structure:

```markdown
# PC-Setup report: <hostname>, <date>
- OS: Windows 11 <DisplayVersion> build <N> (<arch>)
- Playbook: windows/  · Profiles: core, languages, ...
- Restore point: "PC-Setup before changes" (<timestamp>)

| Item | Before | Action | After | Verified |
|---|---|---|---|---|
| Git | 2.x.y | upgraded | 2.z | ✔ |
...

## Failed / skipped
## Manual follow-ups
- [ ] Reboot (pending: WSL / Docker / Hyper-V)
- [ ] `gh auth login`, then add the SSH key to GitHub
- [ ] Open Docker Desktop, accept the licence, enable WSL integration
- [ ] Open Android Studio once (SDK wizard), run `flutter doctor`
- [ ] Sign in to JetBrains IDEs / Postman / cloud CLIs
- [ ] Turn off the python.exe App execution aliases (if still on)
## Keep it updated
winget upgrade --all --include-unknown · fnm install --lts · uv self update · rustup update · mise upgrade · wsl --update
```

## Step 4 — Hand-off message

Post a short summary in chat: profiles done, number of items installed/upgraded/skipped/failed,
pending reboot status, and the path to `REPORT.md`.
