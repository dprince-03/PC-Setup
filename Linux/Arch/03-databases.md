# 03 — Databases (Arch)

**Profiles:** databases · Ask: **A. Docker** (default) / **B. Native** / **Mixed**. Bind to 127.0.0.1.

**Mode A:** Docker from module 09, then the compose file from `Linux/Ubuntu/03-databases.md` Mode A.

**Mode B (native):**

| DB | Source | Commands |
|---|---|---|
| **PostgreSQL** | official repo (newest) | `pac postgresql && sudo -iu postgres initdb -D /var/lib/postgres/data && sudo systemctl enable --now postgresql` |
| **MySQL** | *not* in the official repos (Arch ships MariaDB) | Docker (recommended), or `aur mysql` (builds from source, very slow; ask). `pac mariadb` only if the user is happy with MariaDB |
| **MongoDB** | AUR only | Docker (recommended), or `aur mongodb-bin mongosh-bin` (upstream binaries; ask) |
| **Redis** | Arch moved to **Valkey** (Redis-compatible fork) | `pac valkey && sudo systemctl enable --now valkey`, or Docker `redis:latest` for real Redis |

PostgreSQL major upgrades on Arch need a manual `pg_upgrade` (see the Arch Wiki). Warn the user.
Add `postgresql postgresql-libs` to `IgnorePkg` **only if they ask**.

**GUI clients:** `pac dbeaver` · `aur mongodb-compass-bin` · `flat com.redis.RedisInsight` ·
`aur pgadmin4-desktop` (ask).

## Verify

Same as Ubuntu 03 (`valkey-cli ping` for Valkey).
