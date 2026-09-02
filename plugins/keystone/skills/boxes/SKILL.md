---
name: boxes
description: This skill should be used when a spec or build session needs to "load the boxes", "read the keystone boxes", "which boxes apply", "check the box for", or when a spec's Boxes section names a box and the session must apply it. Locates and reads keystone decision boxes.
version: 0.1.0
---

# Boxes

Keystone boxes are one-page decision procedures, one per concern. They live in
`${CLAUDE_PLUGIN_ROOT}/boxes/`. The index is `${CLAUDE_PLUGIN_ROOT}/boxes/README.md`.

## To apply boxes in a build

1. Read the spec's "Boxes used" section.
2. Read each named file in `${CLAUDE_PLUGIN_ROOT}/boxes/`.
3. Follow every rule. When the spec is silent, use the box's silent default.
4. Do not restate box rules in the spec, DECISIONS.md, or CLAUDE.md.

## To find out whether a box exists

Read `${CLAUDE_PLUGIN_ROOT}/boxes/README.md` and list the index. If the concern
has no box, the spec session writes one using the format in that README before
continuing. A box is written only when a project needs it, never in advance.

## Private overlay

Rules naming a person, host, or deploy target are not in this plugin. They
live in the user's private overlay at `~/.claude/keystone/` if present. Read
that directory's `README.md` when it exists.
