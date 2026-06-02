# 02.1 — How Python Actually Runs

> Goal: build the mental model you need before the bug lessons make sense.

---

## What "running Python" means

When you run `python script.py`, the **Python interpreter** reads your file top to bottom and executes it line by line. There's no separate "compile then run" step you manage — the interpreter does it for you.

Key consequence: **code at the top level of a file runs when the file is imported.** This matters (you'll see why with mutable defaults).

---

## Modules and imports

- A **module** is just a `.py` file.
- `import requests` tells Python: "find the `requests` module and make its contents available."
- `from datetime import datetime` imports one specific thing from a module.

When you `import` something, Python *runs that module's top-level code once* and remembers it. Import the same thing twice → it's only executed the first time.

**Why this matters for your work:** agno's code does `import requests` at the top. Your Discord fix *removed* that import because after switching to `await media.read()`, `requests` was no longer used — and linters flag unused imports.

---

## The call stack (functions calling functions)

When function A calls function B, Python "pushes" B onto a **stack**. When B finishes, it "pops" off and control returns to A. This stack of "who's currently running" is the **call stack**. Errors show you a **traceback** = a printout of the call stack at the moment something broke.

You don't need to master this — just know "the call stack" = the chain of function calls currently in progress.

---

## Variables hold references, not values (important!)

In Python, a variable doesn't "contain" a list — it **points to** a list object in memory.

```python
a = [1, 2, 3]
b = a          # b points to the SAME list as a
b.append(4)
print(a)       # [1, 2, 3, 4]  ← a changed too! they're the same object
```

This "two names, one object" idea is the entire reason the **mutable default argument** bug exists (next lesson). Lock it in: **assignment copies the *pointer*, not the object.**

---

## Mutable vs immutable

- **Immutable** = can't be changed after creation: `int`, `str`, `tuple`, `bool`. To "change" them you make a new object.
- **Mutable** = can be changed in place: `list`, `dict`, `set`. `list.append()` modifies the *existing* object.

This distinction is the whole ballgame for your B006 fixes. Mutable defaults are dangerous *because* they can be changed in place and that change persists.

---

## ✅ Say this in an interview

> *"Python variables are references to objects, not the objects themselves — so two variables can point to the same mutable list, and changing it through one is visible through the other. Plus, top-level and default-argument code runs once at definition time, not per call. Those two facts together explain the mutable-default-argument bug I fixed in agno and mem0."*

Next: functions, arguments, and that exact bug.
