# 02 — Foundation: Terminal, Shell, Git, CLI Tools (Windows)

**Profiles:** core · **Admin:** yes · Load the helpers from `00-preflight.md` Step 6 first.

---

## Step 1 — Core packages

| Tool | winget ID | Why |
|---|---|---|
| PowerShell 7 | `Microsoft.PowerShell` | modern shell (`pwsh`), used from here on |
| Windows Terminal | `Microsoft.WindowsTerminal` | usually preinstalled on Win 11; check it |
| Git for Windows | `Git.Git` | includes Git Bash, OpenSSH, Git Credential Manager |
| Git LFS | `GitHub.GitLFS` | large files in repos |
| GitHub CLI | `GitHub.cli` | `gh` for PRs, auth, repos |
| GitHub Desktop (optional, ask) | `GitHub.GitHubDesktop` | GUI Git |
| 7-Zip | `7zip.7zip` | archives |
| curl / wget | built in (`curl.exe`), `JernejSimoncic.Wget` (optional) | |
| jq / yq | `jqlang.jq`, `MikeFarah.yq` | JSON/YAML on the CLI |
| ripgrep / fd / fzf / bat | `BurntSushi.ripgrep.MSVC`, `sharkdp.fd`, `junegunn.fzf`, `sharkdp.bat` | fast search tools |
| eza / zoxide | `eza-community.eza`, `ajeetdsouza.zoxide` | better `ls` / `cd` |
| Starship prompt | `Starship.Starship` | cross-shell prompt (or Oh My Posh: `JanDeDobbeleer.OhMyPosh`, ask) |
| gsudo (only if built-in sudo isn't available) | `gerardog.gsudo` | sudo for Windows 10 / older 11 |
| Nerd Font | via Oh My Posh or `DEVCOM.JetBrainsMonoNerdFont` | icons in the terminal |
| Neovim (optional) | `Neovim.Neovim` | terminal editor |
| make | `ezwinports.make` | GNU make for Makefiles |

```powershell
$core = 'Microsoft.PowerShell','Microsoft.WindowsTerminal','Git.Git','GitHub.GitLFS','GitHub.cli',
        '7zip.7zip','jqlang.jq','MikeFarah.yq','BurntSushi.ripgrep.MSVC','sharkdp.fd','junegunn.fzf',
        'sharkdp.bat','eza-community.eza','ajeetdsouza.zoxide','Starship.Starship',
        'DEVCOM.JetBrainsMonoNerdFont','ezwinports.make'
foreach ($id in $core) { Install-App $id }
Update-SessionPath
```
If an ID doesn't resolve, run `winget search <name>` and pick the entry from the official publisher.

## Step 2 — Git configuration (ask the user for name and email; never guess)

```powershell
git config --global user.name  "<Full Name>"
git config --global user.email "<email>"
git config --global init.defaultBranch main
git config --global core.autocrlf true          # Windows checkout CRLF, commit LF
git config --global core.longpaths true
git config --global pull.rebase false
git config --global fetch.prune true
git config --global credential.helper manager    # Git Credential Manager
git lfs install
```
Only set name/email if they aren't already set (`git config --global user.email` returns empty).

## Step 3 — SSH key for GitHub/GitLab (skip if `~/.ssh/id_ed25519` exists)

```powershell
if (-not (Test-Path "$HOME\.ssh\id_ed25519")) {
  ssh-keygen -t ed25519 -C "<email>" -f "$HOME\.ssh\id_ed25519"   # user types a passphrase
}
Get-Service ssh-agent | Set-Service -StartupType Automatic; Start-Service ssh-agent
ssh-add "$HOME\.ssh\id_ed25519"
```
Then tell the user to run `gh auth login` (pick SSH, upload the key) themselves. **Never print
the private key.**

## Step 4 — PowerShell profile (idempotent)

```powershell
pwsh -NoProfile -Command {
  if (-not (Test-Path $PROFILE)) { New-Item -ItemType File -Force -Path $PROFILE | Out-Null }
  $lines = @(
    'Invoke-Expression (&starship init powershell)',
    'Invoke-Expression (& { (zoxide init powershell | Out-String) })',
    'Set-PSReadLineOption -PredictionSource HistoryAndPlugin -PredictionViewStyle ListView',
    'Set-Alias ll eza'
  )
  foreach ($l in $lines) { if (-not (Select-String -Path $PROFILE -SimpleMatch $l -Quiet)) { Add-Content $PROFILE $l } }
}
```
Install the PSReadLine and Terminal-Icons modules for pwsh:
`pwsh -c "Install-Module PSReadLine,Terminal-Icons -Scope CurrentUser -Force -AllowClobber"`

## Step 5 — Windows Terminal defaults

Ask before changing them. Recommended: default profile → **PowerShell 7**, font →
**JetBrainsMono Nerd Font**. Edit `settings.json` under
`%LOCALAPPDATA%\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\` by merging keys,
never by overwriting the file.

## Step 6 — Optional: Scoop (user-level package manager)

Only if the user wants it. Some CLI dev tools update faster on Scoop. Don't run it elevated:

```powershell
# run in a NON-admin pwsh
Invoke-RestMethod -Uri https://get.scoop.sh -OutFile "$env:TEMP\install-scoop.ps1"; & "$env:TEMP\install-scoop.ps1"
scoop bucket add extras
```
Rule: one tool, one source. If a tool is installed via winget, don't install it again via scoop.

## Verify

```powershell
pwsh -v; git --version; git lfs version; gh --version; rg --version; fzf --version; starship --version
```
