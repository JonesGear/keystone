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
- **The app sends a Content-Security-Policy: scripts only from 'self' and listed CDN hosts, no unsafe-inline for scripts; styles from 'self', listed hosts, and 'unsafe-inline' (math renderers emit inline styles); every host listed explicitly.**
  Reason: XSS blast radius on generated content; math renderers emit inline styles that cannot pass a strict style-src.
  Silent default: `default-src 'self'; script-src 'self' <hosts>; style-src 'self' 'unsafe-inline' <hosts>; font-src 'self' <hosts>; img-src 'self' data:; connect-src 'self' <hosts>`.
  Check: headless render reports zero `securitypolicyviolation` events.
- **CDN library assets are pinned to an exact version with integrity and crossorigin attributes; pinning takes the version the CDN currently resolves so behavior does not change. A self-updating collector script (analytics) is exempt and named in config.**
  Reason: 3 of 11 tags had SRI; floating majors.
  Silent default: resolve the floating tag once, pin, hash.
  Check: `python scripts/check_templates.py --sri` (skips the URL in the analytics script setting; font stylesheets stay as CSS `@import`, which the check does not see).
- **Wide content scrolls inside its container; the body never scrolls horizontally at 375px.**
  Reason: phone browsers must not break even when desktop is primary.
  Silent default: overflow-x auto on tables and code.
  Check: headless render (tests-fastapi-postgres) with `--mobile-check`: at 375px, `document.documentElement.scrollWidth <= 375`; a spec names the item that turns the flag on.

## Layout
app/static/css/base.css · pages/ · components/ · app/static/js/base.js · js/<page>.js · scripts/check_templates.py (flags: `--classes --inline-scripts --sri --innerhtml --csrf`; scans app/templates/**/*.html and app/static/js/**/*.js; the innerHTML and csrf rules are stated in web-fastapi-jinja and auth-oauth-acl) · scripts/known_dynamic_classes.txt

## Not covered
Fonts and colors (the project's spec or design notes). Route auth (auth-oauth-acl).
