# 02.4 — The AST (the heart of codehound)

> Goal: understand what an Abstract Syntax Tree is and why codehound uses one instead of just searching text.

---

## The problem: how do you find bugs in code automatically?

Say you want to find every `time.sleep()` that sits inside an `async def` function (a real bug — lesson 03.3). Your first instinct: search the text for `time.sleep`. But text search **can't answer the questions that decide if it's a bug**:

- Is this `time.sleep` *inside an async function*? (text doesn't know about function boundaries)
- Is this call *awaited*? `await x()` is fine, `x()` is not — they differ by one keyword, but text search can't reason about structure.

Text is just characters. To reason about code, you need its **structure**. That's the AST.

---

## What an AST is

**AST = Abstract Syntax Tree.** When Python reads your code, it first parses it into a **tree** that represents the code's *structure* — what's a function, what's a call, what's inside what.

Take this code:
```python
async def f():
    time.sleep(1)
```

Python parses it into a tree like:
```
Module
└── AsyncFunctionDef (name="f")        ← "this is an async function"
    └── Expr
        └── Call                        ← "this is a function call"
            ├── func: Attribute          ← "time.sleep"
            │   ├── value: Name "time"
            │   └── attr: "sleep"
            └── args: [Constant 1]
```

Now the structure is explicit. You can *walk* this tree and ask: "Is there a `Call` to `time.sleep` whose **enclosing** node is an `AsyncFunctionDef` and whose **parent** is NOT an `Await`?" — questions text could never answer.

Python gives you this for free:
```python
import ast
tree = ast.parse(source_code)   # turns code text into the tree
for node in ast.walk(tree):     # visit every node
    if isinstance(node, ast.Call):
        ...                     # found a function call
```

---

## The key nodes (codehound uses these)

| Node | Represents |
|------|-----------|
| `ast.FunctionDef` | a normal `def` |
| `ast.AsyncFunctionDef` | an `async def` |
| `ast.Call` | a function call `foo(...)` |
| `ast.Attribute` | a dotted access `time.sleep` |
| `ast.Name` | a bare name `time` |
| `ast.Await` | an `await ...` expression |
| `ast.Assign` | an assignment `x = ...` |
| `ast.arguments` / defaults | function parameters + their defaults (for the B006 check) |

---

## The one clever trick codehound needed: the parent map

Python's AST nodes know their **children** but NOT their **parent**. Yet codehound constantly needs to walk *upward*: "what function contains this call?", "is this call's parent an `await`?".

So codehound builds a **parent map** once per file — a dictionary `{child → parent}`:
```python
parents = {}
for parent in ast.walk(tree):
    for child in ast.iter_child_nodes(parent):
        parents[id(child)] = parent
```
Now any check can climb from a node up to its function, or check if its parent is an `Await`. This single idea powers three of codehound's helpers: `enclosing_function`, `is_awaited`, `inside_with_statement`.

---

## Why AST beats regex (the interview-winning point)

> A regex/grep finds *text that looks like* a bug. An AST lets you ask *structural* questions — "is this awaited?", "what function is this in?" — that actually determine whether it's a bug. That precision is why codehound has few false positives.

In fact you *proved* this: codehound's blocking-call check first flagged awaited calls in AutoGPT as bugs (false positives), and you fixed it by adding the `is_awaited` check — an AST-level question text search literally cannot express.

---

## ✅ Say this in an interview

> *"An AST is the tree-structured representation of source code — Python's `ast` module gives it to you from `ast.parse`. I use it in codehound because finding real bugs needs structural questions text search can't answer: is this call inside an async function, is it awaited? Nodes only know their children, so I build a child-to-parent map once per file, which lets every check walk upward to find the enclosing function or check if a call is awaited. That structural precision is why the tool has a low false-positive rate."*

Next section: async Python, where most of your bugs live.
