# 08 — Desktop App Development (Arch)

**Profiles:** desktop · Ask which stacks.

| Stack | Install |
|---|---|
| GTK 4 / libadwaita | `pac gtk4 libadwaita meson ninja python-gobject blueprint-compiler gnome-builder` |
| Qt 6 | `pac qt6-base qt6-declarative qt6-tools qtcreator` |
| PySide6 | `uv add pyside6` per project |
| Electron | Node → `npm create electron-app@latest` |
| Tauri | Rust + `pac webkit2gtk-4.1 base-devel curl wget file openssl appmenu-gtk-module libappindicator-gtk3 librsvg xdotool` |
| Flutter desktop | module 06 + `flutter config --enable-linux-desktop` |
| .NET Avalonia | `dotnet new install Avalonia.Templates` |
| Packaging | `pac flatpak-builder`; AppImage: `aur appimagetool-bin`; Arch packages: write a PKGBUILD + `namcap` |

## Verify

`pkg-config --modversion gtk4; qmake6 --version | head -1`
