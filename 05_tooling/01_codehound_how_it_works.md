# 05.1 — codehound, Explained From Scratch

> Goal: explain your own tool as if learning it fresh, so you can whiteboard it in an interview. (Repo: github.com/kratos0718/codehound)

Prerequisite: read [02_python/04_the_ast.md](../02_python/04_the_ast.md) first — codehound is built on the AST.

---

## What codehound is, in one line

A **static analyzer**: a tool that reads source code **without running it** and reports likely bugs. ("Static" = analyzing the code as text/structure, as opposed to "dynamic" = watching it run.) It's like a spell-checker, but for a specific class of correctness bugs.

You built it because you kept fixing the *same kinds* of bugs by hand across big AI frameworks — so you encoded each pattern as an automatic check.

---

## The big idea: one check per bug pattern

codehound has **6 checks**, each detecting one bug class you'd actually fixed:

| Code | Detects | The PR it came from |
|------|---------|---------------------|
| **CH001** | blocking call (`time.sleep`, `requests.*`) in an `async def` | agno #8158, #8186 |
| **CH002** | mutable default argument (`def f(x=[])`) | agno #8152, mem0 #5302 |
| **CH003** | deprecated `datetime.utcnow()` | crewAI #5970 |
| **CH004** | deprecated `asyncio.get_event_loop()` | crewAI #5969, ragas #2757 |
| **CH005** | `open()` never closed | agno #8161 |
| **CH006** | fire-and-forget `create_task` | OpenAI #3553, litellm #29417, agno #8183 |

Every check = a real bug you understand (you just read the lessons for all of them).

---

## How it works (the pipeline)

```
source files
   │  1. discover .py files (skip tests/, vendored, generated dirs)
   ▼
   │  2. ast.parse each file → AST tree
   ▼
   │  3. build the parent map once (child → parent)   ← lesson 02.4
   ▼
   │  4. run each Check, which walks the tree and emits Findings
   ▼
   results: path:line:col CODE message   (exit non-zero if any → CI gate)
```

A **Finding** is just `(file, line, column, code, message)` — a precise pointer to a suspected bug.

A **Check** is a small class with a `run(tree, parents, path)` method that returns a list of Findings. Adding a new check = one new file + one line in a registry + a test. (Clean, extensible design — say that.)

---

## Example: how CH001 (blocking-in-async) actually decides

For every `Call` node in the tree, CH001 asks three structural questions (only an AST can answer these):
1. Is the called thing in my blocklist? (`time.sleep`, `requests.get`, ...)
2. Is the **enclosing function** an `AsyncFunctionDef`? (walk up via the parent map)
3. Is the call **NOT awaited**? (is its parent an `ast.Await`?)

All three true → it's a real blocking-in-async bug → emit a Finding.

That third question is the one that makes codehound *precise*. Early on, CH001 flagged `await client.post(...)` calls in AutoGPT as bugs — **false positives**, because they *were* awaited (a variable just happened to be named like the `requests` library). You added the "is it awaited?" check and the false positives vanished, while the real agno bugs were still caught. **That's the story of building precision into a tool** — a great thing to tell.

---

## "False-positive discipline" (your senior-engineer point)

A finding must be *defensible*, or the tool becomes noise people ignore. So codehound deliberately suppresses safe shapes:
- **CH001** skips awaited calls (above).
- **CH005** doesn't flag a file that's `return`ed (caller owns closing it) or explicitly `.close()`d.
- **CH006** doesn't flag `TaskGroup.create_task` (the group keeps the reference).

And every rule has **paired tests**: the bad pattern *is* flagged, the good pattern is *not*. You also rejected two real AST "hits" (in dspy and optuna) after reading the code and deciding they weren't bugs. **"A finding is a lead, not a verdict"** — verify before opening a PR.

---

## The proof it's valuable

You pointed codehound at agno and the OpenAI SDK and it found **new** bugs nobody had reported. One (agno #8186) is **merged**. That's the headline: *a tool I built found a real bug that the maintainers agreed with and merged.*

---

## ✅ Say this in an interview (whiteboard version)

> *"codehound is a static analyzer I built. It parses Python into an AST, builds a child-to-parent map so checks can ask structural questions, then runs six checks — each one a bug class I'd fixed by hand, like blocking calls in async functions or fire-and-forget asyncio tasks. The key design value is false-positive discipline: for instance the blocking-call check skips awaited calls, which I added after it false-positived on AutoGPT. I ran it against agno and OpenAI's SDK and it found new bugs — one's already merged into agno. It's about 750 lines, zero dependencies, with CI and tests."*
