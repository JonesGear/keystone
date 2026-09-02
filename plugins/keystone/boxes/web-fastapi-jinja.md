# Web app: FastAPI + Jinja, no build step
Dated 2026-09-01. Replaces: nothing.

## Decisions
- **Server-render every page with Jinja; client behavior is limited to form posts, fetch calls, and DOM updates.**
  Reason: no build step means one repo, one container, one place to read.
  Silent default: a new page is a template extending base.html (where its JS lives is frontend-css-js's rule).
  Check: review.
- **One router module per resource in app/routes/<resource>.py; main.py only wires.**
  Reason: a 2300-line main.py happened once.
  Silent default: a route goes in the module whose prefix it shares; a new prefix is a new module.
  Check: `wc -l app/main.py` under 120; context-budget for module size.
- **New JSON request bodies are Pydantic models in app/schemas.py; browser forms use Form(). Existing routes are not migrated for their own sake.**
  Reason: freeform request.json() parsing hides validation errors.
  Silent default: JSON in → Pydantic; browser form → Form.
  Check: review of the route diff.
- **Auto-docs (/docs, /redoc, /openapi.json) are disabled unless a spec enables them.**
  Reason: they are unauthenticated and list every route of a private app.
  Silent default: `FastAPI(docs_url=None, redoc_url=None, openapi_url=None)`.
  Check: the route allow-list test (auth-oauth-acl) fails if they appear.
- **Never set innerHTML from a computed string. Text goes through textContent or createElement; markdown and server-rendered HTML fragments go through the sanitizer library as a DOM fragment and replaceChildren.**
  Reason: XSS through generated study content.
  Silent default: textContent.
  Check: `python scripts/check_templates.py --innerhtml` (innerHTML only with a string literal, in templates and static/js).
- **Errors show a page with the failure and the next step; never a blank 500. An unhandled exception renders the error template with a link back.**
  Reason: the student is alone at night.
  Silent default: flash message plus the page that was requested; a 500 handler for the rest.
  Check: `pytest tests/test_errors.py::test_500_renders_error_page` (TestClient with raise_server_exceptions=False on a failing route that a fixture adds to the app and removes from `app.router.routes` on teardown, so route-enumerating tests never see it).

## Layout
app/main.py · app/routes/<resource>.py · app/schemas.py · app/helpers.py · app/templates/<page>.html + partials/ · app/static/js/<page>.js · app/static/css/

## Not covered
Auth and route allow-list (auth-oauth-acl), CSS/JS/CDN rules (frontend-css-js), tests (tests-fastapi-postgres).
