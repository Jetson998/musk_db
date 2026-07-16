# musk_db — applied workflow notes

This project follows `GENERIC_PROJECT_DEVELOPMENT_WORKFLOW.md`. Applied facts and
decisions specific to musk_db:

## Project identity

```text
Project name: musk_db
Repository: git@github.com:Jetson998/musk_db.git
Default branch: main
Primary runtime: static HTML + NewAPI official Docker image (no app code of our own)
Production URL: https://db.muskapis.com (NewAPI) + https://home.muskapis.com (landing host, not yet live)
Deployment target: NewAPI official image (calciumion/new-api:latest) + static host for landing (Cloudflare Pages/Vercel)
Owner/operator: Jetson998
```

## Startup confirmation gate (Section 2)

Run before any change:

```sh
pwd                         # /Users/jets2026/cccli/musk_db
git status --short --branch
git remote -v               # origin git@github.com:Jetson998/musk_db.git
git config --get remote.origin.url
```

Stop if path/branch/remote mismatch.

## Local checks (Section 11)

- Landing page is a single HTML file with inlined JS. node cannot `--check` an
  `.html` directly, so extract the `<script>` block first:

  ```sh
  python3 -c 'import re,s=open("output/landing/index.html").read();open("/tmp/m.js","w").write(re.search(r"<script>([\s\S]*?)</script>",s).group(1))'
  node --check /tmp/m.js
  ```

- i18n sanity: count `data-i18n` referenced keys vs `en`/`zh` dictionary entries;
  must be equal with no missing/extra keys.

## Secrets (Section 10)

- The example curl uses a placeholder `sk-musk-...` — that is an example, not a
  secret, and is safe to commit.
- Never commit real upstream keys, NewAPI admin tokens, or `.env`.

## Layout note

- `docs/` holds durable docs (technical design, admin checklist, capability
  inventory, deployment verification).
- `output/` holds the actual artifacts (landing HTML + previews).
- `deployment/releases/` and `deployment/operations/` hold dated records.
