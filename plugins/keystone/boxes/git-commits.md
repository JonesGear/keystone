# Commits and pushes
Dated 2026-09-01. Replaces: nothing.

## Decisions
- **Commit after every checklist item with check.sh green; push after every commit. Formatting-only changes are their own commit.**
  Reason: remote is the backup; the log is the memory; a reformat buried in a feature commit hides the feature.
  Silent default: `[<n>] <item>` subject; body has a `check: <command>` line, the output, and a final `exit 0` line.
  Check: `git log -1 --format=%B | grep -c "^exit 0"` is at least 1.
- **No attribution trailers or generated-with lines in commits or source.**
  Reason: the owner owns the work.
  Silent default: plain message.
  Check: `git log -1 --format=%B | grep -ciE "co-authored-by|generated with|-session:"` is 0 (no author, tool, or session trailer of any kind).
- **Build on main unless the spec says otherwise.**
  Reason: one owner, one implementer.
  Silent default: main.
  Check: review.

## Not covered
Deploy and project-specific ignored directories (private overlay). Secrets and .env (config-env). CI: none unless a spec adds it.
