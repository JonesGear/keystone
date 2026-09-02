# Google OAuth, allow-list, sessions, CSRF
Dated 2026-09-01. Replaces: nothing.

## Decisions
- **Login is Google OAuth via authlib. An email not in the acl table is rejected after Google succeeds. Roles live on the users table.**
  Reason: the app must not have passwords; the allow-list is the only gate.
  Silent default: first login with the configured admin email gets role admin; everyone else student.
  Check: `pytest tests/test_auth.py::test_acl_rejects_unknown_email`.
- **Session cookie: signed, HttpOnly, SameSite=Lax, Secure whenever the base URL is https, 14 days.**
  Reason: defaults shipped with no Secure flag on an https-only site.
  Silent default: `https_only = BASE_URL.startswith("https")`.
  Check: `pytest tests/test_auth.py::test_cookie_secure_under_https`.
- **Every route uses require_user or require_admin, or is listed in PUBLIC_ROUTES with a reason. A test enumerates the app's routes (decorated routes, mounts, and framework routes) against that list.**
  Reason: an unlisted route is an unauthenticated route.
  Silent default: require_user.
  Check: `pytest tests/test_route_auth.py`.
- **Ownership is checked by the shared helpers; admin bypass is the same on every helper; a resource the user does not own is 404, never 403.**
  Reason: two helpers with different admin semantics.
  Silent default: one helper per resource in the helpers module, whatever the project already names them.
  Check: `pytest tests/test_auth.py::test_ownership_helpers`.
- **State-changing requests (POST, PATCH, DELETE) from the browser carry a CSRF token: hidden input in forms, X-CSRF-Token header on fetch, one dependency checks it after authentication (anonymous requests get the login redirect, authenticated requests without a token get 403). The token exists only for authenticated sessions; anonymous pages set no cookie. Token-in-URL API routes are exempt and listed.**
  Reason: cookie-authenticated POSTs under SameSite=Lax only; a public page must stay cookie-free.
  Silent default: double-submit token stored in the session on first authenticated render.
  Check: `pytest tests/test_csrf.py` (parametrized over every non-exempt state-changing route: authenticated without token → 403) and `python scripts/check_templates.py --csrf` (every `<form` whose method is post contains the token input before its `</form>`; every `fetch(` whose argument list contains `method:` with POST, PATCH, or DELETE also contains `csrfHeaders()`).
- **A secret carried in a URL path is compared with hmac.compare_digest and a miss returns 404.**
  Reason: timing and enumeration.
  Silent default: as stated.
  Check: `pytest tests/test_mcp.py::test_wrong_token_404_all_methods`.

## Layout
app/auth.py (OAuth registry) · app/routes/auth.py · app/helpers.py (dependencies) · app/csrf.py

## Not covered
Who is on the allow-list (private overlay).
