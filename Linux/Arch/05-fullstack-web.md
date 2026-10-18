# 05 — Full-Stack Web (Arch)

**Profiles:** fullstack

| Tool | Install |
|---|---|
| pnpm / Yarn | `corepack enable` (or `pac pnpm`). Pick one |
| Bun / Deno | `pac bun deno` (or `mise use -g bun@latest deno@latest`) |
| Chrome / Chromium / Firefox | `aur google-chrome` / `pac chromium` / `pac firefox` |
| Playwright browsers | `npx playwright install` (Arch isn't officially supported; skip `--with-deps` and install the missing libs with pacman) |
| mkcert | `pac mkcert nss && mkcert -install` |
| cloudflared | `pac cloudflared` |
| HTTPie | `pac httpie` |
| Nginx / Caddy | `pac nginx` / `pac caddy` |
| Mailpit | Docker (see Ubuntu 05) |

## Verify

`node -v; pnpm -v; bun -v; deno -v`
