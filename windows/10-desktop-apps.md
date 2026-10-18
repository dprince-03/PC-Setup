# 10 — Desktop Application Development (Windows)

**Profiles:** desktop · **Admin:** yes · Depends on: 04 (.NET, C++, Rust, Node), 06 (Visual Studio)

Ask which stacks the user wants. Install only those.

| Stack | Language | What to install |
|---|---|---|
| **WPF / WinForms** | C# | VS 2026 workload `ManagedDesktop` (module 06) |
| **WinUI 3 / Windows App SDK** | C#/C++ | VS workload `ManagedDesktop` + component `Microsoft.VisualStudio.ComponentGroup.WindowsAppSDK.Cs` (C#) or `Universal`/`NativeDesktop` with WinAppSDK C++ templates |
| **.NET MAUI** | C# | `dotnet workload install maui` or VS workload `NetCrossPlat` |
| **Avalonia** (cross-platform XAML) | C# | `dotnet new install Avalonia.Templates` |
| **Electron** | JS/TS | Node LTS (04). Per project: `npm create electron-app@latest` (Electron Forge) |
| **Tauri** | Rust + web | Rust (04) + MSVC Build Tools + **WebView2** (built into Win 11). `npm create tauri-app@latest` |
| **Qt** | C++/Python | Qt Online Installer (`winget search Qt` → official "Qt Group"), or `uv add pyside6` for Python |
| **Flutter desktop** | Dart | Flutter (module 08) + VS C++ workload; `flutter config --enable-windows-desktop` |
| **Java desktop (JavaFX/Swing)** | Java | JDK (04); JavaFX via Maven/Gradle (`org.openjfx`) per project |
| **Win32/C++** | C++ | VS workload `NativeDesktop` |

---

## Commands

```powershell
# .NET desktop templates and workloads
dotnet workload update
dotnet workload install maui           # only if MAUI was selected
dotnet new install Avalonia.Templates  # only if Avalonia was selected

# Tauri prerequisites check
rustc --version; cargo --version
Get-AppxPackage -Name *WebView2* -EA 0; reg query "HKLM\SOFTWARE\WOW6432Node\Microsoft\EdgeUpdate\Clients\{F3017226-FE2A-4295-8BDF-00C3A9A7E4C5}" /v pv 2>$null
cargo install create-tauri-app --locked      # optional; or use npm/pnpm create tauri-app

# Packaging tools
Install-App 'Microsoft.WindowsSDK'    2>$null   # check with winget search WindowsSDK; usually comes with VS
dotnet tool install -g wix            # WiX v5+ MSI authoring (optional)
```

Code signing (later, for distribution): explain the options (Azure Trusted Signing, an OV/EV
certificate) and install nothing for it now.

## Verify

```powershell
dotnet workload list
dotnet new list | Select-String -Pattern 'wpf|winforms|maui|avalonia'
rustc --version; node -v
flutter doctor 2>$null | Select-String 'Visual Studio'
```
