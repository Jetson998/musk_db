Date: 2026-07-16
Operator: Jetson998
Change name: Initial repository bootstrap for musk_db
Reason: Stand up project on GitHub remote per GENERIC_PROJECT_DEVELOPMENT_WORKFLOW; bring existing landing + docs under version control.
Affected files/services:
  - README.md (new)
  - .gitignore (new)
  - docs/ (musk-首页技术方案.md, musk-后台配置清单.md, newapi-能力清单与开发清单.md, deployment-verification.md)
  - deployment/releases/README.md, deployment/operations/README.md (new)
  - output/landing/index.html (bilingual landing page — embedded into NewAPI via iframe HomePageContent)
  - output/landing-preview.html, output/landing-preview-dark.html (local previews)
  - output/musk-home.html, output/musk-home-preview*.html (legacy no-JS static fallback + previews)
Expected behavior:
  - git init on main, remote origin = git@github.com:Jetson998/musk_db.git
  - first commit pushed; worktree clean
Local checks:
  - security scan: `rg -n "sk-[A-Za-z0-9]|_API_KEY|BEGIN .* PRIVATE KEY" .` → only placeholder `sk-musk-...` example, no real secrets
  - landing JS: extract `<script>` from output/landing/index.html → `node --check` → PASS
  - i18n keys: 144 referenced == 144 defined in en == 144 in zh (verified)
Production verification:
  - not yet deployed; production = NewAPI official image + landing static host
Rollback:
  - n/a (initial commit)
Commit: (filled after commit)
Deployment result: not deployed this step (repo bootstrap only)
Backup paths/artifacts: n/a
Open risks:
  - git global email uses noreply placeholder (Jetson998@users.noreply.github.com); replace with real GitHub email when available
  - production URL/landing URL not yet live; config checklist still references https://db.muskapis.com
