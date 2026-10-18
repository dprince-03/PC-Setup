# Ubuntu Playbook

Target: **Ubuntu LTS / interim releases** (desktop or server; WSL supported) and derivatives
(Linux Mint, Pop!_OS, Zorin, elementary, KDE neon).

> Agent: read [`../../AGENT.md`](../../AGENT.md) first. Run modules in numeric order, only those the
> chosen profiles need. Package manager: **apt** (+ **snap** where noted, + optional **Flatpak**).

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
| 12 | [Security and antivirus](12-security-antivirus.md) | security (default ON) |
| 13 | [Verify and report](13-verify-and-report.md) | all |

**Codename rule:** third-party repos are keyed by the *Ubuntu* codename. On derivatives, use
`$UBUNTU_CODENAME` (e.g. Mint reports its own `VERSION_CODENAME`). Preflight exports
`$UCODENAME` for this.

**Brand-new release rule (AGENT.md 4b):** right after an Ubuntu release, some vendors (MongoDB,
sometimes Docker/MySQL) haven't published a repo for the new codename yet. Each module shows how
to check (`curl -fsI <repo>/dists/$UCODENAME/Release`) and falls back to the Docker route.
