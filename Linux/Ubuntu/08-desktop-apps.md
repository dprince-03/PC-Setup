# 08 — Desktop App Development (Ubuntu)

**Profiles:** desktop · Ask which stacks. Install only those.

| Stack | Install |
|---|---|
| **GTK 4 / libadwaita** (C, Python, Rust, Vala) | `apt_install libgtk-4-dev libadwaita-1-dev meson ninja-build python3-gi gir1.2-gtk-4.0 blueprint-compiler` + `flatpak install -y flathub org.gnome.Builder` |
| **Qt 6** (C++/QML) | `apt_install qt6-base-dev qt6-declarative-dev qt6-tools-dev qml6-module-qtquick-controls qtcreator` (newest Qt: the official online installer from qt.io) |
| **PySide6 / PyQt** | per project: `uv add pyside6` |
| **Electron** | Node (02); per project: `npm create electron-app@latest` |
| **Tauri** | Rust (02) + `apt_install libwebkit2gtk-4.1-dev libxdo-dev libssl-dev libayatana-appindicator3-dev librsvg2-dev`; `npm create tauri-app@latest` |
| **Flutter desktop** | module 06 + `flutter config --enable-linux-desktop` |
| **.NET (Avalonia / Uno)** | .NET SDK (02) + `dotnet new install Avalonia.Templates` |
| **Java desktop (JavaFX)** | JDK (02); JavaFX via Maven/Gradle `org.openjfx` per project |
| **Packaging** | `apt_install flatpak-builder` (Flatpak), `snap_install snapcraft --classic`, AppImage via `appimagetool` (`mise use -g ubi:AppImage/appimagetool`) |

## Verify

```bash
pkg-config --modversion gtk4 2>/dev/null; qmake6 --version 2>/dev/null | head -1; rustc --version; dotnet new list 2>/dev/null | grep -i avalonia
```
