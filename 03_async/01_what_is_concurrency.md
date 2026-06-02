# 03.1 — Concurrency and the Event Loop (the foundation)

> Goal: understand the machine your async bugs break. Get this and the rest is easy.

---

## The problem async solves

A lot of what programs do is **waiting**: waiting for a network response, a file to load, a database to answer. While waiting, the CPU is idle — wasted.

Imagine a chatbot server handling 100 users. Each user's request spends most of its time *waiting* for the AI model to respond. If the server handles them one-at-a-time, 99 users sit in line while one waits. Terrible.

**Async** lets a single program juggle many waiting tasks: while task A waits for the network, the program switches to task B, then C, and comes back to A when its data arrives. One worker, many jobs in flight.

---

## Concurrency vs parallelism (people confuse these)

- **Parallelism** = doing many things *at literally the same instant*, on multiple CPU cores. (Like 4 chefs cooking 4 dishes simultaneously.)
- **Concurrency** = one worker *switching between* many tasks so they all progress, but only one runs at any instant. (Like 1 chef who starts the rice, then chops veggies while it boils, then stirs the curry — juggling, not cloning.)

**asyncio is concurrency, not parallelism.** One thread, one worker, rapidly switching between tasks whenever one is waiting. This single-worker fact is *why* blocking it is catastrophic (next lessons).

---

## The event loop (the heart of it all)

The **event loop** is the juggler. It's a loop that:
1. Picks a task that's ready to run.
2. Runs it until the task says "I'm now waiting for something" (via `await`).
3. Sets that task aside, picks another ready task.
4. When a waited-for thing is ready (network replied), marks that task ready again.
5. Repeats forever.

```
        ┌─────────── EVENT LOOP (one worker) ───────────┐
        │  run Task A  →  A hits `await network` (waits) │
        │  run Task B  →  B hits `await db` (waits)       │
        │  A's network replied → run A to completion      │
        │  B's db replied → run B to completion           │
        └─────────────────────────────────────────────────┘
```

**Critical insight:** the loop can only switch tasks at an `await`. A task only "yields control" when it hits an `await` and starts waiting. Between awaits, **a task hogs the entire worker.**

That last sentence is the seed of nearly every async bug you fixed:
- If a task does something slow that *isn't* an `await` (like `time.sleep` or a blocking `requests.get`), the loop **can't switch** — every other task freezes. (Lesson 03.3)
- If you start a task but don't keep a reference to it, the loop may forget it exists. (Lesson 03.4)

---

## ✅ Say this in an interview

> *"asyncio is concurrency, not parallelism — a single event loop that juggles many tasks by switching between them whenever one is waiting on I/O. The catch is that it can only switch at an `await`; between awaits, whatever's running monopolizes the one worker. That's the foundation for the async bugs I fixed: a blocking call with no await freezes the whole loop, and a task you don't hold a reference to can get dropped."*

Next: the actual async/await syntax and what a "task" is.
