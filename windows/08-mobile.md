# 08 — Mobile Development (Windows)

**Profiles:** mobile · **Admin:** yes · Depends on: 04 (Java, Node), 06 (VS Code)

> iOS builds need macOS/Xcode. On Windows you can build Android apps, plus Flutter/React Native
> code that targets iOS later (via a Mac, cloud builds like EAS Build or Codemagic, or Expo Go on a
> real iPhone).

---

## Step 1 — Hardware acceleration for emulators

```powershell
(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled      # must be True
Get-WindowsOptionalFeature -Online -FeatureName HypervisorPlatform | Select State
```
The Android Emulator on Windows uses the **Windows Hypervisor Platform (WHPX)**. If it isn't enabled (ask; reboot needed):
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName HypervisorPlatform -All -NoRestart
```
On ARM64 Windows, only ARM64 system images work in the emulator. Mention this.

## Step 2 — Android Studio + SDK

```powershell
Install-App 'Google.AndroidStudio'      # latest stable (the Beta/Canary IDs are separate; don't use them)
```
Then have the user **open Android Studio once** and finish the Setup Wizard (Standard). It
downloads the latest SDK Platform, Build-Tools, Platform-Tools, Emulator, and a system image.

Set the environment variables (idempotent, user scope):
```powershell
$sdk = "$env:LOCALAPPDATA\Android\Sdk"
[Environment]::SetEnvironmentVariable('ANDROID_HOME', $sdk, 'User')
$p = [Environment]::GetEnvironmentVariable('Path','User')
foreach ($d in "$sdk\platform-tools","$sdk\emulator","$sdk\cmdline-tools\latest\bin") {
  if ($p -notlike "*$d*") { $p += ";$d" } }
[Environment]::SetEnvironmentVariable('Path', $p, 'User'); Update-SessionPath
```
If `cmdline-tools\latest` is missing: Android Studio → Settings → Android SDK → SDK Tools →
**Android SDK Command-line Tools (latest)**. Then accept the licences:
```powershell
sdkmanager --licenses       # user types y
sdkmanager --list_installed
```
Android Studio bundles its own JBR (JDK). Gradle builds outside the IDE use `JAVA_HOME` from module 04.

## Step 3 — Flutter (ask)

Flutter isn't in winget's official source. Use the official SDK from Git (stable channel = latest stable):
```powershell
$dir = "$env:USERPROFILE\dev\flutter"
if (-not (Test-Path $dir)) { git clone https://github.com/flutter/flutter.git -b stable $dir } else { git -C $dir pull }
$p = [Environment]::GetEnvironmentVariable('Path','User'); if ($p -notlike "*$dir\bin*") {
  [Environment]::SetEnvironmentVariable('Path', "$p;$dir\bin", 'User') }
Update-SessionPath
flutter --version
flutter config --android-sdk "$env:LOCALAPPDATA\Android\Sdk"
flutter doctor --android-licenses
flutter doctor -v
```
Fix every ✘ that `flutter doctor` reports for the selected targets. For Flutter Windows-desktop
targets you need VS 2026 with the "Desktop development with C++" workload (module 06).
Alternatively the user can install Flutter from VS Code (Dart-Code extension →
"Flutter: New Project" → Download SDK). Pick one way.

## Step 4 — React Native / Expo (ask)

```powershell
# Node LTS and JDK are already installed from module 04
npm i -g eas-cli            # Expo Application Services CLI (cloud builds)
# Scaffold per project; don't install globally:
# npx create-expo-app@latest myapp
# npx @react-native-community/cli@latest init MyApp
```
Make sure `ANDROID_HOME` and `JAVA_HOME` are set (above). React Native targets the JDK version
its docs recommend. If that differs from the installed JDK, add that LTS too with
`Install-App "EclipseAdoptium.Temurin.<N>.JDK"` and tell the user.

## Step 5 — .NET MAUI (optional, C# mobile)

VS 2026 workload `Microsoft.VisualStudio.Workload.NetCrossPlat` (module 06), or with the CLI:
`dotnet workload install maui`.

## Step 6 — Device tools

| Tool | Install |
|---|---|
| scrcpy (mirror/control an Android device) | `Install-App 'Genymobile.scrcpy'` |
| Google USB driver (some phones) | SDK Manager → SDK Tools → Google USB Driver |

## Verify

```powershell
adb version; emulator -list-avds; sdkmanager --version
flutter doctor; npx expo --version 2>$null; eas --version
```
