# 04 — IDEs and Editors (Arch)

**Profiles:** ides

| Tool | Install | Note |
|---|---|---|
| **VS Code (Microsoft build)** | `aur visual-studio-code-bin` | the official `code` package is **Code-OSS** (Open VSX marketplace, no MS-only extensions like Pylance/C# Dev Kit). Ask which |
| **JetBrains Toolbox** | `aur jetbrains-toolbox` | then IntelliJ IDEA / PyCharm (unified editions) from Toolbox |
| IntelliJ IDEA / PyCharm direct | `aur intellij-idea-ultimate-edition` / `aur pycharm-professional` (these now ship the unified builds; check the AUR page) | or via Toolbox (preferred) |
| **Postman** | `aur postman-bin` | or `flat com.getpostman.Postman` |
| Bruno | `pac bruno` (if in extra) / `aur bruno-bin` | |
| Neovim | `pac neovim` | |
| Zed | `pac zed` | |

Extensions for VS Code: per-profile list from `windows/06-ides-editors.md` (skip remote-wsl).

## Verify

`code --version | head -1; pacman -Qm | grep -Ei 'toolbox|postman|code'`
