# 05 — Full-Stack Web (Ubuntu)

**Profiles:** fullstack · Depends on 02 (Node via mise, Python via uv), 03, 04.

| Tool | Install |
|---|---|
| pnpm / Yarn | `corepack enable` (done in 02) |
| Bun | `mise use -g bun@latest` |
| Deno | `mise use -g deno@latest` |
| Chrome | official .deb + repo: `wget -O /tmp/chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb && sudo apt install -y /tmp/chrome.deb` (amd64 only) |
| Firefox | preinstalled (snap). Developer Edition: Mozilla's APT repo (`firefox-devedition`) |
| Chromium for tests | per project: `npx playwright install --with-deps` |
| mkcert | `apt_install libnss3-tools mkcert && mkcert -install` |
| cloudflared | Cloudflare's APT repo: <https://pkg.cloudflare.com/> |
| ngrok (optional) | official APT repo: <https://ngrok.com/download/linux> |
| HTTPie | `uv tool install httpie` |
| Nginx / Caddy (local reverse proxy, optional) | `apt_install nginx` / Caddy's official repo |
| Mailpit (mail testing) | `docker run -d --name mailpit -p 127.0.0.1:8025:8025 -p 127.0.0.1:1025:1025 axllent/mailpit` |

Scaffolders run on demand: `pnpm create vite@latest`, `npx create-next-app@latest`,
`npm create astro@latest`. Don't install them globally.
Optional globals (ask): `npm i -g npm-check-updates vercel netlify-cli @nestjs/cli`.

## Verify

```bash
node -v; pnpm -v; bun -v; deno -v; mkcert -CAROOT; google-chrome --version 2>/dev/null
```
