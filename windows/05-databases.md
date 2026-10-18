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

All services must bind to **127.0.0.1 only** (AGENT.md Rule 6).

**Secrets rule (AGENT.md Rule 6a):** the **user** chooses and types every database password. Passwords
are stored in the Windows Credential Manager through PowerShell SecretManagement (module 14 Step 5).
The agent never generates, reads or prints them, and there are **no `.env` files**.

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

**1. The user stores the passwords** (hidden prompts; run these in the user's own terminal):
```powershell
pwsh -c "Install-Module Microsoft.PowerShell.SecretManagement, Microsoft.PowerShell.SecretStore -Scope CurrentUser -Force"
pwsh -c "if (-not (Get-SecretVault -EA 0)) { Register-SecretVault -Name LocalStore -ModuleName Microsoft.PowerShell.SecretStore -DefaultVault }"
pwsh -c "Set-Secret -Name pg_password"       # prompts for the value
pwsh -c "Set-Secret -Name mysql_password"
pwsh -c "Set-Secret -Name mongo_password"
```

**2. `%USERPROFILE%\dev-services\compose.yaml`.** Passwords arrive as files under `/run/secrets`:
```yaml
services:
  postgres:
    image: postgres:latest
    restart: unless-stopped
    ports: ["127.0.0.1:5432:5432"]
    environment: { POSTGRES_PASSWORD_FILE: /run/secrets/pg_password }
    secrets: [pg_password]
    volumes: [pg:/var/lib/postgresql]
  mysql:
    image: mysql:latest            # 'lts' tag is also available: mysql:lts
    restart: unless-stopped
    ports: ["127.0.0.1:3306:3306"]
    environment: { MYSQL_ROOT_PASSWORD_FILE: /run/secrets/mysql_password }
    secrets: [mysql_password]
    volumes: [mysql:/var/lib/mysql]
  mongo:
    image: mongo:latest
    restart: unless-stopped
    ports: ["127.0.0.1:27017:27017"]
    environment: { MONGO_INITDB_ROOT_USERNAME: admin, MONGO_INITDB_ROOT_PASSWORD_FILE: /run/secrets/mongo_password }
    secrets: [mongo_password]
    volumes: [mongo:/data/db]
  redis:
    image: redis:latest
    restart: unless-stopped
    ports: ["127.0.0.1:6379:6379"]
    volumes: [redis:/data]
secrets:
  pg_password:    { environment: PG_PASSWORD }
  mysql_password: { environment: MYSQL_PASSWORD }
  mongo_password: { environment: MONGO_PASSWORD }
volumes: { pg: {}, mysql: {}, mongo: {}, redis: {} }
```

**3. `%USERPROFILE%\dev-services\up.ps1`.** It reads the vault at launch, and nothing is written to
disk or printed:
```powershell
$ErrorActionPreference = 'Stop'
$env:PG_PASSWORD    = Get-Secret pg_password    -AsPlainText
$env:MYSQL_PASSWORD = Get-Secret mysql_password -AsPlainText
$env:MONGO_PASSWORD = Get-Secret mongo_password -AsPlainText
try { docker compose -f "$HOME\dev-services\compose.yaml" up -d @args }
finally { Remove-Item Env:PG_PASSWORD, Env:MYSQL_PASSWORD, Env:MONGO_PASSWORD -EA 0 }
```
Run it with `pwsh $HOME\dev-services\up.ps1`, and remove the services the user doesn't want.
For the CLI clients without native servers: `Install-App 'MongoDB.Shell'`, and `psql`/`mysql`
via `docker exec -it <container> psql -U postgres`.

## Mode B — Native installs

### PostgreSQL
```powershell
$pg = Resolve-LatestId 'PostgreSQL.PostgreSQL'      # newest major, e.g. PostgreSQL.PostgreSQL.18
# Interactive install: the USER types the superuser password in the installer (never pass it on the command line)
Install-App $pg @('--interactive')
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
