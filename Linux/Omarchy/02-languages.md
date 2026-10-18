# 02 — Languages (Omarchy)

**Profiles:** languages · Omarchy installs languages through **mise** with `@latest`, using
`omarchy install dev-env <lang>`. Supported: `ruby node bun deno go laravel symfony php python elixir
phoenix rust java zig ocaml dotnet clojure scala`.

```bash
mise ls                                      # what's already there
omarchy install dev-env python               # mise python@latest + uv
omarchy install dev-env node                 # mise node (newest)
omarchy install dev-env java                 # mise java@latest
omarchy install dev-env go
omarchy install dev-env dotnet               # mise dotnet@latest
omarchy install dev-env rust                 # optional (rustup + clang)
```

**LTS adjustment (AGENT.md Rule 4).** Omarchy picks the newest releases. If the user wants LTS
(the default for Node/Java in these playbooks), switch after installing:
```bash
mise use -g node@lts
LTS=$(mise ls-remote java | grep -oE '^temurin-[0-9]+' | cut -d- -f2 | sort -n -u | awk '$1>=17 && ($1-17)%4==0' | tail -1)
mise use -g "java@temurin-$LTS"
```

**Build tools and C/C++** (not covered by dev-env):
```bash
mise use -g maven@latest gradle@latest
corepack enable
pkg gcc clang lld lldb gdb cmake ninja meson valgrind ccache
uv tool install ruff; uv tool install pre-commit
go install golang.org/x/tools/gopls@latest
```

## Verify

Same verify loop as Ubuntu 02.
