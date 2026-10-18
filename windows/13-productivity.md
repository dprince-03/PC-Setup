# 13 — Productivity Tools (Windows)

**Profiles:** productivity · **Admin:** yes

Show the list grouped by category and let the user tick what they want. Default ✔ = install unless
the user says no.

| Category | Tool | winget ID | Default |
|---|---|---|---|
| Power-user | **PowerToys** (FancyZones, Run, Color Picker, Text Extractor...) | `Microsoft.PowerToys` | ✔ |
| Search | **Everything** (instant file search) | `voidtools.Everything` | ✔ |
| Launcher | Flow Launcher | `Flow-Launcher.Flow-Launcher` | ask (overlaps with PowerToys Run) |
| Screenshots | **ShareX** | `ShareX.ShareX` | ✔ |
| Notes | **Obsidian** | `Obsidian.Obsidian` | ✔ |
| | Notion | `Notion.Notion` | ask |
| Passwords | **Bitwarden** (+ CLI) | `Bitwarden.Bitwarden`, `Bitwarden.CLI` | ✔ |
| | 1Password / KeePassXC | `AgileBits.1Password`, `KeePassXCTeam.KeePassXC` | ask |
| Communication | Slack | `SlackTechnologies.Slack` | ask |
| | Discord | `Discord.Discord` | ask |
| | Zoom | `Zoom.Zoom` | ask |
| | Microsoft Teams | `Microsoft.Teams` | ask |
| Browsers | Chrome / Firefox / Brave | see module 07 | |
| Office / docs | LibreOffice | `TheDocumentFoundation.LibreOffice` | ask |
| | SumatraPDF | `SumatraPDF.SumatraPDF` | ✔ |
| Media | VLC | `VideoLAN.VLC` | ✔ |
| | OBS Studio (screen recording, demos) | `OBSProject.OBSStudio` | ask |
| Files / sync | LocalSend (AirDrop-like) | `LocalSend.LocalSend` | ✔ |
| | Google Drive / Dropbox | `Google.GoogleDrive`, `Dropbox.Dropbox` | ask |
| Clipboard / text | Ditto clipboard manager | `Ditto.Ditto` | ask (Windows has Win+V built in) |
| Diagrams | draw.io Desktop | `JGraph.Draw` | ✔ |
| | Excalidraw | web app; nothing to install | |
| Time / focus | f.lux (if Night Light isn't enough) | `flux.flux` | ask |
| Disk / cleanup | WizTree | `AntibodySoftware.WizTree` | ✔ |
| Terminal multiplexing | (Windows Terminal panes) / Zellij / tmux in WSL | | |
| AI assistants | Claude Desktop | `Anthropic.Claude` (check with `winget search Claude`; publisher Anthropic) | ask |
| | ChatGPT Desktop | Microsoft Store | ask |

```powershell
$prod = 'Microsoft.PowerToys','voidtools.Everything','ShareX.ShareX','Obsidian.Obsidian',
        'Bitwarden.Bitwarden','SumatraPDF.SumatraPDF','VideoLAN.VLC','LocalSend.LocalSend',
        'JGraph.Draw','AntibodySoftware.WizTree'
# add the user's "ask" picks to $prod, then:
foreach ($id in $prod) { Install-App $id }
```

## PowerToys suggestions (tell the user; don't force them)

- **PowerToys Run / Command Palette** (`Alt+Space`): app/file launcher
- **FancyZones**: window layouts for multiple monitors
- **Text Extractor** (`Win+Shift+T`): OCR from anywhere on screen
- **Keyboard Manager**: remap CapsLock → Ctrl/Esc
- **Hosts File Editor** and **Environment Variables** editor: handy for devs

## Verify

`winget list` for each chosen ID. Log the installed versions.
