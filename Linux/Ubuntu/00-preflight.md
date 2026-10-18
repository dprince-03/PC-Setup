# 00 — Preflight (Ubuntu)

**Profiles:** all · **Goal:** confirm the distro, get sudo, take an inventory, take a snapshot,
and load the helpers.

---

## Step 1 — Confirm the OS

```bash
. /etc/os-release
echo "$PRETTY_NAME | ID=$ID | ID_LIKE=${ID_LIKE:-} | VERSION_ID=$VERSION_ID"
export UCODENAME="${UBUNTU_CODENAME:-$VERSION_CODENAME}"
echo "Ubuntu base codename: $UCODENAME"
uname -srm; dpkg --print-architecture          # amd64 / arm64
IS_WSL=$(grep -qi microsoft /proc/version && echo 1 || echo 0); echo "WSL=$IS_WSL"
echo "Desktop: ${XDG_CURRENT_DESKTOP:-none}  Session: ${XDG_SESSION_TYPE:-tty}"
df -h / | tail -1; free -h | awk '/Mem/{print "RAM:",$2}'
```
- If `ID` isn't `ubuntu` and `ID_LIKE` doesn't contain `ubuntu` → **STOP** (wrong playbook).
- If `VERSION_ID` is an interim (non-LTS) release that's past end of life → warn.
- Note the architecture. Some vendor repos (MongoDB, Android emulator images) differ on `arm64`.

## Step 2 — sudo

```bash
sudo -v      # the user types their password; never store it
```

## Step 3 — Log folder

```bash
export LOG="$HOME/pc-setup-logs/$(date +%Y%m%d-%H%M%S)"; mkdir -p "$LOG"
echo "$(date -Is) [00] $PRETTY_NAME $(uname -r) $(dpkg --print-architecture) WSL=$IS_WSL" | tee "$LOG/os.txt"
```

## Step 4 — Snapshot

```bash
if command -v timeshift >/dev/null; then
  sudo timeshift --create --comments "PC-Setup before changes" --tags O
else
  echo "No Timeshift. Ask: install it (sudo apt install timeshift) and snapshot, or continue without?"
fi
```
Skip this on WSL. Suggest `wsl --export <distro> backup.tar` from Windows instead.

## Step 5 — Inventory

```bash
apt list --installed 2>/dev/null > "$LOG/inventory-before-apt.txt"
snap list 2>/dev/null           > "$LOG/inventory-before-snap.txt"
flatpak list --app 2>/dev/null  > "$LOG/inventory-before-flatpak.txt"
ls /etc/apt/sources.list.d/     > "$LOG/apt-sources-before.txt"

probe() { local n=$1; shift; if command -v "$1" >/dev/null 2>&1; then
  printf '%-12s %-45s %s\n' "$n" "$("$@" 2>&1 | head -1)" "$(command -v "$1")"; else printf '%-12s MISSING\n' "$n"; fi; }
{
probe git git --version; probe gh gh --version; probe curl curl --version; probe mise mise --version
probe python3 python3 --version; probe uv uv --version; probe node node -v; probe npm npm -v; probe pnpm pnpm -v
probe bun bun -v; probe java java -version; probe mvn mvn -v; probe gradle gradle -v; probe go go version
probe gcc gcc --version; probe clang clang --version; probe cmake cmake --version; probe dotnet dotnet --version
probe rustc rustc --version; probe docker docker --version; probe podman podman --version
probe kubectl kubectl version --client; probe helm helm version --short; probe terraform terraform -version
probe tofu tofu -version; probe ansible ansible --version; probe aws aws --version; probe az az version
probe gcloud gcloud --version; probe psql psql --version; probe mysql mysql --version; probe mongod mongod --version
probe mongosh mongosh --version; probe redis-cli redis-cli --version; probe code code --version
probe flutter flutter --version; probe adb adb version; probe nvidia-smi nvidia-smi -L
} | tee "$LOG/inventory-before-probes.txt"

lspci 2>/dev/null | grep -Ei 'vga|3d|display'      # GPU (for module 07)
systemd-detect-virt                                 # running inside a VM?
```
**Conflicts to flag:** Node from apt *and* from nvm/mise; `docker.io` (Ubuntu) *and* `docker-ce`
(Docker Inc.); snap *and* deb copies of the same app (e.g. `code`); Python packages installed with
`sudo pip`.

## Step 6 — Helpers (paste into the shell)

```bash
log()      { echo "$(date -Is) [$1] $2" | tee -a "$LOG/run.log"; }
have()     { command -v "$1" >/dev/null 2>&1; }
deb_has()  { dpkg-query -W -f='${Status}' "$1" 2>/dev/null | grep -q "install ok installed"; }
apt_install() {   # installs only missing pkgs; upgrades are handled by the full-upgrade in 01
  local miss=(); for p in "$@"; do deb_has "$p" || miss+=("$p"); done
  if ((${#miss[@]} == 0)); then log apt "already present: $*"; return 0; fi
  if sudo DEBIAN_FRONTEND=noninteractive apt-get install -y "${miss[@]}"; then log apt "installed ${miss[*]}"
  else log apt "FAILED ${miss[*]}"; return 1; fi
}
add_apt_repo() {  # add_apt_repo <name> <key_url> "<deb line without signed-by>"
  local name=$1 key=$2 line=$3 kr=/etc/apt/keyrings/$1.gpg
  sudo install -d -m 0755 /etc/apt/keyrings
  if [ ! -f "$kr" ]; then        # handles both ASCII-armored and binary keys
    curl -fsSL "$key" -o "/tmp/$name.key" || return 1
    if grep -q 'BEGIN PGP' "/tmp/$name.key"; then sudo gpg --batch --yes --dearmor -o "$kr" "/tmp/$name.key"
    else sudo install -m 0644 "/tmp/$name.key" "$kr"; fi
  fi
  echo "${line/deb /deb [signed-by=$kr] }" | sudo tee "/etc/apt/sources.list.d/$name.list" >/dev/null
  log repo "added $name"
}
repo_supports() { curl -fsI "$1/dists/$UCODENAME/Release" >/dev/null 2>&1; }   # Rule 4b check
snap_install() { snap list "$1" >/dev/null 2>&1 && log snap "$1 present" || { sudo snap install "$@" && log snap "installed $1"; }; }
```

## Step 7 — Plan → approval

Write `$LOG/plan.md` (item | current | action) for the chosen profiles, show it, and **wait for a yes**.
