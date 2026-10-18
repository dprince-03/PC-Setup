# Fedora Playbook

Target: **Fedora Workstation** and spins (KDE, Xfce, Sway, Budgie...), current or previous release.
Package manager: **dnf5** (`dnf`) + **Flatpak/Flathub**, plus vendor repos where noted.

> Agent: read [`../../AGENT.md`](../../AGENT.md) first. Run modules in numeric order, only those the
> chosen profiles need.

**Fedora Atomic desktops** (Silverblue, Kinoite, Bazzite, Aurora; detect with
`[ -e /run/ostree-booted ]`): the base OS is immutable. Rules for these:
- Install GUI apps from **Flathub** only.
- Do dev work inside a **Toolbx** container (`toolbox create && toolbox enter`) and run these
  modules *inside* it (they behave like normal Fedora there).
- Layer with `rpm-ostree install` only for things that must be on the host (drivers, Docker,
  virtualization), and **ask first**. Every layer needs a reboot.

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

**dnf5 syntax reminder:** add a repo with `sudo dnf config-manager addrepo --from-repofile=<url>`.
The old dnf4 form was `--add-repo`. On Fedora ≤ 40 (dnf4), use the old form.
