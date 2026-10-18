# 04 — IDEs, Editors and API Tools (Fedora)

**Profiles:** ides · JetBrains IntelliJ IDEA and PyCharm are now unified products (no separate
Community download).

| Tool | Install | Default |
|---|---|---|
| **VS Code** | Microsoft RPM repo (below) | ✔ |
| **JetBrains Toolbox** → IntelliJ IDEA, PyCharm, others | official tarball (same as Ubuntu 04) | ✔ |
| IntelliJ / PyCharm without Toolbox | `flat com.jetbrains.IntelliJ-IDEA-Ultimate` / `flat com.jetbrains.PyCharm-Professional` (Flathub IDs may change after the unification, so check with `flatpak search intellij`) | fallback |
| **Postman** | `flat com.getpostman.Postman` | ✔ |
| Bruno / Insomnia | `flat com.usebruno.Bruno` / `flat rest.insomnia.Insomnia` | ask |
| Neovim | `dnf_install neovim` | ✔ |
| Zed | `dnf_install zed` if packaged, otherwise the official script | ask |

## VS Code

```bash
have code || {
  sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
  printf '[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\nautorefresh=1\ntype=rpm-md\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc\n' \
    | sudo tee /etc/yum.repos.d/vscode.repo >/dev/null
  dnf_install code; }
```
Extensions: per-profile lists from `windows/06-ides-editors.md` (skip `remote-wsl`).

**Flatpak IDE caveat:** sandboxed IDEs can't see host SDKs (mise/uv/JDK) without extra setup. Prefer
Toolbox/RPM for IDEs. If a Flatpak IDE is used, explain `flatpak-spawn --host` and SDK extensions.

## JetBrains Toolbox

Same commands as `Linux/Ubuntu/04-ides-editors.md` (JetBrains Toolbox section), with `dnf_install fuse fuse-libs` for AppImage support.

## Verify

`code --version | head -1; flatpak list --app | grep -Ei 'postman|jetbrains'`
