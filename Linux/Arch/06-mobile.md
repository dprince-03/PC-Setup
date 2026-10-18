# 06 — Mobile (Arch)

**Profiles:** mobile

```bash
pac qemu-full libvirt android-tools android-udev   # adb/fastboot + udev rules; KVM is in the kernel
sudo usermod -aG kvm "$USER"
aur android-studio                                  # tracks the newest stable from Google (ask; AUR)
```
Alternatively install Android Studio via JetBrains Toolbox. Then run the first-run wizard, set
`ANDROID_HOME`/PATH as in `Linux/Ubuntu/06-mobile.md` Step 2, and run `sdkmanager --licenses`.

**Flutter:** `git clone https://github.com/flutter/flutter.git -b stable ~/dev/flutter` + PATH (as in
Fedora 06), `pac clang cmake ninja pkgconf gtk3` for Linux desktop, then `flutter doctor -v`.
**React Native / Expo:** `npm i -g eas-cli`; `pac watchman` if available, otherwise `aur watchman-bin`.
**scrcpy:** `pac scrcpy`.

## Verify

`adb version | head -1; flutter doctor; groups | grep -q kvm && echo kvm OK`
