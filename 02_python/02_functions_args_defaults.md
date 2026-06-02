# 02.2 — Functions, Arguments, and the Mutable-Default Trap (your B006 bug)

> Goal: fully understand the bug behind your agno #8152 and mem0 #5302 fixes.

---

## Function basics

```python
def greet(name):          # 'name' is a parameter
    return f"Hello {name}"

greet("Abhinav")          # "Abhinav" is an argument
```
- **Parameter** = the variable in the definition.
- **Argument** = the actual value you pass in.

## Default arguments

You can give a parameter a fallback value:
```python
def greet(name="friend"):
    return f"Hello {name}"

greet()          # "Hello friend"  ← used the default
greet("Abhinav") # "Hello Abhinav" ← overrode it
```

---

## 🐞 THE BUG: mutable default arguments (flake8-bugbear code "B006")

Here's the trap. Watch closely:

```python
def add_item(item, basket=[]):   # ← basket defaults to an empty list
    basket.append(item)
    return basket
```

Looks innocent. But:
```python
add_item("apple")     # ['apple']        ... fine
add_item("banana")    # ['apple', 'banana']  ← WHAT? where did apple come from?!
```

### Why this happens (the key insight)

**The default value `[]` is created exactly once — when the function is *defined*, not each time it's called.** (Remember lesson 02.1: defaults run at definition time.)

So there is **one single list object** shared by every call that uses the default. Each call appends to that *same* list. State leaks across completely unrelated calls.

```
Definition time:  basket ──► [] (one list, lives forever)
Call 1:           appends "apple"  ──► that list is now ['apple']
Call 2:           appends "banana" ──► SAME list is now ['apple','banana']
```

In a real library like agno, this means: two different users, two unrelated requests, accidentally **sharing and corrupting each other's data.** Subtle, dangerous, and silent — no error is ever raised.

---

## ✅ THE FIX (exactly what you did)

Default to `None`, then create a fresh object inside the body:

```python
def add_item(item, basket=None):
    if basket is None:
        basket = []          # a NEW list every call — no sharing
    basket.append(item)
    return basket
```

Now each call with no `basket` gets its own brand-new list. Bug gone.

This is **precisely** your fix in:
- **agno #8152** — 10 such defaults across toolkits (`Toolkit`, `Searxng`, `MCPToolbox`, pdf reader).
- **mem0 #5302** — `Completions.create(messages=[])` and `BaseEmbedderConfig.__init__(azure_kwargs={})`.

The same applies to `{}` (dict) and `set()` — any mutable default.

---

## How you tested it (the regression test)

You used Python's `inspect` module to check the *signature* — proving the default is now `None`:
```python
import inspect
sig = inspect.signature(Completions.create)
assert sig.parameters["messages"].default is None   # fails if someone reverts the fix
```
This is a **regression test**: it would fail if anyone re-introduced the bug. (More in lesson 05.2.)

---

## Why it's called "B006"

`flake8-bugbear` is a linter plugin; it assigns codes to bug patterns. `B006` = "mutable data structure as a default argument." Ruff (a faster linter) also implements it. Knowing the code name makes you sound fluent.

---

## ✅ Say this in an interview

> *"A mutable default argument like `def f(x=[])` is a classic Python bug: the default list is created once at function-definition time, so every call that uses the default shares the same list — state silently leaks between unrelated calls. The fix is to default to None and build a fresh list inside the function. I fixed ten of these in agno and two in mem0, and added regression tests using inspect.signature so the fix can't be reverted unnoticed."*

Next: file handles and the leak you fixed.
