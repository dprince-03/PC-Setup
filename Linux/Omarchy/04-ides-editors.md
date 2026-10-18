# 04 — IDEs and Editors (Omarchy)

**Profiles:** ides · Neovim (LazyVim, Omarchy-themed) is already configured.

```bash
omarchy-install-editor-vscode     # Microsoft VS Code (AUR visual-studio-code-bin) + libsecret keyring + Omarchy theme; disables VS Code self-update (omarchy update handles it)
# optional: omarchy-install-editor-zed | omarchy-install-editor-helix | omarchy-install-editor-emacs
aurpkg jetbrains-toolbox           # then IntelliJ IDEA / PyCharm (unified editions) from Toolbox (ask: AUR)
aurpkg postman-bin                 # or flat com.getpostman.Postman
```
Hyprland notes for JetBrains IDEs: if popups or menus misbehave on Wayland, enable the IDE's Wayland
mode (newer JetBrains versions default to it) or set `-Dawt.toolkit.name=WLToolkit` in the VM options.

VS Code extensions: per-profile list from `windows/06-ides-editors.md`.

## Verify

`code --version | head -1; pacman -Qm | grep -Ei 'toolbox|postman|visual-studio'`
