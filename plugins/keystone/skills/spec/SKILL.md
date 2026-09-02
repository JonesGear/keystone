---
name: spec
description: This skill should be used when the user says "spec this", "let's decide", "start a decision session", "respec", "write the spec for", "new project", "plan this feature with keystone", or wants to turn an idea, a feature, or an existing project into a keystone spec with a checked checklist. Runs the decision session and the self-review.
version: 0.1.0
---

# Spec: the decision session

Produce one spec from `${CLAUDE_PLUGIN_ROOT}/templates/spec.md`, in the
project's own folder at `specs/<name>.md`. Run in plan mode. The session ends
when a fresh reviewer has no questions, not when the human is tired.

## 1. Situate

- If the project folder does not exist: create it, `git init`, add
  `.gitignore` and `.env.example`, copy
  `${CLAUDE_PLUGIN_ROOT}/templates/implementer-claude.md` to `CLAUDE.md`.
- If it exists: read `CLAUDE.md`, `DECISIONS.md`, the latest spec, and
  `git log --oneline | head -30`. For a respec, also produce a short
  "current state" section: what exists, what is dead, what is duplicated.

## 2. Name the boxes

Load the `boxes` skill. List which boxes this project uses. For any concern
the project touches that has no box, stop and write the box first, using the
format in `${CLAUDE_PLUGIN_ROOT}/boxes/README.md`. Then continue.

## 3. Interview

Ask until nothing is deferred. No cap on questions. Group them, but ask the
hard ones. The three kinds, in this order:

- **Meaning.** What does this honestly claim? Who is it for and not for? What
  does it own and what does the user own?
- **Shape.** Data, interfaces, layout, only where a box does not decide it.
- **Taste.** Layout, wording, feel. Decide now, or say "mockup first, then
  choose." Never "build both and pick later."

Write each answer into Decisions with its reason as it is given.

## 4. Checklist

Turn the decisions into buildable items in build order. Each item gets one
check: a test name, a command, a route plus screenshot, or an eval case. Read
`${CLAUDE_PLUGIN_ROOT}/templates/loop.md` for what counts. An item with no
possible check goes to Polish. Then write the integration check.

## 5. Self-review

Launch a fresh-context reviewer agent (no shared context) with the spec and
the boxes it names. Its brief: play the senior engineer who will build this.
Report, as numbered findings:

- Ambiguity: any line two builders would read differently.
- Hidden decisions: anything the build would have to decide that the spec does
  not.
- Unrunnable checks: checklist items whose check is not a command, test,
  screenshot, or eval.
- Integration gap: what could pass every item and still not work.
- Box drift: any rule restated inline that a box already decides, or should.
- Missing non-goal: what a build would plausibly add that this does not want.

Fold findings into the spec. Repeat until the reviewer reports none. Then set
status to "reviewed".

## 6. Hand off

Write the project's `check.sh` stub if none exists (see loop.md). Tell the
user which runner to start the build with (`/goal` or `/ralph-loop`) and the
condition to give it. Do not start the build in this session.

## Additional resources

- `${CLAUDE_PLUGIN_ROOT}/templates/spec.md` - the template
- `${CLAUDE_PLUGIN_ROOT}/templates/loop.md` - what "checked" means
- `${CLAUDE_PLUGIN_ROOT}/boxes/README.md` - box format and index
