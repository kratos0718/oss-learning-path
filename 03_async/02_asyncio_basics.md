# 03.2 — async / await / coroutines / tasks

> Goal: read async code fluently and know exactly what `create_task`, `get_event_loop`, and `get_running_loop` do.

---

## `async def` makes a coroutine

```python
async def fetch_data():
    ...
```
- `async def` defines a **coroutine function**. Calling it does NOT run it — it returns a **coroutine object** (a "job description," not the work itself).
- To actually run it, something must `await` it or schedule it on the event loop.

```python
coro = fetch_data()   # nothing happened yet — just a coroutine object
result = await coro    # NOW it runs (only legal inside another async function)
```

## `await` — "pause here until this finishes"

`await something` means: *"start this, and if it needs to wait, hand control back to the event loop so other tasks can run; resume me when it's done."*

You can only use `await` inside an `async def`. `await` is the **yield point** — the only place the event loop can switch tasks (lesson 03.1).

```python
async def handler():
    data = await fetch_data()      # pause, let others run, resume with data
    await save(data)               # pause again
```

## `asyncio.run(...)` — the entry point

Sync code can't `await`. To launch the async world from normal code:
```python
asyncio.run(handler())   # creates an event loop, runs the coroutine, closes the loop
```

---

## Tasks — running coroutines concurrently

`await coro` runs one coroutine and waits for it. But to run several **at the same time**, you wrap them in **tasks**:

```python
task = asyncio.create_task(fetch_data())   # schedules it to run on the loop NOW
# ... do other things while it runs ...
result = await task                        # later, collect its result
```

- **`asyncio.create_task(coro)`** schedules the coroutine on the event loop immediately and returns a **Task** object — a handle to that running work.
- A **Task** is "a coroutine the event loop is actively running in the background."

🔑 **This is where two of your bugs live:** if you call `create_task` but *throw away* the returned Task (don't store it), the loop only keeps a **weak reference** to it — and it can be garbage-collected mid-run. (Full detail in lesson 03.4 — this is OpenAI #3553, litellm #29417, agno #8183.)

---

## `get_event_loop()` vs `get_running_loop()` (your CH004 / crewAI / ragas fixes)

Sometimes code needs a handle to the event loop itself (e.g. to read its clock or schedule work).

- **`asyncio.get_event_loop()`** — the OLD way. Problem: if no loop is running, its behavior is murky (it might create one, might warn, might error depending on version). **Deprecated since Python 3.10.** It emits a `DeprecationWarning` and is slated for removal.
- **`asyncio.get_running_loop()`** — the modern, correct way. Returns *the loop that is currently running*. Inside an `async def`, a loop is guaranteed to be running, so this always works and never warns.

Your fix in **crewAI #5969** and **ragas #2757**: replace `get_event_loop()` with `get_running_loop()` at sites that are inside `async` functions and only use the loop to read its clock (`.time()`) or schedule work. Safe, exact, non-deprecated.

You were also careful: where code used the `get_event_loop()` + `loop.is_running()` pattern to *detect whether a loop exists*, you **left it alone** — because `get_running_loop()` raises instead of returning when no loop is active, so it's not a drop-in there. (That judgment — fixing only the safe sites — is what made the PRs clean.)

---

## ✅ Say this in an interview

> *"`async def` creates a coroutine — calling it returns a job that doesn't run until you await it or wrap it in a task. `asyncio.create_task` schedules a coroutine to run concurrently and returns a Task handle. `get_event_loop()` is deprecated since 3.10 because its behavior with no running loop is undefined; inside a coroutine you should use `get_running_loop()`, which is exactly what I replaced in crewAI and ragas — but only at sites that were genuinely inside async functions, leaving the loop-detection pattern untouched."*

Next: the blocking-the-loop disaster.
