# Job rows and the worker
Dated 2026-09-01. Replaces: nothing.

## Decisions
- **A job is a row with status, started_at, completed_at, error_message. Four states, named by constants on the model: QUEUED, PROCESSING, DONE, ERROR. The stored strings may differ per model; code never writes a status literal.**
  Reason: three job types grew three vocabularies; an undeclared "failed" value shipped and no UI knew it.
  Silent default: a new async task is a new job model with exactly these columns and constants.
  Check: `pytest tests/test_jobs.py::test_status_constants` (every job model has the four constants and the four columns); `grep -rnE --include=*.py "status[\"']?\s*[:=]\s*[\"'](queued|processing|ready|complete|done|error|failed)[\"']" app/ | grep -v models.py | wc -l` is 0.
- **Claiming is one shared function: an atomic conditional UPDATE from queued to processing that sets started_at and returns whether it won.**
  Reason: three claim mechanisms in one worker.
  Silent default: `jobs.claim(db, Model, id)`.
  Check: `grep -rn "with_for_update" app/ | wc -l` is 0; `pytest tests/test_jobs.py::test_claim_is_exclusive`.
- **At worker start, every PROCESSING row of every job model is reset to QUEUED; every tick, PROCESSING rows with started_at older than the stale cutoff are reset.**
  Reason: uploads stuck in processing forever after a crash.
  Silent default: cutoff 45 minutes (a job can legitimately run several 120s calls with retries; a short cutoff re-queues live work and duplicates additive output).
  Check: `pytest tests/test_jobs.py::test_startup_resets_every_model tests/test_jobs.py::test_stale_sweep_all_models`.
- **Errors and cancellations record a message on the row and log the traceback; no bare except: pass.**
  Reason: swallowed exceptions.
  Silent default: `jobs.fail(db, row, exc_or_message)`.
  Check: `grep -rnE "except( Exception)?:\s*pass" app/ | wc -l` is 0.
- **The worker has a --once flag that calls each poller function once (no scheduler), waits for their thread pools, and exits.**
  Reason: tests and the integration check need a deterministic run of the real path.
  Silent default: `python -m app.worker --once`.
  Check: `pytest tests/test_jobs.py::test_once_drains_queue`.
- **Jobs are not retried at the row level; an ERROR row stays until a person re-queues it. Retries live inside the LLM module.**
  Reason: automatic re-queue loops on a bad input and spends money.
  Silent default: as stated.
  Check: `pytest tests/test_jobs.py::test_error_rows_stay_error` (an ERROR row is still ERROR after `--once`).
- **The UI learns a job's state by polling a status endpoint.**
  Reason: minutes-scale generation (that a request never waits on generation is claude-api's rule).
  Silent default: JSON status endpoint + 5–10s poll.
  Check: review of the route diff.

## Layout
app/jobs.py · app/worker.py

## Not covered
What a job does with Claude (claude-api).
