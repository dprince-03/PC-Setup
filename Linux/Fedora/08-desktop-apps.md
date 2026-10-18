# 08 — Desktop App Development (Fedora)

**Profiles:** desktop · Ask which stacks.

| Stack | Install |
|---|---|
| GTK 4 / libadwaita | `dnf_install gtk4-devel libadwaita-devel meson ninja-build python3-gobject blueprint-compiler` + `flat org.gnome.Builder` |
| Qt 6 | `dnf_install qt6-qtbase-devel qt6-qtdeclarative-devel qt6-qttools-devel qt-creator` |
| PySide6 | per project `uv add pyside6` |
| Electron | Node (02) → `npm create electron-app@latest` |
| Tauri | Rust (02) + `dnf_install webkit2gtk4.1-devel openssl-devel curl wget file libappindicator-gtk3-devel librsvg2-devel libxdo-devel` + `sudo dnf group install -y c-development` |
| Flutter desktop | module 06 + `flutter config --enable-linux-desktop` |
| .NET Avalonia | `dotnet new install Avalonia.Templates` |
| Packaging | `dnf_install flatpak-builder rpm-build rpmdevtools` (`rpmdev-setuptree`) |

## Verify

`pkg-config --modversion gtk4; qmake6 --version 2>/dev/null | head -1; rustc --version`
