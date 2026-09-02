# Test harness on a real database
Dated 2026-09-01. Replaces: nothing.

## Decisions
- **Tests run against a real PostgreSQL database named by TEST_DATABASE_URL, created and migrated by scripts/testdb.py, dropped at the end of check.sh. No stubbing of the ORM, the framework, or the templates.**
  Reason: stubbed tests let a NOT NULL drift reach production.
  Silent default: testdb.py imports the app's config module (which loads .env) for the database user and password and builds `postgresql://<user>:<pw>@127.0.0.1:<dev port>/<project>_test`; conftest exports DATABASE_URL and the test-only settings before importing the app.
  Check: `pytest -q` from a clean database.
- **A live-server fixture (tests/server.py) starts the app on the test port against the test database for browser tests and the render script; browser tests are ordinary tests, not integration-marked.**
  Reason: a browser test with no server is a silent skip.
  Silent default: port from TEST_APP_PORT.
  Check: `pytest tests/test_quiz_flows.py`.
- **One seed module (tests/seed.py) builds the fixture world used by pytest, the render script, and the integration test. Every consumer starts from a clean world: a function-scoped fixture truncates every table with RESTART IDENTITY and reseeds; the render script and the integration test reseed the same way before they start.**
  Reason: three seeders drift; stages that share one database mutate it (a rendered exam page inserts a session) and later assertions read stale state.
  Silent default: `reset_and_seed(db) -> ids`.
  Check: `pytest tests/test_seed.py` (two consecutive seeds yield identical ids).
- **Route tests use TestClient with a minted session cookie (tests/cookies.py, same signer as the session middleware); the minted session carries a known CSRF token and the client fixture sends it on every mutating request; ownership tests use two users; browser tests set the same cookie and read the page's token.**
  Reason: auth and ownership are SQL, not Python; every POST in every test needs the token once CSRF lands.
  Silent default: fixtures `student`, `other_student`, `admin`, each a TestClient with cookie and header preset.
  Check: `pytest tests/test_route_auth.py`.
- **External services are faked at their one module boundary (llm replay), never by patching call sites; the test environment sets every external API key to an invalid value so a call that escapes the fake fails fast instead of spending.**
  Reason: mocks at call sites drift from the real signature; a loaded .env would otherwise supply the real key.
  Silent default: LLM_MODE=replay; `<SERVICE>_API_KEY=invalid-test-key`.
  Check: `grep -rnE "patch\(\"app\.(pipeline|prepare|ai)" tests/ | wc -l` is 0.
- **The host venv runs the same Python minor version as the image, created with uv; pyproject.toml declares `requires-python` and ruff `target-version` for that minor; dev tooling is pinned in requirements-dev.txt; ruff selects E, F, I, B, UP at line-length 100, E501 ignored, and any per-file-ignores entry carries a comment saying why.**
  Reason: a 3.9 venv could not import a 3.12 codebase; pytest was installed but declared nowhere; "fix or ignore" invites ignore.
  Silent default: `uv venv --python <image minor> .venv && uv pip install -r requirements.txt -r requirements-dev.txt && playwright install chromium`; `[tool.ruff.lint.flake8-bugbear] extend-immutable-calls` lists the framework's dependency callables (`fastapi.Depends`, `Form`, `File`, `Query`, `Header`, `Body`, `Path`) so B008 does not fire on route signatures; lint config grows only on an observed failure recorded in the commit body.
  Check: `.venv/bin/python --version` matches the Dockerfile base image minor; `ruff check . && ruff format --check .`.
- **A test never imports a copy of production code.**
  Reason: a pasted function was tested instead of the real one.
  Silent default: fix the import chain, don't copy.
  Check: review.
- **Headless render (scripts/render_routes.py): start the app on a test port against the seeded test database, load each listed route at 1280px and 375px, save screenshots, fail on HTTP ≥ 400, `pageerror`, `console.error`, or a CSP violation (an `add_init_script` listener on `securitypolicyviolation` collects them into `window.__cspViolations`, read after load). Aborted or blocked network requests are not errors. Network to listed CDN hosts is allowed. `--allow-errors --record-baseline <file>` records errors to a committed file instead of failing; `--baseline <file>` fails only on errors not in that file. Error identity is route path plus message with runs of digits and hex ids replaced by `#`. `--mobile-check` adds the 375px width rule from frontend-css-js.**
  Reason: pages that render but throw are the common regression; a consolidation inherits errors it did not cause; ids in messages change every run; screenshots are ignored but the baseline must survive a fresh checkout.
  Silent default: analytics unset in the test env; baseline file `tests/baseline_errors.json`.
  Check: `python scripts/render_routes.py --out screenshots/<label>`.
- **check.sh runs, in loop.md order: (`--full` only: compose smoke, per docker-compose-stack) → create test db → tests → lint and format → static checks (`scripts/check_greps.sh`, which holds every grep invariant the boxes and the spec name, each with `--include` filters for py/html/js and `--exclude-dir=__pycache__`; the template, size, env, compose checks; `alembic check` against the test db) → headless render → evals → integration test → drop test db. It exports `PYTHONDONTWRITEBYTECODE=1`. Exit 0 or no commit. It always runs `up -d db` with the dev compose pair (which recreates the container when its config changed) and never migrates the dev database itself. Until a project's first checklist item fills the stages, the stub exits 0 and prints which stages are missing.**
  Reason: loop.md; a stub that exits 1 blocks the first commit; grep invariants checked once at their item and never again get reintroduced by later refactors; stale bytecode makes unfiltered greps print binary matches.
  Silent default: wait for pg_isready before creating the test db; `UPLOAD_ROOT` is a temp dir for the run.
  Check: `./check.sh`.

## Layout
tests/conftest.py · tests/seed.py · tests/cookies.py · tests/server.py · tests/test_<area>.py · tests/test_integration.py · scripts/testdb.py · scripts/render_routes.py · scripts/check_greps.sh · screenshots/ (gitignored)

## Not covered
What the integration walk-through does (the spec).
