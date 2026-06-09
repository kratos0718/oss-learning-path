# 04 — Every Contribution, in Plain English

> One row per PR. What it was, why it mattered, the fix, the status. Read the linked lesson for the deep version. This is your "tell me about a contribution" lookup table.

---

## ✅ MERGED (9)

### 1. huggingface_hub #4289 — *shipped in v1.17.0*
- **What:** 4 public API parameters (`token`, `endpoint`, `maxdepth`, `original_order`) were undocumented — users couldn't tell they existed.
- **Why it matters:** undocumented auth params mean people can't authenticate correctly. Documentation IS the product for a library.
- **Status:** Merged by the lead maintainer (@Wauplin) and **shipped in the v1.17.0 release** — `pip install huggingface_hub==1.17.0` runs your change.
- **Type:** docs.

### 2. agno #8138 — cookbook typos
- **What:** duplicate-word typos in example files. **Status:** Merged. Type: docs. (Small, but it's how you learned the workflow.)

### 3. agno #8158 — blocking `time.sleep` in async ⭐
- **What:** `time.sleep(1)` froze the event loop inside an async Couchbase routine.
- **Fix:** `await asyncio.sleep(1)`. **Status:** Merged ("Thanks for spotting this!"). 
- **Deep dive:** [03_async/03_blocking_the_loop.md](../03_async/03_blocking_the_loop.md). Real concurrency bug.

### 4. pydantic #13239 — docstring fix
- **What:** docstring typo in `create_model`. **Status:** Merged by maintainer @Viicos. Type: docs. (pydantic = 22k⭐, very respected.)

### 5. agno #8186 — blocking `requests.get` in async Discord handler ⭐ (codehound found it)
- **What:** synchronous `requests.get` froze the whole Discord bot on every video/document attachment; also a latent 403.
- **Fix:** `await media.read()` (native async + authenticated). **Status:** Merged.
- **Deep dive:** [03_async/03_blocking_the_loop.md](../03_async/03_blocking_the_loop.md).

### 6. agno #8161 — file-handle leak ⭐
- **What:** `open()` without `close()` in `transcribe_audio` leaked file descriptors → "too many open files" crash in a loop.
- **Fix:** `with open(...)`. **Status:** Merged after a review round (you rewrote the test to be behavioral).
- **Deep dive:** [02_python/03_files_and_resources.md](../02_python/03_files_and_resources.md).

### 7. marimo #9667 — `filter` parameter (a FEATURE) ⭐
- **What:** added a NEW public API — a `filter` argument on `mo.ui.file_browser()` (regex / pattern / callable).
- **Why it stands out:** a *feature*, not a bug fix — a maintainer trusted your API design. The review taught you to isolate callable-filter errors to `OSError` (so one broken file doesn't hide the rest) and centralize logic.
- **Status:** Merged by maintainer @kirangadhave. (marimo = 11k⭐, YC.)

### 8. mem0 #5302 — mutable default arguments (B006)
- **What:** mutable defaults in `Completions.create` and `BaseEmbedderConfig` — the same dict/list shared across every call.
- **Fix:** default to `None`, build a fresh one inside the body; added a regression test.
- **Status:** Merged by maintainer @kartik-mem0. (mem0 = 35k⭐.) Deep dive: [02_python/02_functions_args_defaults.md](../02_python/02_functions_args_defaults.md).

### 9. accelerate #4051 — missing public-API parameters
- **What:** documented undocumented params in `load_accelerator_state`, `find_executable_batch_size`, and `send_to_device`.
- **Status:** Merged by maintainer @SunMarc. (HuggingFace accelerate = 8k⭐.) Type: docs.

---

## 🟢 OPEN — real bug fixes awaiting review

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
- **langchain #37763** — auto-closed: needs a pre-approved issue first.

> In an interview, owning these shows maturity: *"I learned that contributing well isn't about volume — it's about respecting each project's norms and submitting changes maintainers actually want."*

---

## The numbers (as of June 2026)
**9 merged** · **6 open real-bug PRs** · **18+ organizations** · **codehound: a bug it found is merged into agno, more flagged at OpenAI, litellm, and Future AGI (founder-invited).**
