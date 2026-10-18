# 05 — Databases (Windows)

**Profiles:** databases · **Admin:** yes

Databases: **PostgreSQL, MySQL, MongoDB, Redis**, plus GUI clients.

## Choose the install mode (ask the user once)

| Mode | Pros | Cons |
|---|---|---|
| **A. Docker containers** (recommended for dev) | one command each, any version, no services left running, matches prod | needs Docker Desktop (module 11) running |
| **B. Native Windows services** | no Docker needed, starts with Windows | uses RAM all the time; upgrades are harder |
| **Mixed** | e.g. native PostgreSQL + Docker Redis | |

> **Redis has no official native Windows build.** Options: Docker (mode A), Redis inside WSL2
> (`sudo apt install redis-server`), or **Memurai Developer** (`Memurai.MemuraiDeveloper`), the
> Redis-endorsed Windows port (Redis 7-compatible). Don't use the very old `Redis.Redis`
> (MSOpenTech 3.x) winget package.

All services must bind to **127.0.0.1 only** (AGENT.md Rule 6). Generate passwords randomly, show
them to the user once, and save them in the user's password manager. Never write them to the log.

---

## Detect

```powershell
Get-Service *postgres*, *mysql*, *mongo*, *redis*, *memurai* -ErrorAction SilentlyContinue | Format-Table Name,Status,StartType
'psql --version','mysql --version','mongod --version','mongosh --version','redis-cli --version' |
  % { try { iex "$_ 2>&1" | Select -First 1 } catch { "$_ : missing" } }
docker ps --format '{{.Names}} {{.Image}}' 2>$null
Get-NetTCPConnection -State Listen -LocalPort 5432,3306,27017,6379 -EA 0 | Select LocalAddress,LocalPort,OwningProcess
```
If a port is already taken, report what's using it. Don't start a second server on the same port.

## Mode A — Docker (after module 11 has installed Docker Desktop)

Use a single compose file the user owns. `latest`-style tags follow AGENT.md Rule 4. Pin the
**major** when a project needs it.

`%USERPROFILE%\dev-services\compose.yaml`:
```yaml
services:
  postgres:
    image: postgres:latest
    restart: unless-stopped
    ports: ["127.0.0.1:5432:5432"]
    environment: { POSTGRES_PASSWORD: "${PG_PASSWORD}" }
    volumes: [pg:/var/lib/postgresql]
  mysql:
    image: mysql:latest            # 'lts' tag is also available: mysql:lts
    restart: unless-stopped
    ports: ["127.0.0.1:3306:3306"]
    environment: { MYSQL_ROOT_PASSWORD: "${MYSQL_PASSWORD}" }
    volumes: [mysql:/var/lib/mysql]
  mongo:
    image: mongo:latest
    restart: unless-stopped
    ports: ["127.0.0.1:27017:27017"]
    environment: { MONGO_INITDB_ROOT_USERNAME: admin, MONGO_INITDB_ROOT_PASSWORD: "${MONGO_PASSWORD}" }
    volumes: [mongo:/data/db]
  redis:
    image: redis:latest
    restart: unless-stopped
    ports: ["127.0.0.1:6379:6379"]
    volumes: [redis:/data]
volumes: { pg: {}, mysql: {}, mongo: {}, redis: {} }
```
Put the passwords in `%USERPROFILE%\dev-services\.env` (`PG_PASSWORD=...` etc.). The user stores
them. Then `docker compose -f $HOME\dev-services\compose.yaml up -d`.
For the CLI clients without native servers: `Install-App 'MongoDB.Shell'`, and `psql`/`mysql`
via `docker exec -it <container> psql -U postgres`.

## Mode B — Native installs

### PostgreSQL
```powershell
$pg = Resolve-LatestId 'PostgreSQL.PostgreSQL'      # newest major, e.g. PostgreSQL.PostgreSQL.18
Install-App $pg @('--override','--mode unattended --unattendedmodeui none --superpassword "<generated>" --serverport 5432 --disable-components stackbuilder')
# Add bin to PATH if the installer didn't:
$bin = (Get-ChildItem 'C:\Program Files\PostgreSQL' -Directory | Sort-Object { [int]$_.Name } | Select -Last 1).FullName + '\bin'
[Environment]::SetEnvironmentVariable('Path', [Environment]::GetEnvironmentVariable('Path','User') + ";$bin", 'User')
```
In `postgresql.conf`, check that `listen_addresses = 'localhost'`.

### MySQL
`Oracle.MySQL` installs MySQL Community Server. Check `winget show Oracle.MySQL --versions`.
Take the newest **GA** (an LTS 8.4.x or an Innovation 9.x). Ask which line the user prefers.
LTS is the safer default.
```powershell
Install-App 'Oracle.MySQL'
```
Then configure it as a service bound to 127.0.0.1 using MySQL Configurator (it opens after install).
The user sets the root password interactively.

### MongoDB
```powershell
Install-App 'MongoDB.Server'      # newest GA Community Server, installs as a service
Install-App 'MongoDB.Shell'       # mongosh
Install-App 'MongoDB.DatabaseTools'
```
The default config at `C:\Program Files\MongoDB\Server\<ver>\bin\mongod.cfg` already has
`bindIp: 127.0.0.1`. Check it.

### Redis
Ask: Memurai Developer (native), or Redis in WSL2/Docker.
```powershell
Install-App 'Memurai.MemuraiDeveloper'
```

## GUI clients (ask which)

| Tool | winget ID | Covers |
|---|---|---|
| DBeaver Community | `DBeaver.DBeaver` | everything (SQL + Mongo via plugin) |
| pgAdmin 4 | `PostgreSQL.pgAdmin` | PostgreSQL |
| MySQL Workbench | `Oracle.MySQLWorkbench` | MySQL |
| MongoDB Compass | `MongoDB.Compass.Full` (check with `winget search MongoDB.Compass`) | MongoDB |
| Redis Insight | `RedisInsight.RedisInsight` | Redis |
| DataGrip (paid / free non-commercial) | `JetBrains.DataGrip` | everything |

## Verify

```powershell
# Docker mode
docker compose -f $HOME\dev-services\compose.yaml ps
docker exec <pg-container> psql -U postgres -c "select version();"
docker exec <redis-container> redis-cli ping            # PONG
# Native mode
psql -U postgres -h 127.0.0.1 -c "select version();"
mysql -u root -p -h 127.0.0.1 -e "select version();"
mongosh --quiet --eval "db.runCommand({ping:1})"
memurai-cli ping
```
