# 03 — Databases (Fedora)

**Profiles:** databases · Ask: **A. Containers** (default) / **B. Native** / **Mixed**. Bind to
127.0.0.1. Passwords are typed by the user into the keyring (never generated, read or logged by the agent; no `.env`).

## Detect

```bash
systemctl list-units --type=service --all | grep -Ei 'postgres|mysql|mariadb|mongod|redis|valkey' || true
ss -ltnp | grep -E ':(5432|3306|27017|6379)\b' || echo "ports free"
```

## Mode A — Containers

Fedora ships **Podman**. Docker CE comes in module 09. Either works with the compose file from
`Linux/Ubuntu/03-databases.md` Mode A: the user stores the passwords with `secret-tool` (`dnf_install libsecret`), then use the same `compose.yaml` + `up.sh` (no `.env`).
- Docker: `~/dev-services/up.sh` (it reads the passwords from the keyring at launch)
- Podman: edit `up.sh` to call `podman compose` instead of `docker compose`. Environment-sourced
  compose secrets need a recent compose provider, so if they aren't supported, use
  `podman secret create pg_password -` (the user types the value) and mark those secrets
  `external: true`. With SELinux, named volumes just work. For bind mounts, add `:Z`.

## Mode B — Native

### PostgreSQL (Fedora repo: usually the newest major)
```bash
dnf_install postgresql-server postgresql-contrib
[ -d /var/lib/pgsql/data/base ] || sudo postgresql-setup --initdb
sudo systemctl enable --now postgresql
sudo -u postgres psql -c 'select version();'
```
If Fedora lags behind the newest major: use the official PGDG repo
(<https://www.postgresql.org/download/linux/redhat/>, Fedora section).

### MySQL
```bash
dnf_install community-mysql-server
sudo systemctl enable --now mysqld
sudo mysql -e 'select version();'
```
(Fedora defaults to MariaDB for `mysql`-named tooling. `community-mysql-*` is real MySQL.)

### MongoDB
MongoDB Inc. publishes **no Fedora repo**. Use **Mode A** (`mongo:latest`) (Rule 4b). Install
**mongosh** from MongoDB's RPM (<https://www.mongodb.com/try/download/shell>) or `npm i -g mongosh`.

### Redis / Valkey
Recent Fedora releases replaced `redis` with **Valkey** (the Linux Foundation's Redis-compatible
fork, same protocol, `valkey-cli`). Ask the user:
- Valkey native: `dnf_install valkey && sudo systemctl enable --now valkey`
- Real Redis (newest): Mode A with `redis:latest`

## GUI clients

```bash
flat io.dbeaver.DBeaverCommunity       # DBeaver CE
flat com.redis.RedisInsight            # Redis Insight
# MongoDB Compass: latest .rpm from https://github.com/mongodb-js/compass/releases/latest → sudo dnf install ./mongodb-compass-*.rpm
# pgAdmin 4: official repo https://www.pgadmin.org/download/pgadmin-4-rpm/
```

## Verify

Same commands as Ubuntu 03 Verify (`valkey-cli ping` for Valkey).
