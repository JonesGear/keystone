# Boxes

One page per concern. A box is a decision procedure, not a preference list.
It exists so that a build never has to ask, and a spec never has to re-decide.

## Format

```
# <concern>            e.g. "Web app: FastAPI + Jinja"
Dated YYYY-MM-DD. Replaces: <what it superseded, or "nothing">.

## Decisions
- **Rule.** One sentence, imperative.
  Reason: why, in one line. Usually a failure that happened.
  Silent default: what to do when the spec says nothing.
  Check: the command or test that proves it, or "review" if none exists yet.

## Layout            (only if the concern has a shape on disk)
## Not covered       (what belongs to another box, by name)
```

## Rules

- Written only when a spec session hits a concern with no box.
- Every rule carries a reason. No reason, no rule.
- Every rule names its check. "Review" is allowed but is a debt.
- Replace lines, never append corrections. The file is the current truth.
- One page. If a box grows past that, split by concern.
- Generic boxes live here. Anything naming a person, host, or deploy target
  lives in the private overlay, not in this plugin.

## Index

- web-fastapi-jinja.md — Web app: FastAPI + Jinja, no build step
- postgres-alembic.md — Database: PostgreSQL + Alembic
- docker-compose-stack.md — Docker stack: prod compose + dev override
- claude-api.md — Claude API: one module, evals, replay
- background-jobs.md — Job rows and the worker
- auth-oauth-acl.md — Google OAuth, allow-list, sessions, CSRF
- file-storage.md — Storage behind one module
- frontend-css-js.md — Tokens, CSS layout, static JS, CDN, CSP
- context-budget.md — File size as implementer context
- config-env.md — Configuration and secrets
- tests-fastapi-postgres.md — Test harness on a real database
- git-commits.md — Commits and pushes
