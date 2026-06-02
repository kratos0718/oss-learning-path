# 03.3 — Blocking the Event Loop (agno #8158 and #8186)

> Goal: deeply understand your two strongest bug fixes — the blocking `time.sleep` and blocking `requests.get` inside async functions.

---

## Recap of the one fact that matters

From lesson 03.1: the event loop is **one worker** that can only switch tasks at an `await`. Between awaits, whatever runs **monopolizes the entire loop** — every other task is frozen until it finishes.

So: inside an `async def`, you must NEVER do something slow that isn't awaited. If you do, you "block the loop."

---

## What "blocking" means precisely

A **blocking call** is a normal (synchronous) function that takes time and doesn't yield control. Examples:
- `time.sleep(1)` — sleeps for 1 second, doing nothing, holding the worker.
- `requests.get(url)` — downloads over the network, synchronously, holding the worker the whole time.
- `subprocess.run(...)`, `os.system(...)` — run an external program and wait.

In *sync* code these are fine. Inside an `async def`, they're a disaster — because while they run, the single event-loop worker is stuck, and **all other tasks freeze.**

---

## 🐞 BUG 1: blocking `time.sleep` in async (agno #8158 — your strongest)

agno's Couchbase vector store had:
```python
async def _async_create_collection_and_scope(self):
    ...
    time.sleep(1)          # ← BLOCKING, inside an async function
    ...
```

That one second of `time.sleep` **freezes the entire event loop for a full second**. In an AI agent, the loop is simultaneously running: the model's token stream, tool calls, vector-DB writes, the agent loop itself. **All of them stall** every time a collection is overwritten.

### The fix
```python
await asyncio.sleep(1)     # ← the async version: yields to the loop while waiting
```
`asyncio.sleep` does the same "wait 1 second" but **as an `await`** — so the loop is free to run other tasks during that second. One-word category change, huge correctness impact.

**Bonus credibility:** a *sibling function in the same file* (`_async_wait_for_index_ready`) already used `await asyncio.sleep` correctly — so your fix matched the maintainers' own established pattern. Reviewers love that. Merged by @kausmeows: "Thanks for spotting this!"

---

## 🐞 BUG 2: blocking `requests.get` in async (agno #8186 — codehound found this one)

agno's Discord bot had:
```python
async def on_message(message):
    ...
    if media_type.startswith("video/"):
        req = requests.get(media_url)    # ← BLOCKING network call, inside async
        message_video = req.content
```

`on_message` is the Discord event handler — it runs on the event loop. `requests.get` is a **synchronous** network download. So every time someone posts a video or document, the **entire bot freezes** for the full download — no other messages, no typing indicators, no concurrent agent runs — possibly seconds for a large file.

### The fix
```python
message_video = await media.read()    # discord.py's native ASYNC download
```
`media.read()` is discord.py's built-in async method — it downloads without blocking the loop (it's awaited). **Bonus:** it also goes through the bot's authenticated session, fixing a *latent 403 error* that plain HTTP downloads of Discord URLs can hit. So one fix solved two problems. The now-unused `import requests` was removed.

This one is special: **codehound found it.** Its CH001 check flags blocking calls inside async functions. A bug your own tool discovered is now merged into a 25k⭐ repo.

---

## How do you fix blocking calls in general?

Three options:
1. **Use the async equivalent** — `await asyncio.sleep` instead of `time.sleep`; `httpx.AsyncClient` or a library's native async method instead of `requests`. (What you did.)
2. **Offload to a thread** — `await loop.run_in_executor(None, blocking_func)` runs the blocking call in a separate thread so the loop stays free. (Used in ragas's code.)
3. Redesign so the slow thing isn't on the loop at all.

---

## ✅ Say this in an interview

> *"The event loop is a single worker that can only switch tasks at an await. A synchronous call like time.sleep or requests.get inside an async function doesn't yield, so it freezes every other task for its whole duration. In agno I fixed a time.sleep in an async Couchbase routine — replacing it with await asyncio.sleep so the loop stays free — and a blocking requests.get in the async Discord handler, switching to discord.py's native await media.read(), which also fixed a latent 403. My static analyzer codehound actually found that second one."*

Next: the subtlest async bug — fire-and-forget tasks.
