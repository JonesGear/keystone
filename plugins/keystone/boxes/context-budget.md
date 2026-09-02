# File size as implementer context
Dated 2026-09-01. Replaces: nothing.

## Decisions
- **The limit is what an implementer must read to do one task. 500 lines per file is the proxy for Python (app/, scripts/, evals/), templates, and static JS; tests 600; CSS and migrations are exempt.**
  Reason: an implementer that reads a 1600-line module to change one prompt spends its context before it starts.
  Silent default: split by responsibility so a task touches one file under the limit; CSS is split only when a task touches it.
  Check: `python scripts/check_sizes.py`.
- **Prompts and tool schemas live in a prompts module beside the code that calls them; repeated template blocks are partials.**
  Reason: a 1600-line module was 250 lines of prompts plus code; a 1254-line template carried four copies of a card.
  Silent default: `<package>/prompts.py`; `templates/partials/<block>.html`.
  Check: same script.
- **Implementers read with grep and line ranges; a full read of a file over 300 lines is noted in the build report.**
  Reason: context is the budget.
  Silent default: grep the symbol, read ±40 lines.
  Check: review of the build report.
- **One copy of a thing. A second implementation of an existing helper is a defect.**
  Reason: five quiz UIs, eleven Claude call blocks.
  Silent default: import the existing one; if it doesn't fit, change it.
  Check: review.

## Not covered
How to split a specific module (the spec names the boundaries).
