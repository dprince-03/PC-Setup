# 04 — IDEs, Editors and API Tools (Ubuntu)

**Profiles:** ides · Desktop only (on WSL, use the Windows-side IDEs plus *Remote - WSL*).

> JetBrains: IntelliJ IDEA (2025.3+) and PyCharm (2025.1+) are now **single unified products**.
> The free tier is built in, and the old "Community" editions are gone. Use the `intellij-idea` /
> `pycharm` snaps or Toolbox. Don't use the legacy `-community`/`-ultimate`/`-professional` snaps.

| Tool | Install (preferred → fallback) | Default |
|---|---|---|
| **VS Code** | Microsoft APT repo (below) → `snap_install code --classic` | ✔ |
| **JetBrains Toolbox** | official tarball (below) | ✔ |
| **IntelliJ IDEA** | via Toolbox → `snap_install intellij-idea --classic` | ✔ if Java |
| **PyCharm** | via Toolbox → `snap_install pycharm --classic` | ✔ if Python |
| WebStorm / GoLand / Rider / CLion / DataGrip | Toolbox | ask |
| **Postman** | `snap_install postman` (official publisher) | ✔ |
| Bruno | `snap_install bruno` or `flatpak install -y flathub com.usebruno.Bruno` | ask |
| Insomnia | `flatpak install -y flathub rest.insomnia.Insomnia` | ask |
| Neovim | apt (01) or newest: `mise use -g neovim@latest` | ask |
| Zed | `curl -f https://zed.dev/install.sh -o /tmp/zed.sh && sh /tmp/zed.sh` | ask |
| Sublime Text | official APT repo <https://www.sublimetext.com/docs/linux_repositories.html> | ask |

Pick **one** source per app. If a snap *and* a deb of the same app both exist → CONFLICT. Report it.

---

## VS Code (Microsoft repo, auto-updates with apt)

```bash
have code || {
  add_apt_repo vscode https://packages.microsoft.com/keys/microsoft.asc \
    "deb [arch=amd64,arm64,armhf] https://packages.microsoft.com/repos/code stable main"
  sudo apt update && apt_install code; }
code --version | head -1
```
Extensions: use the same per-profile lists as `windows/06-ides-editors.md` Step 2 (they're
cross-platform), except `ms-vscode-remote.remote-wsl`. Loop over them with `code --install-extension <id>`.

## JetBrains Toolbox

```bash
URL=$(curl -fsSL 'https://data.services.jetbrains.com/products/releases?code=TBA&latest=true&type=release' \
      | jq -r '.TBA[0].downloads.linux.link')            # use .linuxARM64.link on arm64
mkdir -p ~/.local/share/JetBrains/Toolbox && curl -fL "$URL" -o /tmp/toolbox.tar.gz
tar -xzf /tmp/toolbox.tar.gz -C /tmp && /tmp/jetbrains-toolbox-*/bin/jetbrains-toolbox &   # first run self-installs
```
Toolbox needs FUSE for AppImages: `apt_install libfuse2t64 || apt_install libfuse2`.
Then the user installs IntelliJ IDEA / PyCharm from Toolbox and signs in (paid features, or free
student licence via GitHub Education).

## Postman

```bash
snap_install postman
```

## Verify

```bash
code --version | head -1; snap list 2>/dev/null | grep -Ei 'postman|intellij|pycharm'; ls ~/.local/share/JetBrains/Toolbox 2>/dev/null
```
