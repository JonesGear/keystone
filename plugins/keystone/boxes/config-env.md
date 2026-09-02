# Configuration and secrets
Dated 2026-09-01. Replaces: nothing.

## Decisions
- **All environment reads happen in app/config.py, which loads .env first (override=False) so host tooling and containers see the same settings; modules import names from it.**
  Reason: 14 scattered os.getenv calls, two undocumented variables, and nothing loaded .env on the host.
  Silent default: `from app.config import SETTING`; exported variables win over .env; a full connection URL setting, when set, wins over its component settings.
  Check: `grep -rn "os.getenv\|os.environ" app/ | grep -v config.py | wc -l` is 0.
- **Required settings fail at import with the variable name; optional settings have a documented default; an empty string counts as unset. Required: the session secret, database credentials or URL, the admin email, the auth provider's client credentials. Optional with documented behavior: API keys for external services (a call without the key raises that module's error), feature tokens (unset disables the feature), analytics, timezone, tuning values.**
  Reason: database credentials silently defaulted to a weak value; a host running tests in replay mode has no API key; a feature token missing must disable, not crash; a test can only export "" over a loaded .env value.
  Silent default: as stated.
  Check: `pytest tests/test_config.py::test_required_fails_fast tests/test_config.py::test_optional_defaults tests/test_config.py::test_empty_is_unset`.
- **.env.example lists every variable config.py reads, with a comment and a placeholder.**
  Reason: two variables read but never documented.
  Silent default: add the line in the same commit as the read.
  Check: `python scripts/check_env_example.py`.
- **Identifiers that differ per deployment (analytics ids, hosts, ports, database URL) come from config, never a template literal; a snippet renders only when its id is set.**
  Reason: a staging deploy reporting into production analytics.
  Silent default: unset locally.
  Check: `grep -rnE "data-[a-z-]*id=\"[0-9a-f-]{36}\"" app/templates | wc -l` is 0 (no UUID literals in data attributes).
- **.env is gitignored; .env.example is committed; nothing under .claude/ is committed.**
  Reason: secrets and personal tool settings.
  Silent default: as stated.
  Check: `git ls-files .env .claude | wc -l` is 0.

## Layout
app/config.py · .env.example · scripts/check_env_example.py

## Not covered
Docker env wiring (docker-compose-stack).
