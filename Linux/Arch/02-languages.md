# 02 — Languages and SDKs (Arch)

**Profiles:** languages · Arch's repos already carry the newest stable versions, so use pacman
for system toolchains and **mise** where projects need LTS or several versions.

```bash
pac mise uv
grep -qxF 'eval "$(mise activate bash)"' ~/.bashrc || echo 'eval "$(mise activate bash)"' >> ~/.bashrc
eval "$(mise activate bash)"
```

| Language | Install | Notes |
|---|---|---|
| **Python** | `pac python python-pip python-pipx` + `uv python install` | never `sudo pip`; projects use `uv` |
| **Node.js** | `mise use -g node@lts` (default) → `corepack enable` | pacman's `nodejs` is the *current* release; `nodejs-lts-*` also exists. Don't mix pacman Node with mise Node |
| **Java** | `mise use -g java@temurin-<newest LTS>` (see Ubuntu 02 for the LTS one-liner), `mise use -g maven@latest gradle@latest` | pacman alternative: `pac jdk-openjdk maven gradle` + `archlinux-java status` / `sudo archlinux-java set <env>` |
| **Go** | `pac go` (newest) | `go install golang.org/x/tools/gopls@latest`; `pac delve` |
| **C/C++** | `pac gcc clang lld lldb gdb cmake ninja meson valgrind ccache autoconf automake` | newest GCC/LLVM |
| **C#/.NET** | `pac dotnet-sdk aspnet-runtime` (newest) | older/LTS SDKs: `aur dotnet-sdk-<N>-bin`, or `mise use -g dotnet@<N>`. Ask |
| **Rust** | `pac rustup && rustup default stable && rustup component add rust-analyzer clippy rustfmt` | don't also install the `rust` package (it conflicts) |
| Kotlin (optional) | `pac kotlin` | |

```bash
pac python python-pip python-pipx go gcc clang lld lldb gdb cmake ninja meson valgrind ccache dotnet-sdk rustup
uv python install && uv tool install ruff && uv tool install pre-commit
mise use -g node@lts && corepack enable
LTS=$(mise ls-remote java | grep -oE '^temurin-[0-9]+' | cut -d- -f2 | sort -n -u | awk '$1>=17 && ($1-17)%4==0' | tail -1)
mise use -g "java@temurin-$LTS" maven@latest gradle@latest
rustup default stable
grep -qxF 'export PATH="$HOME/go/bin:$HOME/.dotnet/tools:$PATH"' ~/.bashrc || echo 'export PATH="$HOME/go/bin:$HOME/.dotnet/tools:$PATH"' >> ~/.bashrc
```

## Verify

Same verify loop as Ubuntu 02 → `$LOG/run.log`.
