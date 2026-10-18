# 11 — Productivity (Ubuntu)

**Profiles:** productivity · Desktop only. Let the user tick from this list.

| Category | Tool | Install | Default |
|---|---|---|---|
| Launcher | Ulauncher / GNOME search | `apt_install ulauncher` (or its PPA, ask) | ask |
| Clipboard | GNOME extension *Clipboard Indicator* / CopyQ | `apt_install copyq` | ✔ |
| Screenshots | Flameshot | `apt_install flameshot` | ✔ |
| Notes | Obsidian | `flatpak install -y flathub md.obsidian.Obsidian` | ✔ |
| Passwords | Bitwarden | `flatpak install -y flathub com.bitwarden.desktop` | ✔ |
| | KeePassXC | `apt_install keepassxc` | ask |
| Comms | Slack / Discord / Zoom | `flatpak install -y flathub com.slack.Slack com.discordapp.Discord us.zoom.Zoom` | ask |
| Office | LibreOffice | usually preinstalled; else `apt_install libreoffice` | |
| PDF | Okular / Evince | preinstalled or `apt_install okular` | |
| Media | VLC, OBS | `apt_install vlc`; `flatpak install -y flathub com.obsproject.Studio` | ✔ / ask |
| File sharing | LocalSend | `flatpak install -y flathub org.localsend.localsend_app` | ✔ |
| Diagrams | draw.io | `flatpak install -y flathub com.jgraph.drawio.desktop` | ✔ |
| Sync | Syncthing | `apt_install syncthing` | ask |
| GNOME tweaks | GNOME Tweaks + Extension Manager | `apt_install gnome-tweaks gnome-shell-extension-manager` | ✔ (GNOME only) |
| Terminal | Ghostty / Kitty / WezTerm | `snap_install ghostty --classic` / `apt_install kitty` | ask |
| Disk usage | Baobab / ncdu | preinstalled / done in 01 | |
| AI | Claude Desktop / ChatGPT | the vendor's official Linux build if available, otherwise the web app | ask |

## Verify

`flatpak list --app` and `apt list --installed 2>/dev/null | grep -E 'flameshot|copyq|vlc'`. Log the versions.
