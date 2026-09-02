# <project or feature name>
Dated YYYY-MM-DD. Status: deciding | reviewed | building | done.

## Goal
One sentence. What is true when this is finished.

## Boxes used
List by name. Anything a box already decides is not restated here.
- boxes/<name>.md

## Decisions
Everything this project decides that a box does not. Each locked, with a reason.
Includes meaning decisions (what does this honestly claim) and taste decisions
(what it looks like, what it says). A taste decision that can't be made blind
says "mockup first, then choose" and the build waits for it.

- **D1.** <decision>. Reason: <why>.

## Non-goals
What this will not do, stated so a build can't drift into it.

## Shape
Only the parts that are new. Reference boxes for the rest.
- Data: <entities, fields, ownership>
- Interfaces: <routes, functions, contracts between parts>
- Layout: <files and modules that will exist>

## Checklist
One buildable item per line, in build order, each with the check that proves
it. No check means it moves to Polish or the spec is not done.

| # | Item | Check (a command, test, screenshot, or eval) | Decision |
|---|------|----------------------------------------------|----------|
| 1 | | | D1 |

## Integration check
The one command or walk-through that proves the items work together, not just
alone. Runs last in the build and first in review.

## Polish
Taste work deferred on purpose. Done after the checklist is green, browser
open, small iterations. Not built unattended.

## Open questions
Must be empty before status moves to "reviewed".
