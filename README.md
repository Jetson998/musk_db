# musk_db

A Dubai-deployed AI API gateway built on [NewAPI](https://github.com/QuantumNous/new-api),
plus a bilingual (EN/中文) marketing landing page embedded into NewAPI via iframe.

## What's in this repo

| Path | What |
|---|---|
| `output/landing/index.html` | **Musk landing page** — standalone bilingual home, embedded into NewAPI via `HomePageContent` = this URL |
| `output/landing-preview.html` / `landing-preview-dark.html` | Local light/dark previews of the landing page |
| `output/musk-home.html` | Legacy no-JS static HTML fallback (paste into `HomePageContent` if not using iframe) |
| `output/musk-home-preview*.html` | Previews of the legacy version |
| `docs/musk-首页技术方案.md` | Technical design doc (rendering path, constraints, upgrade path) |
| `docs/musk-后台配置清单.md` | NewAPI admin config checklist (site / billing / channels / redemption / security) |
| `docs/newapi-能力清单与开发清单.md` | NewAPI built-in capability inventory + dev backlog |
| `docs/deployment-verification.md`, `docs/workflow-applied.md` | Applied workflow + smoke checks |
| `deployment/` | Release & operation records |

## Identity

```text
Project name: musk_db
Repository: git@github.com:Jetson998/musk_db.git
Default branch: main
Primary runtime: none (static HTML + NewAPI official Docker image)
Production URL: https://db.muskapis.com
Deployment target: NewAPI (official image) + static host for landing (Cloudflare Pages/Vercel)
Owner/operator: Jetson998
```

## Architecture (one-line)

NewAPI official Docker image (`calciumion/new-api`) serves the gateway at
`db.muskapis.com`; `HomePageContent` is set to the landing page URL, so NewAPI's
top bar (theme + language switch + auth) renders, with the landing page inside
an iframe that follows theme/language via `postMessage`.

## How to run / preview

- Landing preview: open `output/landing-preview.html` (or `-dark`) in a browser.
- NewAPI: follow `output/musk-后台配置清单.md` (pull `calciumion/new-api:latest`,
  docker-compose up).

## Development standard

This project follows `GENERIC_PROJECT_DEVELOPMENT_WORKFLOW.md`. See `docs/`
for the applied version.

- Do not commit secrets (scan: `rg -n "sk-[A-Za-z0-9]|_API_KEY|BEGIN .* PRIVATE KEY" .`).
- Diagnose read-only first; represent changes in repo before touching production.
- Local check for landing JS: extract `<script>` then `node --check`.
