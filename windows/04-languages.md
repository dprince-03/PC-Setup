# 04 — Programming Languages and SDKs (Windows)

**Profiles:** languages (C# and C++ toolchains are also needed by `desktop`) · **Admin:** partly

Languages: **Python, JavaScript/Node.js, Java, Go, C/C++, C#/.NET**, plus optional **Rust** and
**Kotlin**. Follow AGENT.md Rule 4: resolve the latest version at run time. Never pin versions.

> Before each language, run its detection probe. If it's already installed from a *different*
> source than the one below, it's a **CONFLICT**: report it and ask. Don't stack a second copy.

---

## Python

**Detect:** `py list` (new install manager) / `py -0p` (old launcher) / `python --version` / `uv --version`.
If `(Get-Command python).Source` points to `...\WindowsApps\python.exe`, that's the Store alias
stub. Have the user turn it off (module 01 Step 5.6).

**Install (two parts):**
1. **Python Install Manager** (official python.org manager that replaces the old `py` launcher
   and per-version installers):
   ```powershell
   Install-App 'Python.PythonInstallManager'
   Update-SessionPath
   py install --update default     # newest stable CPython, set as default
   py list                         # confirm
   ```
   *Fallback* if the install manager isn't available: `Install-App (Resolve-LatestId 'Python.Python.3')`.
2. **uv** (fast package/venv/project manager, the recommended per-project workflow):
   ```powershell
   Install-App 'astral-sh.uv'
   Install-App 'astral-sh.ruff'     # linter/formatter
   uv --version
   ```
   Global CLI tools go through `uv tool install <tool>` (e.g. `uv tool install pre-commit`), not
   `pip install` into the global Python.

**Verify:** `python --version`, `py list`, `uv python list --only-installed`,
`python -c "import ssl,sqlite3;print('ok')"`

## JavaScript / TypeScript: Node.js

Use a version manager so projects can pin their own Node versions.

**Detect:** `node -v`, `npm -v`, `fnm --version`, `nvm version`. If `nvm` (nvm-windows) is
already in use, keep it and skip fnm (ask the user).

```powershell
Install-App 'Schniz.fnm'
Update-SessionPath
# shell integration for pwsh (idempotent)
$line = 'fnm env --use-on-cd --shell powershell | Out-String | Invoke-Expression'
if (-not (Select-String -Path $PROFILE -SimpleMatch $line -Quiet -EA 0)) { Add-Content $PROFILE $line }
fnm env --use-on-cd --shell powershell | Out-String | Invoke-Expression

fnm install --lts          # newest LTS (default per AGENT.md Rule 4 LTS exception)
fnm default lts-latest
# If the user picked "current" instead: fnm install --latest; fnm default <that version>

corepack enable            # enables pnpm and yarn shims, matching each project's packageManager field
npm install -g npm@latest
```
TypeScript and other globals: prefer per-project `devDependencies`. Global extras only if asked:
`npm i -g typescript tsx npm-check-updates`.

**Verify:** `node -v; npm -v; pnpm -v; npx tsc -v`

## Java (JDK + build tools)

**Detect:** `java -version`, `javac -version`, `$env:JAVA_HOME`, `mvn -v`, `gradle -v`,
`winget list Temurin`, `winget list Microsoft.OpenJDK`, `winget list Oracle.JDK`.

Distribution: **Eclipse Temurin** (Adoptium, free, TCK-tested). LTS releases are 17, 21, 25, then
every 2 years. Rule: a major `N ≥ 17` is LTS when `(N - 17) % 4 == 0`.

```powershell
# Resolve the newest LTS Temurin JDK
$rows = winget search --id EclipseAdoptium.Temurin. --source winget | Out-String
$majors = [regex]::Matches($rows,'EclipseAdoptium\.Temurin\.(\d+)\.JDK') | % { [int]$_.Groups[1].Value } | Sort-Object -Unique
$lts = $majors | Where-Object { $_ -ge 17 -and (($_ - 17) % 4 -eq 0) } | Select-Object -Last 1
Install-App "EclipseAdoptium.Temurin.$lts.JDK"     # the installer sets JAVA_HOME and PATH
# If the user wants the newest feature release too: Install-App "EclipseAdoptium.Temurin.$($majors[-1]).JDK"
Update-SessionPath
```

**Maven and Gradle** aren't in the official winget source. Use **mise** (cross-platform version
manager, also used on Linux):
```powershell
Install-App 'jdx.mise'
Update-SessionPath
mise use -g maven@latest gradle@latest
# add mise shims to PATH for pwsh (idempotent)
$line = '(&mise activate pwsh) | Out-String | Invoke-Expression'
if (-not (Select-String -Path $PROFILE -SimpleMatch $line -Quiet -EA 0)) { Add-Content $PROFILE $line }
```
Fallback: `scoop install maven gradle` (only if the user picked Scoop in module 02). Projects should
prefer their own `./mvnw` and `./gradlew` wrappers.

