# 03 — Databases (Omarchy)

**Profiles:** databases · Docker is preinstalled, so containers are the natural default here.

## Option 1 — Omarchy's built-in helper (quick)

```bash
omarchy-install-docker-dbs PostgreSQL MySQL Redis MongoDB      # or run it with no args for a menu
```
Tell the user what it does (from the script): it binds to **127.0.0.1**, uses
`--restart unless-stopped`, and sets **dev-only credentials** (PostgreSQL `trust` auth, MySQL empty
root password, MongoDB `admin/admin123`). It pins specific image majors chosen by Omarchy, which
may not be the newest. Fine for local dev, never for anything exposed.

## Option 2 — Latest images + real passwords (AGENT.md default)

Use the keyring + compose secrets setup from `Linux/Ubuntu/03-databases.md` Mode A (the user types the passwords; `pkg libsecret` for `secret-tool`)
(`postgres:latest`, `mysql:latest`, `mongo:latest`, `redis:latest`).
If Option 1 containers already exist on the same ports → CONFLICT. Ask which to keep.

## Native (only if asked)

Same table as `Linux/Arch/03-databases.md`, with `pkg` instead of `pac` and `aurpkg` instead of `aur`.

**GUIs:** `pkg dbeaver` · `aurpkg mongodb-compass-bin` · `flat com.redis.RedisInsight`. Omarchy also
ships `lazydocker` (`omarchy-launch-docker-tui`) for managing containers.

## Verify

`docker ps --format '{{.Names}}\t{{.Ports}}'` and the client checks from Ubuntu 03.
