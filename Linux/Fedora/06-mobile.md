# 06 — Mobile (Fedora)

**Profiles:** mobile

```bash
# KVM for the emulator
dnf_install @virtualization 2>/dev/null || sudo dnf group install -y virtualization
sudo usermod -aG kvm "$USER"
ls -l /dev/kvm
```

**Android Studio:** install it via **JetBrains Toolbox** (lists Android Studio, latest stable), or
use the official tarball from <https://developer.android.com/studio> extracted to `/opt/android-studio`.
The Flathub build isn't published by Google, so offer it only as a fallback.
Then run the Setup Wizard and set `ANDROID_HOME` + PATH exactly as in `Linux/Ubuntu/06-mobile.md`
Step 2, then `sdkmanager --licenses`.

udev rules for physical devices: `dnf_install android-tools` (adb/fastboot + udev rules).

**Flutter:**
```bash
git clone https://github.com/flutter/flutter.git -b stable ~/dev/flutter 2>/dev/null || git -C ~/dev/flutter pull
grep -qxF 'export PATH="$HOME/dev/flutter/bin:$PATH"' ~/.bashrc || echo 'export PATH="$HOME/dev/flutter/bin:$PATH"' >> ~/.bashrc
export PATH="$HOME/dev/flutter/bin:$PATH"
dnf_install clang cmake ninja-build pkgconf-pkg-config gtk3-devel xz-devel libstdc++-static   # Linux desktop target
flutter config --android-sdk "$HOME/Android/Sdk"; flutter doctor --android-licenses; flutter doctor -v
```

**React Native / Expo:** `npm i -g eas-cli`; scaffold per project (see Ubuntu 06 Step 4).
`dnf_install watchman` if available.

**Tools:** `dnf_install scrcpy` (RPM Fusion or COPR; ask if a COPR is needed).

## Verify

`adb version | head -1; flutter doctor; groups | grep -q kvm && echo kvm OK`
