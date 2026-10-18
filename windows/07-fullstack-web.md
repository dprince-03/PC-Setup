# 07 — Full-Stack Web Development (Windows)

**Profiles:** fullstack · Depends on: 04 (Node, Python, Java/Go/.NET backends), 05 (databases),
06 (editors), optionally 11 (Docker)

---

## JS runtimes and package managers

| Tool | Install | Notes |
|---|---|---|
| pnpm / Yarn | `corepack enable` (done in 04) | follows each project's `packageManager` field |
| pnpm standalone (optional) | `Install-App 'pnpm.pnpm'` | only if corepack isn't wanted |
| Bun | `Install-App 'Oven-sh.Bun'` | runtime + bundler + test runner |
| Deno | `Install-App 'DenoLand.Deno'` | |

## Global CLIs (ask; per-project `npx`/`pnpm dlx` is usually better)

```powershell
npm i -g npm-check-updates vercel netlify-cli @nestjs/cli
# Angular CLI only if asked: npm i -g @angular/cli
```
Framework scaffolders (Next.js, Vite, Astro, SvelteKit, Nuxt, Remix/React Router) run on demand:
`pnpm create vite@latest`, `npx create-next-app@latest`. **Don't install them globally.**

## Browsers for testing

| Browser | winget ID |
|---|---|
| Google Chrome | `Google.Chrome` |
| Mozilla Firefox (+ Developer Edition, optional) | `Mozilla.Firefox`, `Mozilla.Firefox.DeveloperEdition` |
| Brave (optional) | `Brave.Brave` |
| Edge | preinstalled; never remove it |

End-to-end testing browsers come per project: `npx playwright install --with-deps`.

## Local HTTPS, tunnels, certificates

| Tool | Install | Use |
|---|---|---|
| mkcert | `Install-App 'FiloSottile.mkcert'` then `mkcert -install` | trusted local HTTPS certs |
| cloudflared | `Install-App 'Cloudflare.cloudflared'` | Cloudflare Tunnels |
| ngrok (optional) | `winget search ngrok` → official `ngrok.ngrok` | public tunnels (needs an account) |

## Backend helpers

- Python web: `uv tool install httpie`, `uv tool install pre-commit`
- Java/Spring: Spring Boot CLI via `mise use -g spring-boot@latest` (optional)
- .NET: `dotnet dev-certs https --trust`
- Go: `go install github.com/air-verse/air@latest` (hot reload, optional)
- Mail testing: Mailpit via Docker: `docker run -d --name mailpit -p 127.0.0.1:8025:8025 -p 127.0.0.1:1025:1025 axllent/mailpit`

## Verify

```powershell
node -v; pnpm -v; bun -v; deno -v; mkcert -CAROOT; cloudflared --version
```
