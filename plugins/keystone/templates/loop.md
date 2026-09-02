# Loop standard
Dated 2026-09-01. What "verified" means and how it gates every commit.

## Every project has `check.sh`
One script at the project root. Exit 0 or the build does not commit. It runs,
in order, whatever applies:
1. Tests (`pytest`, `vitest`, ...)
2. Lint and format (`ruff check`, `ruff format --check`, `eslint`, ...)
3. Typecheck where the stack has one
4. Static template check: every CSS class referenced in a template has a rule
5. Headless render: load the listed routes, save screenshots to `screenshots/`,
   fail on HTTP errors or console errors
6. Prompt evals: run `evals/` cases for any prompt the project ships

The spec's Verification section lists which of these apply and which routes.

## Every checklist item has a check
Named in the spec. The build runs it before marking the item done, and pastes
the command output as evidence in the commit body. An item without evidence is
not done. Assertions ("tests pass") are not evidence. Output is.

## Every build ends with the integration check
The composed thing does the thing. Components passing alone is not enough.

## The judge is not the worker
Spec self-review and code review run in a fresh context that did not write the
work. The worker never grades itself.

## Leaks become checks
When a human catches something in review that a check should have caught, the
question is "what check would have caught this," and that check is added here
or to the box it belongs to. This list grows only that way.

## Taste is not in the loop
Layout, wording, and feel are decided in the spec (mockup first if needed) or
deferred to polish. They are never a checklist item.

## Adopted runners
- `/goal "<condition naming a check>"` gates stopping on a judged condition.
- `/ralph-loop "<prompt>" --max-iterations N --completion-promise "<text>"`
  re-feeds a fixed prompt until the promise is true.
- `/verify` drives the running app and records its recipe.
- Playwright MCP (`--headless`) for screenshots without a visible window.