**Kotlin (optional):** `mise use -g kotlin@latest` (IntelliJ also bundles it).

**Verify:** `java -version; javac -version; echo $env:JAVA_HOME; mvn -v; gradle -v`

## Go

**Detect:** `go version`, `go env GOROOT GOPATH`.
```powershell
Install-App 'GoLang.Go'
Update-SessionPath
go version
go install golang.org/x/tools/gopls@latest        # language server (VS Code installs this too)
go install github.com/go-delve/delve/cmd/dlv@latest
```
Make sure `%USERPROFILE%\go\bin` is on the user PATH (the installer usually adds it).

## C / C++

There are three toolchains on Windows. Install **MSVC** always (it's the native one), plus CMake/Ninja,
and offer LLVM and MinGW.

| Toolchain | How | Notes |
|---|---|---|
| **MSVC** (cl.exe, Windows SDK) | Visual Studio 2026 "Desktop development with C++" workload (module 06) **or**, without the IDE, `Microsoft.VisualStudio.BuildTools` + the `Microsoft.VisualStudio.Workload.VCTools` workload | needed by Rust, Python native wheels, node-gyp |
| **LLVM / Clang** | `LLVM.LLVM` | clang, clang-format, clang-tidy, lldb |
| **GCC (MinGW-w64)** | `BrechtSanders.WinLibs.POSIX.UCRT` *or* MSYS2 (`MSYS2.MSYS2` → `pacman -S mingw-w64-ucrt-x86_64-gcc`) | ask which; don't install both |
| **CMake / Ninja** | `Kitware.CMake`, `Ninja-build.Ninja` | |
| **vcpkg** | `git clone https://github.com/microsoft/vcpkg $HOME\vcpkg; & $HOME\vcpkg\bootstrap-vcpkg.bat` | C++ package manager (also bundled with VS 2026) |

Build Tools only (if VS 2026 isn't wanted):
```powershell
Install-App 'Microsoft.VisualStudio.BuildTools' @('--override','--quiet --wait --norestart --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended')
Install-App 'Kitware.CMake'; Install-App 'Ninja-build.Ninja'; Install-App 'LLVM.LLVM'
```
The unversioned IDs (`Microsoft.VisualStudio.BuildTools`, `Microsoft.VisualStudio.Community`) track
the newest Visual Studio (2026). The `...2022...` IDs pin the old release. Don't use them.

**Verify:** in a "Developer PowerShell for VS": `cl`; anywhere: `clang --version; cmake --version; ninja --version; gcc --version`

## C# / .NET

**Detect:** `dotnet --list-sdks`, `dotnet --list-runtimes`.
```powershell
$dotnet = Resolve-LatestId 'Microsoft.DotNet.SDK'      # newest numbered SDK, e.g. ...SDK.10
Install-App $dotnet
Update-SessionPath
dotnet --list-sdks
dotnet new install Microsoft.DotNet.Web.ProjectTemplates.* 2>$null   # optional: templates come with the SDK
dotnet tool install -g dotnet-ef        # EF Core CLI (commonly needed)
```
Note: .NET even-numbered majors are LTS (8, 10, 12...). If the newest major is an odd/STS release,
tell the user and offer to also install the newest LTS SDK. Several SDKs can sit side by side
safely.

## Rust (optional, ask; also needed by Tauri in module 10)

```powershell
Install-App 'Rustlang.Rustup'      # needs the MSVC build tools above
Update-SessionPath
rustup default stable; rustup update
rustup component add rust-analyzer clippy rustfmt
```

## Verify this module

```powershell
$checks = 'python --version','uv --version','node -v','pnpm -v','java -version','mvn -v','gradle -v',
          'go version','clang --version','cmake --version','dotnet --version','rustc --version'
foreach ($c in $checks) { try { "{0,-18} {1}" -f $c, ((iex "$c 2>&1" | Select -First 1)) } catch { "{0,-18} MISSING" -f $c } }
```
Record the results in `$Log\run.log`.
