# 06 — Mobile Development (Ubuntu)

**Profiles:** mobile · Desktop only (the emulator needs KVM; on WSL, use the Windows-side Android
Studio instead).

---

## Step 1 — KVM acceleration

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo     # > 0 means the CPU supports virtualization
apt_install qemu-kvm cpu-checker
kvm-ok
sudo usermod -aG kvm "$USER"           # takes effect after the next login
```

## Step 2 — Android Studio (latest stable)

Preferred: `snap_install android-studio --classic` (official Google-published snap, auto-updates).
Alternatively, the official tarball:
```bash
# Get the newest stable Linux tarball URL from https://developer.android.com/studio (or JetBrains Toolbox)
# then:  sudo tar -xzf android-studio-*-linux.tar.gz -C /opt && /opt/android-studio/bin/studio.sh
apt_install libc6 libncurses6 libstdc++6 zlib1g libbz2-1.0 2>/dev/null   # runtime libs (most are already present)
```
The user opens Android Studio once and runs the Setup Wizard (Standard), which downloads the
latest SDK, platform-tools, emulator and system image.

```bash
add_rc() { grep -qxF "$1" ~/.bashrc || echo "$1" >> ~/.bashrc; }
add_rc 'export ANDROID_HOME="$HOME/Android/Sdk"'
add_rc 'export PATH="$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator:$ANDROID_HOME/cmdline-tools/latest/bin:$PATH"'
source ~/.bashrc
sdkmanager --licenses          # user answers y
```
If `cmdline-tools/latest` is missing: SDK Manager → SDK Tools → *Android SDK Command-line Tools (latest)*.
USB debugging rules: `apt_install android-sdk-platform-tools-common` (udev rules), and the user
logs out and back in.

## Step 3 — Flutter (ask)

```bash
snap_install flutter --classic     # official snap (stable channel = latest stable)
# or the git checkout:  git clone https://github.com/flutter/flutter.git -b stable ~/dev/flutter && add_rc 'export PATH="$HOME/dev/flutter/bin:$PATH"'
flutter config --android-sdk "$HOME/Android/Sdk"
flutter doctor --android-licenses
apt_install clang cmake ninja-build pkg-config libgtk-3-dev liblzma-dev libstdc++-12-dev 2>/dev/null   # Linux desktop target
flutter doctor -v
```

## Step 4 — React Native / Expo (ask)

```bash
npm i -g eas-cli
# per project: npx create-expo-app@latest myapp   |   npx @react-native-community/cli@latest init MyApp
apt_install watchman 2>/dev/null || true
```
React Native needs `JAVA_HOME` set to the JDK its docs recommend (mise Java from 02).

## Step 5 — Tools

`apt_install scrcpy` (screen mirroring) · Kotlin Multiplatform: Android Studio + the KMP plugin.

## Verify

```bash
adb version | head -1; emulator -list-avds; flutter doctor; groups | grep -q kvm && echo "kvm group OK"
```
