# Keystone

A library of decisions and checks for finishing projects with Claude Code.
Not a process, not a memory system, not an orchestrator, not a workspace.

Dated 2026-09-01. Every file here encodes an assumption about what the model
can't do alone. Assumptions expire. Prune on every use.

## What it holds

1. **Boxes** (`boxes/`). One page per concern, written as decision procedures:
   rule, reason, default when the spec is silent, how it is checked. Projects
   reference boxes by name and never re-decide them.
2. **One spec template** (`templates/spec.md`). For the part of a project that
   is genuinely new. Ends in a checklist where every item has a runnable check.
3. **The loop standard** (`templates/loop.md`). What "verified" means and how
   it gates every commit.

## The process

Two loops. The project loop builds things. The keystone loop captures what the
project loop learns.

1. **Decide** (skill `keystone:spec`, in the project folder, plan mode).
   Name the boxes this project uses. Decide only what is new: data, interfaces,
   meaning, taste. Lock each with a reason. Taste that can't be decided blind
   gets a mockup first. When a concern has no box, stop and write the box.
2. **Checklist.** The spec ends in numbered items, each paired with the check
   that proves it. No check means it is polish or it is not specified yet.
3. **Self-review.** A fresh-context reviewer plays the senior dev against the
   spec until it has no questions. Findings go back into the spec.
4. **Build, unattended.** Fresh session. `/goal` or a Ralph loop on the
   checklist. Build item, run its check, run `check.sh`, commit with evidence,
   next item. Stuck: log it, simplest workaround, continue. Do not loop.
5. **Review once.** Fresh-context reviewer (`/code-review`) plus the human,
   against the checklist and the evidence. Leaks become checks. Taste goes to
   polish.
6. **Polish.** Short, browser open, small iterations on a feature that works.
7. **Close the keystone loop.** Non-project decisions become boxes. Leaks become
   checks. Friction becomes one of those or stays friction.

The human decides, grades samples, reviews once, polishes. The agent does
everything else, including proving it.

## Rules that keep it small

- Nothing goes in unless a project needed it.
- A box is a page. Longer means two concerns or process creeping in.
- Zero project state. A file naming a project is in the wrong place.
- Decisions get replaced, not appended to.
- Readable in one sitting. If not, prune.

## Adopted, not built

Plan mode for deciding. `/goal` and `ralph-wiggum` for loop-until-done.
`/verify` and Playwright MCP for driving the app and headless screenshots.
`/code-review`, `/simplify`, `/security-review` for review. `/frontend-design`
and `/design` for mockups. `plugin-dev` for validating this plugin.
