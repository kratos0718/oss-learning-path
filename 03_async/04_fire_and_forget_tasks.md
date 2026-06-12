# 03.4 — Fire-and-Forget Tasks (vLLM #45249, Microsoft autogen #7825, Future AGI #821, OpenAI #3553, litellm #29417, agno #8183)

> Goal: understand the subtlest bug you fixed — discarded asyncio tasks getting garbage-collected. This is your **signature bug class** — it appears in SIX of your PRs, including **vLLM** (the LLM serving standard), **Microsoft autogen**, the official **OpenAI** Agents SDK, and your founder-invited **Future AGI** PR. Every one: a bare `create_task` whose result is dropped, so the GC can collect it mid-run and silently lose the work.

---

## First, two background concepts

### Garbage collection (GC)
Python automatically frees memory for objects nobody is using anymore. The rule of thumb: **if no variable (or anything reachable) points to an object, it becomes "garbage" and Python may delete it at any time.** This automatic cleanup is the **garbage collector**.

### Strong vs weak references
- A **strong reference** keeps an object alive. As long as one exists, GC won't delete the object.
- A **weak reference** lets you *point at* an object without keeping it alive. If only weak references remain, GC can still delete it.

Hold these two ideas — the bug is entirely about them.

---

## The trap: `asyncio.create_task` only weakly references your task

When you do `asyncio.create_task(coro)`, the event loop schedules it — but here's the killer detail, straight from Python's docs:

> *The event loop only keeps a **weak** reference to the task.*

So **you** are responsible for keeping a **strong** reference to the Task. If you don't, then:
1. You call `create_task(...)` and throw away the returned Task.
2. Nothing holds a strong reference to it.
3. The garbage collector is free to delete the Task **before it finishes running**.
4. The task silently vanishes — the work it was doing never completes. No error, no log. Just... gone.

This is called a **"fire-and-forget"** task: you fire it off and forget about it — literally, because Python forgets it too.

```python
# 🐞 BUG — result discarded, only a weak ref remains
asyncio.create_task(send_important_thing())   # may be GC'd before it runs!

# ✅ FIX — keep a strong reference until it's done
task = asyncio.create_task(send_important_thing())
my_task_set.add(task)                          # strong ref → safe from GC
task.add_done_callback(my_task_set.discard)    # remove it once finished (no leak)
```

The fix pattern — **store the task in a `set`, remove it via a done-callback** — is the canonical RUF006 fix (Ruff's code for this). It keeps a strong reference for exactly the task's lifetime, then lets go.

---

## 🐞 Your THREE bugs of this kind

### OpenAI #3553 (official OpenAI Agents SDK)
`RealtimeSession` emitted **error events** via `asyncio.create_task(self._put_event(...))` with the result discarded — in three places. If GC'd, the SDK could **silently drop the very error events it was trying to report.** You routed them through a helper that keeps the task in a set (mirroring a pattern the maintainers *already used elsewhere in the same file* for other task types). Found by codehound's CH006.

### litellm #29417
`async_set_cache` scheduled **cache writes** with fire-and-forget `create_task` — three sites. If GC'd, cache entries silently vanish → wasted money on the resulting cache misses (re-calling the LLM). **Proof it was a real bug:** litellm's *own* code in another file already kept its cache-write task referenced. greptile (an AI reviewer bot) confirmed: "fixes a silent cache-write failure." Found by codehound.

### agno #8183
The OpenTelemetry **span exporter** did fire-and-forget `create_task` for trace exports → traces could be silently dropped. Same fix (set + done-callback). *Plus* it used the deprecated `get_event_loop()` (lesson 03.2), which you also fixed. Found by codehound (two checks: CH006 + CH004).

---

## Why this bug is "senior-level" to spot

It's invisible: no crash, no error, no warning. The code *looks* correct. It only manifests as occasional, unreproducible "lost data" under specific GC timing. Catching it requires knowing the weak-reference detail of `create_task` — which is exactly why a static analyzer (codehound) that encodes that knowledge is valuable.

---

## ✅ Say this in an interview

> *"`asyncio.create_task` only keeps a weak reference to the task, so if you discard the returned handle, the garbage collector can delete the task before it finishes — the work silently disappears. I fixed this in three repos: in OpenAI's Agents SDK three discarded tasks could drop error events; in litellm fire-and-forget cache writes could silently vanish and waste spend; in agno's tracing exporter traces could be dropped. The fix is to keep a strong reference — store the task in a set and remove it with a done-callback. My tool codehound flags exactly this pattern, and it found all three."*

This is your most impressive bug class — own it cold. Next: the catalog of every PR in plain English.
