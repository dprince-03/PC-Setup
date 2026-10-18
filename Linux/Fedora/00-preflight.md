# 00 — Preflight (Fedora)

## Step 1 — Confirm the OS

```bash
. /etc/os-release
echo "$PRETTY_NAME | ID=$ID | VERSION_ID=$VERSION_ID | VARIANT=${VARIANT_ID:-}"
uname -srm; rpm -E %fedora; dnf --version | head -1
[ -e /run/ostree-booted ] && echo "ATOMIC (Silverblue/Kinoite...): see README rules"
IS_WSL=$(grep -qi microsoft /proc/version && echo 1 || echo 0); echo "WSL=$IS_WSL"
getenforce                                   # should be Enforcing: keep it that way
findmnt -no FSTYPE /                         # btrfs (default) or other
mokutil --sb-state 2>/dev/null               # Secure Boot (matters for NVIDIA akmods)
df -h / | tail -1; free -h | awk '/Mem/{print "RAM:",$2}'
```
`ID` must be `fedora`, otherwise **STOP**. Warn if `VERSION_ID` is older than the previous release (EOL).

## Step 2 — sudo, logs, snapshot

```bash
sudo -v
export LOG="$HOME/pc-setup-logs/$(date +%Y%m%d-%H%M%S)"; mkdir -p "$LOG"
echo "$(date -Is) [00] $PRETTY_NAME $(uname -r) WSL=$IS_WSL" | tee "$LOG/os.txt"
# Snapshot on btrfs with snapper (ask before configuring it the first time)
if [ "$(findmnt -no FSTYPE /)" = btrfs ]; then
  have_snapper=$(command -v snapper)
  [ -z "$have_snapper" ] && echo "Ask: sudo dnf install -y snapper && sudo snapper -c root create-config /"
  sudo snapper -c root create -d "PC-Setup before changes" 2>/dev/null && echo "snapshot created"
fi
```
On non-btrfs systems: tell the user there's no snapshot and ask whether to continue.

## Step 3 — Inventory

```bash
rpm -qa --qf '%{NAME} %{VERSION}-%{RELEASE}\n' | sort > "$LOG/inventory-before-rpm.txt"
dnf repolist --enabled > "$LOG/repos-before.txt"
flatpak list --app 2>/dev/null > "$LOG/inventory-before-flatpak.txt"
probe() { local n=$1; shift; if command -v "$1" >/dev/null 2>&1; then
  printf '%-12s %-45s %s\n' "$n" "$("$@" 2>&1 | head -1)" "$(command -v "$1")"; else printf '%-12s MISSING\n' "$n"; fi; }
{
probe git git --version; probe gh gh --version; probe mise mise --version; probe python3 python3 --version
probe uv uv --version; probe node node -v; probe pnpm pnpm -v; probe java java -version; probe mvn mvn -v
probe gradle gradle -v; probe go go version; probe gcc gcc --version; probe clang clang --version
probe cmake cmake --version; probe dotnet dotnet --version; probe rustc rustc --version
probe docker docker --version; probe podman podman --version; probe kubectl kubectl version --client
probe helm helm version --short; probe terraform terraform -version; probe ansible ansible --version
probe aws aws --version; probe az az version; probe gcloud gcloud --version; probe psql psql --version
probe mysql mysql --version; probe mongosh mongosh --version; probe valkey-cli valkey-cli --version
probe redis-cli redis-cli --version; probe code code --version; probe flutter flutter --version
probe adb adb version; probe nvidia-smi nvidia-smi -L
} | tee "$LOG/inventory-before-probes.txt"
lspci | grep -Ei 'vga|3d|display'
```
Conflicts to flag: `podman-docker` *and* `docker-ce`; Node from dnf *and* from mise/nvm; Flatpak
*and* RPM copies of the same app.

## Step 4 — Helpers

```bash
log()  { echo "$(date -Is) [$1] $2" | tee -a "$LOG/run.log"; }
have() { command -v "$1" >/dev/null 2>&1; }
dnf_install() {
  local miss=(); for p in "$@"; do rpm -q "$p" >/dev/null 2>&1 || miss+=("$p"); done
  if ((${#miss[@]} == 0)); then log dnf "already present: $*"; return 0; fi
  if sudo dnf install -y "${miss[@]}"; then log dnf "installed ${miss[*]}"; else log dnf "FAILED ${miss[*]}"; return 1; fi
}
add_repo_file() { # add_repo_file <url-to-.repo>
  local f="/etc/yum.repos.d/$(basename "$1")"
  [ -f "$f" ] && { log repo "exists $f"; return 0; }
  sudo dnf config-manager addrepo --from-repofile="$1" && log repo "added $1"
}
flat() { flatpak info "$1" >/dev/null 2>&1 && log flatpak "$1 present" || { flatpak install -y flathub "$1" && log flatpak "installed $1"; }; }
```

## Step 5 — Plan → approval

Write `$LOG/plan.md`, show it, and **wait for a yes**.
