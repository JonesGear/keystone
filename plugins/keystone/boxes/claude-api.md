# Claude API: one module, evals, replay
Dated 2026-09-01. Replaces: nothing.

## Decisions
- **One module (app/llm.py) owns the client, the model constants, the exception types, and every call shape. Nothing else imports anthropic, names a model, or catches an SDK exception; callers catch the module's own exceptions.**
  Reason: four clients and 17 hardcoded model strings meant no place to add caching, retry, or cost logging.
  Silent default: `llm.call_tool(prompt_id, ...)` for structured output; `llm.complete(prompt_id, ...)` for plain text (image blocks allowed); `llm.stream(prompt_id, ...)` yielding `("text", str)` chunks and, when a tool is attached, a final `("tool", dict)`; `llm.LLMError` with subclass `llm.LLMBadRequest` so callers can keep warn-and-continue semantics for bad inputs. Per-call timeouts passed explicitly override the defaults.
  Check: `grep -rln "anthropic" app/ scripts/ | grep -v llm.py | wc -l` is 0; `grep -rn "claude-" app/ | grep -v llm.py | wc -l` is 0 (comments and docstrings included: scrub them).
- **Structured output is tool use with tool_choice forced; never parse freeform JSON.**
  Reason: freeform JSON fails on long outputs.
  Silent default: one tool per prompt, schema next to the prompt.
  Check: `pytest tests/test_llm.py::test_call_tool_forces_tool`.
- **Every system prompt carries cache_control ephemeral; every call sets max_retries, a timeout, and logs one structured line: prompt_id, model, input/output/cached tokens, ms, and any ids the caller passes.**
  Reason: uncached 16k-token calls with no retry and no cost record.
  Silent default: retries 2; timeout 120s for generation, 15s for runtime calls; log to the app logger.
  Check: `pytest tests/test_llm.py::test_cache_control_and_usage_log`.
- **Every prompt has a prompt_id and an eval case in evals/<prompt_id>.json: recorded inputs, one or more recorded outputs, invariants. check.sh replays; `--record` re-records live.**
  Reason: the only way to test a prompt change was uploading a real file and waiting.
  Silent default: tool prompts assert schema validity plus the domain rules the prompt states; text prompts assert non-empty output and the rules the prompt states. A stream's recorded output is `{"text": "<full text>", "tool": <input dict or null>}`; replay yields the text as one chunk, then the tool event. One prompt_id per prompt/tool variant.
  Check: `python -m evals` exits 0.
- **LLM_MODE=replay returns recorded outputs for the prompt_id, in recorded order, wrapping around; tests and the integration check run in replay.**
  Reason: deterministic pipeline tests without spend.
  Silent default: live in containers, replay under pytest.
  Check: `pytest tests/test_llm.py::test_replay_returns_fixture`.
- **Content generation runs in a background job. An HTTP request may make one bounded runtime call (stream or ≤15s) and nothing else.**
  Reason: minutes-scale generation inside a request.
  Silent default: new generation = new job row.
  Check: review of the route diff.
- **Runtime text calls stream to the browser over SSE. The browser never holds an API key.**
  Reason: the student waits on a spinner otherwise.
  Silent default: `llm.stream` → StreamingResponse.
  Check: `grep -rn "ANTHROPIC\|anthropic" app/templates app/static | wc -l` is 0.

## Layout
app/llm.py · evals/<prompt_id>.json · evals/inputs/ · evals/__main__.py

## Not covered
Job rows (background-jobs). Which model is FAST/STRONG is a constant in llm.py, changed by a spec.
