# Windows Playbook

Target: **Windows 11** (23H2 or newer), x64 or ARM64. Windows 10 22H2 mostly works, but it's out
of mainstream support. Warn the user, then continue only if they agree.

> Agent: read [`../AGENT.md`](../AGENT.md) first. Run modules in order, skipping any whose
> profiles weren't selected. Use **PowerShell 7 (`pwsh`) as Administrator** unless a step says
> otherwise. Module 02 installs it; until then use Windows PowerShell 5.1 as Administrator.

| # | Module | Profiles | Needs admin | May reboot |
|---|---|---|---|---|
| 00 | [Preflight](00-preflight.md) | all | yes | no |
| 01 | [Debloat and privacy](01-debloat-and-privacy.md) | debloat (ask) | yes | yes |
| 02 | [Foundation](02-foundation.md) | core | yes | no |
| 03 | [WSL2 and Linux](03-wsl.md) | linux-admin, devops-cloud, data-ml | yes | **yes** |
| 04 | [Languages and SDKs](04-languages.md) | languages | partly | no |
| 05 | [Databases](05-databases.md) | databases | yes | no |
| 06 | [IDEs and editors](06-ides-editors.md) | ides | yes | no |
| 07 | [Full-stack web](07-fullstack-web.md) | fullstack | no | no |
| 08 | [Mobile](08-mobile.md) | mobile | yes | maybe |
| 09 | [Data science / ML / DL](09-data-science-ml.md) | data-ml | yes | maybe |
| 10 | [Desktop app dev](10-desktop-apps.md) | desktop | yes | no |
| 11 | [DevOps and cloud](11-devops-cloud.md) | devops-cloud | yes | **yes** |
| 12 | [Sysadmin and networking](12-sysadmin-networking.md) | sysadmin-net | yes | maybe |
| 13 | [Productivity](13-productivity.md) | productivity | yes | no |
| 14 | [Security and antivirus](14-security-antivirus.md) | security (default ON) | yes | maybe |
| 15 | [Verify and report](15-verify-and-report.md) | all | no | no |

## The install pattern (used everywhere)

Every module uses the helper functions defined in **00-preflight.md → Step 6**. Load them into the
session first:

- `Test-App <WingetId> [<command>]`: reports `MISSING` / `OUTDATED` / `CURRENT`
- `Install-App <WingetId>`: installs the latest stable from the `winget` source, or upgrades it
- `Resolve-LatestId <prefix>`: for IDs with the version baked in (`Python.Python.3.*`,
  `EclipseAdoptium.Temurin.*.JDK`, ...), returns the ID with the highest stable version
- `Update-SessionPath`: reloads PATH so newly installed tools can be found without a new shell

**Package ID policy:** the IDs in these files were checked against `microsoft/winget-pkgs`.
IDs do change, so always confirm with `winget show --id <Id> -e` first. If it isn't found,
run `winget search <name>` and choose the entry whose **Publisher** is the official vendor.
