# 03 — Databases (Ubuntu)

**Profiles:** databases · PostgreSQL, MySQL, MongoDB, Redis + GUI clients.

**Ask the mode first:** **A. Docker** (default for dev; needs module 09 Docker) · **B. Native** ·
**Mixed**. All services bind to **127.0.0.1**. Generate passwords, show them once, and never log them.

## Detect

```bash
systemctl list-units --type=service --all | grep -Ei 'postgres|mysql|mariadb|mongod|redis' || true
ss -ltnp 2>/dev/null | grep -E ':(5432|3306|27017|6379)\b' || echo "ports free"
for c in psql mysql mongod mongosh redis-server; do have $c && echo "$c: $($c --version 2>&1 | head -1)"; done
docker ps --format '{{.Names}} {{.Image}}' 2>/dev/null
```

## Mode A — Docker (recommended)

```bash
mkdir -p ~/dev-services && cd ~/dev-services
cat > compose.yaml <<'EOF'
services:
  postgres:
    image: postgres:latest
    restart: unless-stopped
    ports: ["127.0.0.1:5432:5432"]
    environment: { POSTGRES_PASSWORD: "${PG_PASSWORD}" }
    volumes: [pg:/var/lib/postgresql]
  mysql:
    image: mysql:latest
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
EOF
[ -f .env ] || printf 'PG_PASSWORD=%s\nMYSQL_PASSWORD=%s\nMONGO_PASSWORD=%s\n' \
  "$(openssl rand -hex 16)" "$(openssl rand -hex 16)" "$(openssl rand -hex 16)" > .env && chmod 600 .env
docker compose up -d && docker compose ps
```
Tell the user the passwords are in `~/dev-services/.env` (mode 600), and remove services they
don't want from the file. Clients only: `apt_install postgresql-client mysql-client redis-tools`, plus
`mongosh` (below).

## Mode B — Native

### PostgreSQL (official PGDG repo → newest major)
```bash
apt_install postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh -y     # adds apt.postgresql.org for $UCODENAME
sudo apt update
PGV=$(apt-cache search --names-only '^postgresql-[0-9]+$' | awk '{print $1}' | sort -V | tail -1)
apt_install "$PGV" postgresql-contrib
sudo -u postgres psql -c "select version();"
```
If PGDG doesn't support this codename yet (`repo_supports https://apt.postgresql.org/pub/repos/apt`
fails), use Ubuntu's own `postgresql` package or Mode A.

### MySQL
Ubuntu's archive ships a supported MySQL (usually the current LTS line):
```bash
apt_install mysql-server
sudo systemctl enable --now mysql
sudo mysql -e "select version();"
```
For the newest Innovation release instead, use Mode A (`mysql:latest`) or Oracle's APT config
package from <https://dev.mysql.com/downloads/repo/apt/> (ask). MariaDB is a different product.
Install it only if the user asks for it.

### MongoDB (official repo; check codename support first)
```bash
# Find the newest MongoDB series published for this Ubuntu codename (Rule 4b):
MV=$(curl -fsSL "https://repo.mongodb.org/apt/ubuntu/dists/$UCODENAME/mongodb-org/" 2>/dev/null \
      | grep -oE '>[0-9]+\.[0-9]+/<' | tr -d '></' | sort -V | tail -1)
echo "MongoDB series for $UCODENAME: ${MV:-NONE}"
```
- If `MV` is set:
  ```bash
  add_apt_repo mongodb-org "https://pgp.mongodb.com/server-$MV.asc" \
    "deb [arch=amd64,arm64] https://repo.mongodb.org/apt/ubuntu $UCODENAME/mongodb-org/$MV multiverse"
  sudo apt update && apt_install mongodb-org
  sudo systemctl enable --now mongod
  ```
- If `NONE` (common right after a new Ubuntu release): **use Mode A (`mongo:latest`)**. Ask
  before using the previous LTS codename's repo as a fallback. Install `mongosh` either way, from
  the same repo or the `.deb` on <https://www.mongodb.com/try/download/shell>.
- **Never** install Ubuntu's own `mongodb` package. It isn't from MongoDB Inc. and conflicts with `mongodb-org`.

### Redis (official packages.redis.io repo → newest Redis)
```bash
add_apt_repo redis https://packages.redis.io/gpg "deb https://packages.redis.io/deb $UCODENAME main"
sudo apt update && apt_install redis
sudo systemctl enable --now redis-server
redis-cli ping
```
If the Redis repo lacks this codename → `apt_install redis-server` (Ubuntu's own) or Mode A.

## GUI clients (desktop only; ask which)

| Tool | Install |
|---|---|
| DBeaver CE | `flatpak install -y flathub io.dbeaver.DBeaverCommunity` (or `snap_install dbeaver-ce`) |
| pgAdmin 4 | official repo: <https://www.pgadmin.org/download/pgadmin-4-apt/> (`pgadmin4-desktop`) |
| MongoDB Compass | latest `.deb` from `https://github.com/mongodb-js/compass/releases/latest` → `sudo apt install ./mongodb-compass_*.deb` |
| Redis Insight | `snap_install redis-insight` or `flatpak install -y flathub com.redis.RedisInsight` |
| MySQL Workbench | `snap_install mysql-workbench-community` |
| Beekeeper Studio (optional) | `flatpak install -y flathub io.beekeeperstudio.Studio` |

## Verify

```bash
psql -h 127.0.0.1 -U postgres -c 'select version();'   # or: docker compose -f ~/dev-services/compose.yaml exec postgres psql -U postgres -c 'select 1'
mysql -h 127.0.0.1 -u root -p -e 'select version();'
mongosh --quiet --eval 'db.runCommand({ping:1})'
redis-cli -h 127.0.0.1 ping
```
