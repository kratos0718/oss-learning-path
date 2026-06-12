# 04 — Every Contribution, in Plain English

> One row per PR. What it was, why it mattered, the fix, the status. Read the linked lesson for the deep version. This is your "tell me about a contribution" lookup table.

---

## ✅ MERGED (10)

### 1. unsloth #6135 — blocking `time.sleep` in async ⭐⭐ (codehound found it)
- **What:** the model-export route's `load_checkpoint` (an async FastAPI handler) waited for a training subprocess to exit with `time.sleep(0.5)` in a loop — up to **30s of blocking** inside async, freezing the whole event loop (every other request stalls).
- **Fix:** `await asyncio.sleep(0.5)`, matching the async patterns already used elsewhere in the same file (`asyncio.to_thread`, `await asyncio.sleep`).
- **Why it stands out:** **first concurrency-bug merge into the prestige tier (unsloth = 40k⭐)** — a *real* bug my own tool found, merged by maintainer @wasimysaid. The cleanest proof codehound works.
- **The 2-sentence explanation (memorize):** *"An async route backed off with a blocking time.sleep in a loop — up to 30s — which freezes the entire event loop so every other request stalls. I swapped it for await asyncio.sleep, matching the async pattern the file already used elsewhere."*
- **Deep dive:** [03_async/03_blocking_the_loop.md](../03_async/03_blocking_the_loop.md).

### 2. huggingface_hub #4289 — *shipped in v1.17.0*
- **What:** 4 public API parameters (`token`, `endpoint`, `maxdepth`, `original_order`) were undocumented — users couldn't tell they existed.
- **Why it matters:** undocumented auth params mean people can't authenticate correctly. Documentation IS the product for a library.
- **Status:** Merged by the lead maintainer (@Wauplin) and **shipped in the v1.17.0 release** — `pip install huggingface_hub==1.17.0` runs your change.
- **Type:** docs.

