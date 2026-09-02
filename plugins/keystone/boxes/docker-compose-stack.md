# Docker stack: prod compose + dev override
Dated 2026-09-01. Replaces: nothing.

## Decisions
- **`docker-compose.yml` is production; `docker-compose.dev.yml` is applied explicitly with a second -f. Never a file named docker-compose.override.yml.**
  Reason: auto-merged overrides broke production on git pull.
  Silent default: dev command is `docker compose -f docker-compose.yml -f docker-compose.dev.yml up --build`.
  Check: `test ! -e docker-compose.override.yml`.
- **The dev override may set only: bind mounts, environment, dev port publishing (database port per the rule below), and a `command` that is the production command plus flags. It never sets `entrypoint`.**
  Reason: an override that replaced the command re-ran migrations and bypassed the entrypoint.
  Silent default: restate the production command tokens and append `--reload`.
  Check: `python scripts/check_compose.py` (override has no entrypoint key; each overridden command starts with that service's production command tokens — the production compose `command` if set, else the Dockerfile CMD).
- **Python images upgrade pip and wheel before installing requirements; requirements.txt pins exact versions (==).**
  Reason: stale pip breaks wheels; unpinned requirements rebuild into a different app.
  Silent default: `RUN pip install --upgrade pip wheel` then `pip install -r requirements.txt`; pin to what the current image resolves.
  Check: `grep -n "upgrade pip wheel" Dockerfile && ! grep -E ">=|~=" requirements.txt`.
- **Every service has restart: unless-stopped; every service except the database has depends_on with a condition; the app exposes /health and a healthcheck.**
  Reason: services boot in the wrong order otherwise (which process migrates is postgres-alembic's rule).
  Silent default: db → app (service_healthy) → workers; on slim Python images the healthcheck is `python -c` with urllib (no curl in the image).
  Check: `python scripts/check_compose.py`.
- **Scheduled or background work is a service in the compose file, never host cron.**
  Reason: host cron is invisible to the repo.
  Silent default: worker service on the same image.
  Check: review.
- **Named volumes carry the project prefix; the database port is never published in production; the dev override may publish it on 127.0.0.1 only.**
  Reason: two apps on one host.
  Silent default: `<project>_postgres_data`, `<project>_uploads_data`.
  Check: `python scripts/check_compose.py`.
- **Never `docker compose down -v` on the dev or production project. Named volumes are the data.**
  Reason: it deletes uploads and the database.
  Silent default: `docker compose stop`.
  Check: review.
- **A .dockerignore excludes .git, virtualenvs, data directories, screenshots, and anything .gitignore ignores.**
  Reason: `COPY . .` shipped a host venv into an image.
  Silent default: mirror .gitignore plus `.git`.
  Check: `test -f .dockerignore && grep -q "^\.venv" .dockerignore`.
- **The stack is booted by scripts/compose_smoke.sh under its own compose project name (which gives it its own volumes) using the production compose file plus a smoke file only (no dev override: it tests the built image, not the host tree) that clears published ports with `ports: !reset []`, so it coexists with the running dev stack: build, up, wait for app healthy (the healthcheck requires HTTP 200 on the health path itself, a redirect is a failure), assert every service running, then `down -v` for that project only. When it runs is tests-fastapi-postgres's rule. It never touches the dev project's database.**
  Reason: a parsed compose file can pass every static check and never start the worker; the dev project's volumes must stay untouched; Compose merges port lists by union.
  Silent default: project name `<project>_smoke`, file `docker-compose.smoke.yml`.
  Check: `scripts/compose_smoke.sh`.

## Layout
Dockerfile · .dockerignore · start.sh · docker-compose.yml · docker-compose.dev.yml · docker-compose.smoke.yml · .env.example · scripts/check_compose.py (takes --port and --volume-prefix) · scripts/compose_smoke.sh

## Not covered
Which host, port, and deploy path (private overlay). Database rules (postgres-alembic).
