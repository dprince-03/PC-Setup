# 06 — Mobile (Omarchy)

**Profiles:** mobile · Same as `Linux/Arch/06-mobile.md`, using `pkg`/`aurpkg`:

```bash
pkg android-tools android-udev qemu-full
sudo usermod -aG kvm "$USER"
aurpkg android-studio          # ask (AUR); or install it via JetBrains Toolbox
```
Flutter: git clone the stable channel (Arch 06). On Hyprland, the Android Emulator may need
`QT_QPA_PLATFORM=xcb` if its window misbehaves.

## Verify

`adb version | head -1; flutter doctor`
