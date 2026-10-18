# 02 — Languages and SDKs (Fedora)

**Profiles:** languages · Strategy: **uv** (Python), **mise** (Node/Java/Go/Maven/Gradle),
**dnf** (C/C++, .NET).

## mise (official RPM repo)

```bash
have mise || { add_repo_file https://mise.jdx.dev/rpm/mise.repo && dnf_install mise; }
grep -qxF 'eval "$(mise activate bash)"' ~/.bashrc || echo 'eval "$(mise activate bash)"' >> ~/.bashrc
eval "$(mise activate bash)"
```

## Python

```bash
dnf_install python3 python3-devel python3-pip pipx
have uv || { curl -LsSf https://astral.sh/uv/install.sh -o /tmp/uv.sh && sh /tmp/uv.sh; }   # or: dnf_install uv
export PATH="$HOME/.local/bin:$PATH"
uv python install && uv tool install ruff && uv tool install pre-commit
```
Fedora also ships many Python versions as `python3.X` packages. Not needed with uv.

## Node.js

```bash
mise use -g node@lts        # newest LTS (default); node@latest for "current"
corepack enable && npm i -g npm@latest
```

## Java + Maven + Gradle

```bash
LTS=$(mise ls-remote java | grep -oE '^temurin-[0-9]+' | cut -d- -f2 | sort -n -u | awk '$1>=17 && ($1-17)%4==0' | tail -1)
mise use -g "java@temurin-$LTS" maven@latest gradle@latest
java -version
```
Distro alternative: `dnf_install java-latest-openjdk-devel` (newest feature release), or
`java-<N>-openjdk-devel` (LTS), with `alternatives --config java` to switch.

## Go

```bash
mise use -g go@latest      # or dnf_install golang (Fedora tracks the latest Go closely)
go install golang.org/x/tools/gopls@latest; go install github.com/go-delve/delve/cmd/dlv@latest
grep -qxF 'export PATH="$HOME/go/bin:$PATH"' ~/.bashrc || echo 'export PATH="$HOME/go/bin:$PATH"' >> ~/.bashrc
```

## C / C++

```bash
sudo dnf group install -y c-development development-tools
dnf_install gcc gcc-c++ gdb clang clang-tools-extra lldb lld cmake ninja-build meson valgrind ccache \
  autoconf automake libtool
```
Fedora usually ships the newest GCC and LLVM within weeks of their release.

## C# / .NET

```bash
SDK=$(dnf list --available 'dotnet-sdk-*' 2>/dev/null | grep -oE '^dotnet-sdk-[0-9]+\.[0-9]+' | sort -V | uniq | tail -1)
dnf_install "$SDK"
dotnet --list-sdks
dotnet tool install -g dotnet-ef 2>/dev/null
grep -q '.dotnet/tools' ~/.bashrc || echo 'export PATH="$HOME/.dotnet/tools:$PATH"' >> ~/.bashrc
```
Newest STS/preview not packaged yet → `dotnet-install.sh` (see Ubuntu 02). Ask first.

## Rust (optional)

`dnf_install rustup && rustup-init -y` (or the official rustup script), then
`rustup component add rust-analyzer clippy rustfmt`.

## Verify

Run the same verify loop as Ubuntu 02 and append the output to `$LOG/run.log`.
