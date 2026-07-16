# Deployment Verification — musk_db

Project-specific smoke checks (workflow Section 14). Replace the endpoints with
the real production values before relying on them.

## NewAPI gateway (db.muskapis.com)

```sh
curl -I https://db.muskapis.com
curl -s https://db.muskapis.com/api/status | grep -o '"success":\s*true'
```

Minimum coverage:
- service reachable (200 / 302);
- `/api/status` returns `success: true`;
- login page loads (`/login`);
- a real token call succeeds and logs to console;
- landing iframe loads inside NewAPI home;
- top-bar language toggle flips the iframe content (EN ↔ 中文);
- top-bar theme toggle flips the iframe (light ↔ dark).

## Landing page host (home.muskapis.com)

```sh
curl -I https://home.muskapis.com/
curl -s https://home.muskapis.com/ | grep -c 'data-i18n'
```

Minimum coverage:
- served over HTTPS, 200;
- HTML contains `data-i18n` nodes (i18n wired);
- local JS syntax check passes (`node --check` on extracted `<script>`).

## Rollback path

- Landing: revert `HomePageContent` field in NewAPI admin to the legacy static
  HTML (or empty) — no NewAPI image change needed.
- NewAPI: `docker-compose down && docker-compose up -d` with the previous image
  tag pinned in `docker-compose.yml`.
