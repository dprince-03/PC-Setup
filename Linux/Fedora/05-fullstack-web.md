# 05 — Full-Stack Web (Fedora)

**Profiles:** fullstack

| Tool | Install |
|---|---|
| pnpm/Yarn | `corepack enable` (02) |
| Bun / Deno | `mise use -g bun@latest deno@latest` |
| Chrome | `dnf_install fedora-workstation-repositories && sudo dnf config-manager setopt google-chrome.enabled=1 && dnf_install google-chrome-stable` |
| Firefox | preinstalled (RPM) |
| Playwright browsers | per project: `npx playwright install --with-deps` (Fedora isn't an officially supported Playwright OS, so if `--with-deps` fails, install the deps it lists with dnf) |
| mkcert | `dnf_install mkcert nss-tools && mkcert -install` |
| cloudflared | Cloudflare's RPM repo: <https://pkg.cloudflare.com/> |
| HTTPie | `uv tool install httpie` |
| Nginx / Caddy | `dnf_install nginx` / `dnf_install caddy` |
| Mailpit | `podman run -d --name mailpit -p 127.0.0.1:8025:8025 -p 127.0.0.1:1025:1025 docker.io/axllent/mailpit` |

SELinux and local web servers: serving from `$HOME` with nginx needs `httpd_read_user_content`, or
better, a dev server on a high port. Don't disable SELinux.

Scaffolders run on demand (`pnpm create vite@latest`, `npx create-next-app@latest`).

## Verify

`node -v; pnpm -v; bun -v; deno -v; mkcert -CAROOT`
