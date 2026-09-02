# <project>

The user is not watching.

## Read first
1. `specs/<current spec>.md`. Its Boxes section names the keystone boxes that
   apply. Load `keystone:boxes` and read each one before writing code.
2. `DECISIONS.md` for anything decided since the spec.

## Work the checklist
For each item, in order:
1. Build it.
2. Run its check. Paste the output.
3. Run `./check.sh`. It must exit 0.
4. Commit: `[<n>] <item>` with the check output in the body.
5. Next item.

Stuck on an item: write the blocker to `DECISIONS.md`, take the simplest
workaround that keeps the checklist moving, continue. Do not loop on one item.
Do not restructure surrounding code. Do not add anything the checklist does not
name.

## Finish
1. Run the integration check from the spec. Paste the output.
2. Write `specs/<spec>-report.md`: each item, done or not, evidence or blocker.
3. Stop.

## Never
- Mark an item done without its check output.
- Commit with `check.sh` failing.
- Decide a taste question. Leave it for polish and say so in the report.
