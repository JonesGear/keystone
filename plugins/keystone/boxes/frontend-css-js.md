# Tokens, CSS layout, static JS, CDN, CSP
Dated 2026-09-01. Replaces: nothing.

## Decisions
- **Colors, type, radii, shadows are tokens in base.css :root; new or touched CSS uses tokens, never literals.**
  Reason: a design system that survives new pages.
  Silent default: light theme; no dark mode unless a spec says so.
  Check: review of the CSS diff (a literal count script is a debt).
- **CSS is base.css + pages/<page>.css + components/<component>.css; a page loads base plus its own files in the head block.**
  Reason: one 2000-line stylesheet happened.
  Silent default: new page = new pages file.
  Check: review.
- **Every literal class token in a template has a rule in a stylesheet or a template `<style>` block, or is listed in scripts/known_dynamic_classes.txt with the library or script that adds it.**
  Reason: orphan classes from dead branches.
  Silent default: delete the class or add the rule.
  Check: `python scripts/check_templates.py --classes` (tokens containing `{{` or `{%` are skipped).
- **No inline scripts, no `on*=` handler attributes, no `javascript:` URLs. Page behavior lives in app/static/js/<page>.js; shared helpers in static/js/base.js; server values reach JS via data-* attributes or a `<script type="application/json">` block.**
  Reason: inline blocks and handlers make a script-src policy without unsafe-inline impossible.
  Silent default: addEventListener in the page file.
  Check: `python scripts/check_templates.py --inline-scripts` (counts `<script>` without src except `type="application/json"` data blocks, `on[a-z]+=` attributes, `javascript:` hrefs).
- **The app sends a Content-Security-Policy: scripts, styles and fonts from 'self' only (styles also 'unsafe-inline' because math renderers emit inline styles); the analytics host is the one listed third party.**
  Reason: XSS blast radius on generated content; math renderers emit inline styles that cannot pass a strict style-src.
  Silent default: `default-src 'self'; script-src 'self' <analytics>; style-src 'self' 'unsafe-inline'; font-src 'self'; img-src 'self' data:; connect-src 'self' <analytics>`.
  Check: headless render reports zero `securitypolicyviolation` events.
- **Third-party fonts and libraries are vendored under static/ at pinned versions recorded in static/vendor/VERSIONS; no script, stylesheet, or font loads from a CDN or a font service. The only external request is a deferred analytics script.**
  Reason: a school-managed browser profile filtered every cross-origin request and every page waited seconds on Google Fonts and a CDN before first paint; the student's own browser is that profile.
  Silent default: fetch the exact version once, verify against the published hash, commit the files; fonts as latin + latin-ext woff2 with a local @font-face stylesheet linked before base.css.
  Check: `python scripts/check_templates.py --sri` (any external script or stylesheet other than the analytics URL fails).
- **Wide content scrolls inside its container; the body never scrolls horizontally at 375px.**
  Reason: phone browsers must not break even when desktop is primary.
  Silent default: overflow-x auto on tables and code.
  Check: headless render (tests-fastapi-postgres) with `--mobile-check`: at 375px, `document.documentElement.scrollWidth <= 375`; a spec names the item that turns the flag on.

## Layout
app/static/css/base.css · pages/ · components/ · app/static/js/base.js · js/<page>.js · scripts/check_templates.py (flags: `--classes --inline-scripts --sri --innerhtml --csrf`; scans app/templates/**/*.html and app/static/js/**/*.js; the innerHTML and csrf rules are stated in web-fastapi-jinja and auth-oauth-acl) · scripts/known_dynamic_classes.txt

## Not covered
Fonts and colors (the project's spec or design notes). Route auth (auth-oauth-acl).