### 3. agno #8138 — cookbook typos
- **What:** duplicate-word typos in example files. **Status:** Merged. Type: docs. (Small, but it's how you learned the workflow.)

### 4. agno #8158 — blocking `time.sleep` in async ⭐
- **What:** `time.sleep(1)` froze the event loop inside an async Couchbase routine.
- **Fix:** `await asyncio.sleep(1)`. **Status:** Merged ("Thanks for spotting this!"). 
- **Deep dive:** [03_async/03_blocking_the_loop.md](../03_async/03_blocking_the_loop.md). Real concurrency bug.

### 5. pydantic #13239 — docstring fix
- **What:** docstring typo in `create_model`. **Status:** Merged by maintainer @Viicos. Type: docs. (pydantic = 22k⭐, very respected.)

### 6. agno #8186 — blocking `requests.get` in async Discord handler ⭐ (codehound found it)
- **What:** synchronous `requests.get` froze the whole Discord bot on every video/document attachment; also a latent 403.
- **Fix:** `await media.read()` (native async + authenticated). **Status:** Merged.
- **Deep dive:** [03_async/03_blocking_the_loop.md](../03_async/03_blocking_the_loop.md).

### 7. agno #8161 — file-handle leak ⭐
- **What:** `open()` without `close()` in `transcribe_audio` leaked file descriptors → "too many open files" crash in a loop.
- **Fix:** `with open(...)`. **Status:** Merged after a review round (you rewrote the test to be behavioral).
- **Deep dive:** [02_python/03_files_and_resources.md](../02_python/03_files_and_resources.md).

### 8. marimo #9667 — `filter` parameter (a FEATURE) ⭐
- **What:** added a NEW public API — a `filter` argument on `mo.ui.file_browser()` (regex / pattern / callable).
- **Why it stands out:** a *feature*, not a bug fix — a maintainer trusted your API design. The review taught you to isolate callable-filter errors to `OSError` (so one broken file doesn't hide the rest) and centralize logic.
- **Status:** Merged by maintainer @kirangadhave. (marimo = 11k⭐, YC.)

### 9. mem0 #5302 — mutable default arguments (B006)
- **What:** mutable defaults in `Completions.create` and `BaseEmbedderConfig` — the same dict/list shared across every call.
- **Fix:** default to `None`, build a fresh one inside the body; added a regression test.
- **Status:** Merged by maintainer @kartik-mem0. (mem0 = 35k⭐.) Deep dive: [02_python/02_functions_args_defaults.md](../02_python/02_functions_args_defaults.md).

### 10. accelerate #4051 — missing public-API parameters
- **What:** documented undocumented params in `load_accelerator_state`, `find_executable_batch_size`, and `send_to_device`.
- **Status:** Merged by maintainer @SunMarc. (HuggingFace accelerate = 8k⭐.) Type: docs.

---

## 🟢 OPEN — real bug fixes awaiting review

### vLLM #45249 — fire-and-forget abort task ⭐⭐⭐ (codehound found it · the STAR item)
- **What:** the `/abort_requests` route in vLLM's disagg `api_router.py` schedules `engine_client.abort()` with a bare `asyncio.create_task(...)` and discards it. The loop only weak-references tasks, so the abort can be GC'd before it runs → requested aborts silently don't happen, and the endpoint returns 200 either way (invisible failure).
- **Fix:** module-level `_background_tasks` set + `add_done_callback` to keep a strong reference until done.
- **Why it's huge:** **vLLM is *the* LLM serving framework (40k⭐)** — the most prestigious target possible, a real reliability bug in a hot path. Signed off with DCO.
- **2-sentence:** *"They scheduled a request-abort with a bare create_task and threw the task away; asyncio only weak-references tasks so it can be GC'd before the abort runs. I hold the task in a set until it completes."*
- **Deep dive:** [03_async/04_fire_and_forget_tasks.md](../03_async/04_fire_and_forget_tasks.md).

### Microsoft autogen #7825 — fire-and-forget websocket stream task ⭐⭐ (codehound found it)
- **What:** autogen-studio's run websocket handler starts `ws_manager.start_stream(...)` via bare `create_task` → a user's streaming run can be GC'd mid-stream.
- **Fix:** module-level task set + `add_done_callback`. **Why it's huge:** **Microsoft** (50k⭐). Uses the Microsoft CLA.
- **Deep dive:** same fire-and-forget lesson.

### sglang #28029 — blocking `requests.get` in async server route ⭐⭐ (codehound found it)
- **What:** `remote_instance_transfer_engine_info` (async FastAPI route) calls `requests.get(timeout=5)` synchronously → blocks the event loop up to 5s per call, stalling every other request.
- **Fix:** `await asyncio.to_thread(requests.get, ...)` — offload the blocking call to a thread, no new dependency.
- **Why it matters:** sglang = 15k⭐, a top LLM serving framework. **Note the new tool in your kit: `asyncio.to_thread`** — the right fix when you must keep a sync call but can't block the loop.
- **Deep dive:** [03_async/03_blocking_the_loop.md](../03_async/03_blocking_the_loop.md) (see "offload to a thread").

### jina #6243 — blocking `time.sleep` in async voter retry ⭐ (codehound found it)
- **What:** `_async_call_add_voters` awaits, then backs off with `time.sleep(2.0)` in a retry loop (up to ~10s blocking). **Fix:** `await asyncio.sleep(2.0)`.
- **The careful bit (interview gold):** the file also had a **sync** sibling `_call_add_voters` with the same `time.sleep` — I left that one alone (blocking is fine in sync code) and fixed only the async one. jina = 21k⭐.

### khoj #1342 — blocking `requests.post` (OAuth) in async auth route ⭐ (codehound found it)
- **What:** the Google OAuth callback (async `auth` route) exchanges the code for a token with a synchronous `requests.post` → freezes the loop during every login's token round-trip.
- **Fix:** `await asyncio.to_thread(requests.post, ...)`. khoj = 28k⭐.

### Future AGI #821 — fire-and-forget tasks in PromptStreamConsumer ⭐⭐⭐ (codehound found it · FOUNDER-INVITED)
- **What:** their WebSocket consumer ran every prompt execute/improve/generate via bare `asyncio.create_task(...)` and discarded the task. asyncio only weak-references tasks, so the GC could collect one mid-run → a user's prompt execution silently drops (no result, no error).
- **Fix:** a `_spawn()` helper that adds each task to a `self._background_tasks` set (strong ref) and removes it via `add_done_callback` on completion; cancel leftovers on `disconnect()`.
- **Why it's huge:** Future AGI's **founder Nikhil Pareek personally invited me to contribute** after seeing my agno/phidata work. This is my first PR to his repo — opened issue #819 + PR #821 the same day. An AI-*reliability* company, and the bug is a silent reliability hole in their own stack.
- **The 2-sentence explanation (memorize):** *"They fired background tasks without keeping a reference, and asyncio only weak-references tasks, so the GC could collect them mid-run and silently drop a request. I store each task in a set and discard it on completion, so it stays alive until it's actually done."*
- **Deep dive:** [03_async/04_fire_and_forget_tasks.md](../03_async/04_fire_and_forget_tasks.md).

### OpenAI #3553 — fire-and-forget tasks in RealtimeSession ⭐⭐ (codehound found it)
- **What:** 3 discarded `create_task` calls could drop error events. **Fix:** strong-reference helper.
- **Why it's huge:** it's the **official OpenAI Agents SDK**. Deep dive: [03_async/04_fire_and_forget_tasks.md](../03_async/04_fire_and_forget_tasks.md).

### litellm #29417 — fire-and-forget cache writes (codehound found it)
- **What:** discarded `create_task` cache writes → silently lost cache → wasted LLM spend. Bot greptile confirmed it's real. Deep dive: same async lesson.

### ragas #2757 — deprecated `get_event_loop()` (codehound found it)
- **What:** `get_event_loop()` → `get_running_loop()` at 3 async sites. Deep dive: [03_async/02_asyncio_basics.md](../03_async/02_asyncio_basics.md).

### agno #8183 — span-exporter fire-and-forget + deprecated loop (codehound found it)
- **What:** dropped trace tasks + deprecated `get_event_loop()`. Two bugs, one PR.

### agno #8152 — mutable default arguments (B006)
- **What:** 10 mutable defaults across toolkits. **Fix:** `None` + create inside body. Deep dive: [02_python/02_functions_args_defaults.md](../02_python/02_functions_args_defaults.md).

---

## 🟢 OPEN — earlier (docs/smaller, across many orgs)
crewAI #5969/#5970/#5968 (deprecated APIs + docs, coderabbit-approved) · PyTorch torchtune #2964 · HuggingFace PEFT/datasets · pydantic-ai · instructor · llama_index. These built your **org diversity** (18+ organizations).

---

## 🔴 CLOSED — and what they taught you (be honest about these)
- **transformers #46232** — closed: "no small doc fixes, it's noise." → Lesson: big repos ban tiny PRs.
- **scikit-learn #34095** — closed: PR before the issue was triaged. → Lesson: follow each project's process.
- **langchain #37763 / #38011** — auto-closed: needs a pre-approved issue first. → Lesson: some big repos auto-close any cold PR; don't waste shots there.
- **dspy #9907** — closed by maintainer: *"`get_event_loop()` doesn't emit a DeprecationWarning when called from inside a running loop — which is exactly this code."* → **Important lesson: `get_event_loop()` is only deprecated when there's no running loop.** Inside a running loop it's fine. My CH004 check over-flagged it. This is the single most useful correction I got — it sharpened my understanding of the exact rule.

> In an interview, owning these shows maturity: *"I learned that contributing well isn't about volume — it's about respecting each project's norms and submitting changes maintainers actually want. One maintainer even corrected my understanding of when get_event_loop is actually deprecated — only outside a running loop — which I took straight back into my own tool."*

---

## The numbers (as of June 2026)
**10 merged** · **11 open real-bug PRs** · **18+ organizations** · **codehound: bugs it found are merged into unsloth (40k⭐) and agno (25k⭐); more flagged at vLLM, Microsoft autogen, sglang, jina, khoj, OpenAI, litellm, and Future AGI (founder-invited).**
