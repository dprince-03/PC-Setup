# 02 — Languages and SDKs (Ubuntu)

**Profiles:** languages · Python, Node.js, Java, Go, C/C++, C#/.NET, plus optional Rust/Kotlin.

Strategy: **uv** for Python, **mise** for Node/Java/Go/Maven/Gradle (latest via `@latest`), the
**distro toolchain** for C/C++, and **Microsoft's packages** for .NET.

---

## Step 0 — mise (official APT repo)

```bash
have mise || {
  add_apt_repo mise https://mise.jdx.dev/gpg-key.pub "deb https://mise.jdx.dev/deb stable main"
  sudo apt update && apt_install mise; }
grep -qxF 'eval "$(mise activate bash)"' ~/.bashrc || echo 'eval "$(mise activate bash)"' >> ~/.bashrc
eval "$(mise activate bash)"
mise --version
```
mise installs into `~/.local/share/mise`. `mise use -g <tool>@latest` writes `~/.config/mise/config.toml`.
`mise upgrade` keeps tools current later.

## Python

```bash
# System python3 belongs to the OS. Leave it alone. Add venv support + headers for native builds:
apt_install python3 python3-venv python3-dev python3-pip pipx
# uv: official installer, saved to a file first (or: pipx install uv)
have uv || { curl -LsSf https://astral.sh/uv/install.sh -o /tmp/uv.sh && sh /tmp/uv.sh; }
source ~/.local/bin/env 2>/dev/null || export PATH="$HOME/.local/bin:$PATH"
uv python install            # newest stable CPython, managed by uv
uv python list --only-installed
uv tool install ruff; uv tool install pre-commit
```
Per project: `uv init`, `uv add <pkg>`, `uv run ...`. **Never `sudo pip install`.** (Ubuntu also
blocks it by default: PEP 668.)

## Node.js

```bash
mise use -g node@lts          # newest LTS (default per AGENT.md). For the newest "current": node@latest
corepack enable               # pnpm/yarn per project
npm install -g npm@latest
node -v && npm -v
```
If Node is already installed via apt/NodeSource/nvm → CONFLICT. Report it and ask before removing
the old one, because two `node` binaries on PATH cause hard-to-debug problems.

## Java (+ Maven, Gradle, Kotlin)

```bash
mise ls-remote java | grep -E '^temurin-[0-9]+' | sed 's/-.*//' | sort -u | tail -3   # see the available Temurin majors
# newest LTS: a major N >= 17 is LTS when (N-17) % 4 == 0  (17, 21, 25, 29 ...)
LTS=$(mise ls-remote java | grep -oE '^temurin-[0-9]+' | cut -d- -f2 | sort -n -u | awk '$1>=17 && ($1-17)%4==0' | tail -1)
mise use -g "java@temurin-$LTS"
mise use -g maven@latest gradle@latest
# optional: mise use -g kotlin@latest
# make JAVA_HOME available to IDEs/tools (mise sets it inside activated shells):
java -version && mvn -v | head -1 && gradle -v | grep Gradle
```
Alternative if the user prefers system packages: `sudo apt install openjdk-<N>-jdk` (the newest
`<N>` in `apt-cache search openjdk | grep -oE 'openjdk-[0-9]+-jdk ' | sort -V`) plus `maven`.
The apt Maven/Gradle are often old, so prefer mise for those.

## Go

```bash
mise use -g go@latest
go version
go install golang.org/x/tools/gopls@latest
go install github.com/go-delve/delve/cmd/dlv@latest
grep -qxF 'export PATH="$HOME/go/bin:$PATH"' ~/.bashrc || echo 'export PATH="$HOME/go/bin:$PATH"' >> ~/.bashrc
```

## C / C++

```bash
apt_install build-essential gcc g++ gdb clang clangd clang-format clang-tidy lldb lld \
  cmake ninja-build meson autoconf automake libtool valgrind ccache
gcc --version | head -1; clang --version | head -1; cmake --version | head -1
```
The newest LLVM (if newer than Ubuntu's) comes from LLVM's official script, so ask first:
`wget https://apt.llvm.org/llvm.sh -O /tmp/llvm.sh && sudo bash /tmp/llvm.sh` (installs the latest stable).
Newer CMake than the distro has: `mise use -g cmake@latest`.
vcpkg (optional): `git clone https://github.com/microsoft/vcpkg ~/vcpkg && ~/vcpkg/bootstrap-vcpkg.sh`.

## C# / .NET

Recent Ubuntu releases ship .NET SDKs in the **Ubuntu archive** (built by Canonical, supported by
Microsoft):
```bash
apt-cache search --names-only '^dotnet-sdk-[0-9.]+$' | sort -V     # see what's available
SDK=$(apt-cache search --names-only '^dotnet-sdk-[0-9.]+$' | awk '{print $1}' | sort -V | tail -1)
apt_install "$SDK"
dotnet --list-sdks
dotnet tool install -g dotnet-ef 2>/dev/null; grep -q '.dotnet/tools' ~/.bashrc || echo 'export PATH="$HOME/.dotnet/tools:$PATH"' >> ~/.bashrc
```
If the newest .NET isn't in the archive yet, use Microsoft's script (user-local, no root):
`curl -fsSL https://dot.net/v1/dotnet-install.sh -o /tmp/dotnet-install.sh && bash /tmp/dotnet-install.sh --channel STS` (or `--channel LTS`).
Ask before mixing it with the apt SDK.

## Rust (optional)

```bash
have rustup || { curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs -o /tmp/rustup.sh && sh /tmp/rustup.sh -y; }
source "$HOME/.cargo/env"; rustup update stable; rustup component add rust-analyzer clippy rustfmt
```

## Verify

```bash
for c in "python3 --version" "uv --version" "node -v" "pnpm -v" "java -version" "mvn -v" "gradle -v" \
         "go version" "gcc --version" "clang --version" "cmake --version" "dotnet --version" "rustc --version"; do
  printf '%-18s %s\n' "${c%% *}" "$($c 2>&1 | head -1)"; done | tee -a "$LOG/run.log"
```
