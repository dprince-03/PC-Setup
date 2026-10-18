# Arch Linux Playbook

Target: **vanilla Arch Linux** (rolling). Package managers: **pacman** (official repos) +
**yay** (AUR helper) + optional **Flatpak**. For Omarchy, use [`../Omarchy/`](../Omarchy/) instead.

> Agent: read [`../../AGENT.md`](../../AGENT.md) first.

Arch rules for the agent:
- **Never partial-upgrade.** Always `pacman -Syu` before installing (`-Sy pkg` alone is forbidden).
- Official repos first. **AUR only with consent:** show the user the package's AUR page and
  maintainer, and read the PKGBUILD before building. Prefer `-bin` packages published by the
  upstream vendor.
- Read the **Arch news** (<https://archlinux.org/news/>) before big upgrades. Some need manual
  steps.
- Arch is rolling, so the official repos already carry the *latest* versions. mise is still used
  for runtimes where projects need LTS or several versions (Node, Java).

| # | Module | Profiles |
|---|---|---|
| 00 | [Preflight](00-preflight.md) | all |
| 01 | [Base system](01-base-system.md) | core |
| 02 | [Languages](02-languages.md) | languages |
| 03 | [Databases](03-databases.md) | databases |
| 04 | [IDEs and editors](04-ides-editors.md) | ides |
| 05 | [Full-stack web](05-fullstack-web.md) | fullstack |
| 06 | [Mobile](06-mobile.md) | mobile |
| 07 | [Data science / ML](07-data-science-ml.md) | data-ml |
| 08 | [Desktop apps](08-desktop-apps.md) | desktop |
| 09 | [DevOps and cloud](09-devops-cloud.md) | devops-cloud |
| 10 | [Sysadmin and networking](10-sysadmin-networking.md) | sysadmin-net, linux-admin |
| 11 | [Productivity](11-productivity.md) | productivity |
| 12 | [Verify and report](12-verify-and-report.md) | all |
