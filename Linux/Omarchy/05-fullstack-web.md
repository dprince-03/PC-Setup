# 05 — Full-Stack Web (Omarchy)

**Profiles:** fullstack · Chromium is the default browser.

```bash
omarchy install dev-env bun; omarchy install dev-env deno
corepack enable
omarchy-install-browser 2>/dev/null   # optional menu for other browsers (Firefox, Brave, Chrome...)
pkg mkcert nss && mkcert -install
pkg cloudflared httpie
```
Rails/Laravel/Phoenix shortcuts: `omarchy install dev-env ruby` (Rails), `laravel`, `phoenix`.
The rest is the same as `Linux/Arch/05-fullstack-web.md`.

## Verify

`node -v; bun -v; deno -v; mkcert -CAROOT`
