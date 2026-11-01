# Agent notes

## What this repo is

A single static page: `index.html`. No build step, no package manager, no backend,
no database and no credentials. Tailwind, Google Fonts and Lucide load from public
CDNs at runtime (so the page needs internet access to render styled).

The "Encargos" contact form is client-side only (`handleFormSubmit` just shows a
confirmation message); it posts nowhere.

## Running it locally

```bash
docker compose -f docker-compose.base44.yml up -d   # http://localhost:3000
docker compose -f docker-compose.base44.yml logs -f web
```

`nginx:alpine` serves the repository root read-only on port 3000 with
`Cache-Control: no-store`, so edits to `index.html` show on refresh — no rebuild
or restart needed.

### Quirk: workers run as root

The sandbox checkout directory is mode `0700`, so nginx's default unprivileged
worker cannot traverse the bind mount and every request returns `403`
(`stat() ... failed (13: Permission denied)` in the logs). `.base44/nginx.conf` is
mounted as the **main** nginx config and sets `user root;`. Dotfiles are denied so
`.git` is not exposed.

## Verifying

```bash
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:3000/   # 200, ~109 KB
curl -s http://localhost:3000/ | grep -c ASTROALQUIMIA            # 1
```

Compose healthcheck does the same check, so `docker compose ps` showing `healthy`
means the page is actually being served.
