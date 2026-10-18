# 06 — IDEs, Editors and API Tools (Windows)

**Profiles:** ides · **Admin:** yes

> JetBrains note (checked 2026): **IntelliJ IDEA** (from 2025.3) and **PyCharm** (from 2025.1) now
> ship as **one unified product** each. The separate "Community" downloads are gone; core features
> are free and the paid features unlock with a subscription. Use the unversioned IDs
> `JetBrains.IntelliJIDEA` and `JetBrains.PyCharm`. Don't use the legacy `...Community` /
> `...Ultimate` / `...Professional` IDs.

---

## Step 1 — Ask the user which they want

| Tool | winget ID | Default |
|---|---|---|
| Visual Studio Code | `Microsoft.VisualStudioCode` | ✔ |
| Visual Studio 2026 Community | `Microsoft.VisualStudio.Community` | ✔ if C#/C++/desktop |
| JetBrains Toolbox (manages every JetBrains IDE and its updates) | `JetBrains.Toolbox` | ✔ (recommended way to get JetBrains IDEs) |
| IntelliJ IDEA (unified) | `JetBrains.IntelliJIDEA` | ✔ if Java/Kotlin |
| PyCharm (unified) | `JetBrains.PyCharm` | ✔ if Python/data |
| Others on request: WebStorm, GoLand, Rider, CLion, DataGrip | `JetBrains.WebStorm`, `JetBrains.GoLand`, `JetBrains.Rider`, `JetBrains.CLion`, `JetBrains.DataGrip` | ask |
| Postman | `Postman.Postman` | ✔ |
| Bruno (offline, Git-friendly API client) | `Bruno.Bruno` | optional |
| Insomnia | `Insomnia.Insomnia` | optional |
| Neovim | `Neovim.Neovim` | optional |
| Zed | `ZedIndustries.Zed` (check with `winget search Zed`) | optional |
| Cursor (AI editor) | `Anysphere.Cursor` (check with `winget search Cursor`) | optional |
| Notepad++ | `Notepad++.Notepad++` | optional |

**JetBrains: Toolbox or direct?** If the user picks Toolbox, install only Toolbox and have them
install the IDEs from it (it keeps them updated). Don't also install the same IDE via winget.

## Step 2 — VS Code + extensions

```powershell
Install-App 'Microsoft.VisualStudioCode' @('--override','/VERYSILENT /MERGETASKS=!runcode,addcontextmenufiles,addcontextmenufolders,associatewithfiles,addtopath')
Update-SessionPath
code --version
```

Install extensions **only for the languages/profiles that were selected** (idempotent:
`code --install-extension` skips ones that are already installed):

```powershell
$ext = @{
  core       = 'eamodio.gitlens','EditorConfig.EditorConfig','usernamehw.errorlens','streetsidesoftware.code-spell-checker','redhat.vscode-yaml','tamasfe.even-better-toml','yzhang.markdown-all-in-one','ms-vscode-remote.remote-wsl','ms-vscode-remote.remote-ssh','ms-vscode-remote.remote-containers'
  python     = 'ms-python.python','ms-python.vscode-pylance','ms-python.debugpy','charliermarsh.ruff','ms-toolsai.jupyter'
  js         = 'dbaeumer.vscode-eslint','esbenp.prettier-vscode','bradlc.vscode-tailwindcss','Prisma.prisma','ms-playwright.playwright','vitest.explorer'
  java       = 'vscjava.vscode-java-pack','vmware.vscode-spring-boot','vscjava.vscode-gradle'
  go         = 'golang.go'
  cpp        = 'ms-vscode.cpptools-extension-pack','llvm-vs-code-extensions.vscode-clangd','ms-vscode.cmake-tools'
  csharp     = 'ms-dotnettools.csdevkit','ms-dotnettools.csharp'
  rust       = 'rust-lang.rust-analyzer','vadimcn.vscode-lldb'
  devops     = 'ms-azuretools.vscode-containers','ms-kubernetes-tools.vscode-kubernetes-tools','hashicorp.terraform','redhat.ansible','github.vscode-github-actions'
  databases  = 'mtxr.sqltools','mongodb.mongodb-vscode','cweijan.vscode-database-client2'
  mobile     = 'Dart-Code.flutter','Dart-Code.dart-code','msjsdiag.vscode-react-native','expo.vscode-expo-tools'
  data       = 'ms-toolsai.jupyter','ms-toolsai.datawrangler','mechatroner.rainbow-csv'
}
# pick the keys for the selected profiles, e.g.:
$want = 'core','python','js','java','go','cpp','csharp','devops','databases'
foreach ($k in $want) { foreach ($e in $ext[$k]) { code --install-extension $e --force | Out-Null } }
code --list-extensions | Out-File "$Log\vscode-extensions.txt"
```
(If an extension ID has been renamed upstream, `code` reports "not found". Search the Marketplace
for the official publisher's replacement and log the change.)

## Step 3 — Visual Studio 2026 with workloads

Pick workloads from the selected profiles:

| Profile / need | Workload ID |
|---|---|
| C# web / ASP.NET | `Microsoft.VisualStudio.Workload.NetWeb` |
| Desktop .NET (WPF/WinForms/WinUI) | `Microsoft.VisualStudio.Workload.ManagedDesktop` |
| C++ desktop | `Microsoft.VisualStudio.Workload.NativeDesktop` |
| .NET MAUI (mobile/desktop) | `Microsoft.VisualStudio.Workload.NetCrossPlat` |
| Azure | `Microsoft.VisualStudio.Workload.Azure` |
| Data / Python (optional) | `Microsoft.VisualStudio.Workload.Data` |
| Game dev with C++ (optional) | `Microsoft.VisualStudio.Workload.NativeGame` |
| Windows app SDK / UWP | `Microsoft.VisualStudio.Workload.Universal` |

```powershell
$wl = 'Microsoft.VisualStudio.Workload.NetWeb','Microsoft.VisualStudio.Workload.ManagedDesktop','Microsoft.VisualStudio.Workload.NativeDesktop'
$add = ($wl | ForEach-Object { "--add $_" }) -join ' '
Install-App 'Microsoft.VisualStudio.Community' @('--override',"--quiet --wait --norestart --includeRecommended $add")
```
If VS is already installed, **modify** it to add workloads instead of reinstalling:
```powershell
& "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vs_installer.exe" modify `
  --installPath (& "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe" -latest -property installationPath) `
  $($wl | % { '--add', $_ }) --includeRecommended --quiet --norestart
```
Paid editions (Professional/Enterprise) only if the user has a licence: `Microsoft.VisualStudio.Professional` / `.Enterprise`.

## Step 4 — JetBrains

```powershell
# Option 1 (recommended): Toolbox
Install-App 'JetBrains.Toolbox'
# Option 2: direct
Install-App 'JetBrains.IntelliJIDEA'
Install-App 'JetBrains.PyCharm'
```
Tell the user: open each IDE once, sign in with their JetBrains account for paid features (free
for students via GitHub Education), and point the IDE at the JDK and Python interpreters
installed in module 04.

## Step 5 — API clients

```powershell
Install-App 'Postman.Postman'
# optional: Install-App 'Bruno.Bruno'; Install-App 'Insomnia.Insomnia'
```

## Verify

```powershell
code --version
& "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe" -latest -format value -property catalog_productDisplayVersion
winget list --id JetBrains -e 2>$null; winget list Postman
```
