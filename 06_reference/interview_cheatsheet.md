# 06 — Interview Cheatsheet (60-second answers)

> The night before an interview, read only this file. Each answer is self-contained. Understand them (don't just memorize) by reading the linked lessons.

---

## "Tell me about your open-source work."
> *"I've had 12 PRs merged into major AI/ML projects — HuggingFace (hub shipped in a production release, plus accelerate and peft), unsloth, xorbitsai/inference, pydantic, marimo, mem0, and agno — plus open PRs to vLLM, Microsoft autogen, and OpenAI's Agents SDK, across 18+ organizations. Most are real correctness and async-safety bugs, not typos. And I built a static analyzer called codehound that found several of them — two are now merged into prestige-tier repos: unsloth (40k stars) and xorbitsai/inference."*

## "Give me a concrete example of knowing *which* fix to apply."
> *"Same bug class — a blocking requests.get freezing an async event loop — came up in two repos. In agno's Discord handler the library (discord.py) had a native async download, so I switched to await media.read(). In xorbitsai/inference there was no async equivalent available, so instead of a risky rewrite I wrapped the exact same call in await asyncio.to_thread, which offloads the blocking I/O to a thread while the loop stays free. Both merged. The skill isn't memorizing one fix — it's matching the fix to what the codebase actually gives you."*

## "What's the most interesting bug you fixed?"
> *"A blocking time.sleep inside an async function in agno's vector store. asyncio runs on a single event loop that can only switch tasks at an await — so a synchronous sleep freezes every other task for its whole duration: model streams, tool calls, the agent loop. I replaced it with await asyncio.sleep, which yields to the loop. It matched a pattern the maintainers already used elsewhere in the same file, and they merged it."*

## "What is the event loop?"
> *"It's asyncio's single-worker juggler. It runs a task until the task hits an await and starts waiting, then switches to another ready task. It's concurrency, not parallelism — one worker switching, not many running at once. The catch: it can only switch at an await, so blocking it freezes everything."*

## "Explain a fire-and-forget task bug." (your strongest — OpenAI/litellm/agno)
> *"asyncio.create_task only keeps a weak reference to the task. If you discard the returned handle, the garbage collector can delete the task before it finishes — the work silently vanishes, no error. I fixed this in three repos including OpenAI's SDK, where it could drop error events. The fix is to keep a strong reference: store the task in a set and remove it with a done-callback."*

## "What's a mutable default argument?"
> *"def f(x=[]) — the default list is created once at definition time, so every call sharing the default mutates the same list, leaking state between unrelated calls. Fix: default to None and create a fresh list inside the body. I fixed ten of these in agno and two in mem0."*

## "Why does `with open(...)` matter?"
> *"open() takes a file handle from the OS, and there's a hard limit on how many you can hold. If you don't close it, you leak handles until the program crashes with 'too many open files' — and it leaks on the error path too. A `with` block closes the file automatically even if an exception is raised. That was my agno transcribe_audio fix."*

## "What is an AST and why use it?"
> *"An Abstract Syntax Tree is the structured, tree form of source code. I use it in codehound because finding real bugs needs structural questions text search can't answer — is this call inside an async function, is it awaited? I build a child-to-parent map so checks can walk upward to find the enclosing function or check for an await."*

## "How does codehound work?"
> *"It parses Python to an AST, builds a parent map, and runs six checks — each a bug class I'd fixed by hand. The key value is false-positive discipline: the blocking-call check skips awaited calls, which I added after it false-positived on AutoGPT. I ran it on agno and OpenAI's SDK and it found new bugs; one's merged into agno."*

## "get_event_loop vs get_running_loop?"
> *"get_event_loop is deprecated since Python 3.10 — its behavior with no running loop is undefined and it warns. Inside a coroutine a loop is always running, so get_running_loop is the correct, non-deprecated call. I made that swap in crewAI and ragas, but only at sites genuinely inside async functions — I left the loop-detection pattern alone because get_running_loop raises instead of returning when no loop exists."*

## "What's a regression test?"
> *"A test written specifically to fail if a bug I fixed ever comes back. For my B006 fix I assert via inspect.signature that the default is None; for the file-handle fix I behaviorally mock the client and assert the handle is closed. It's the fix's permanent bodyguard."*

## "Tell me about a PR that got rejected." (maturity check)
> *"My transformers doc PR was closed — the maintainer said small doc fixes create noise. I took the lesson: contributing well isn't about volume, it's about respecting each project's norms and submitting changes maintainers actually want. After that I targeted real bugs in repos that welcome them, and my merge rate went way up."*

## "What did building codehound teach you?"
> *"That precision is a feature. A tool that cries wolf gets ignored, so I built in suppressions for safe patterns and paired every rule with tests for both a true positive and a true negative. And I treat every finding as a lead, not a verdict — I rejected two of its own hits after reading the code. Verify before you ship."*

---

### If you only learn 3 things, learn these (they cover ~80% of questions):
1. **The event loop + blocking it** ([03_async/01](../03_async/01_what_is_concurrency.md), [03_async/03](../03_async/03_blocking_the_loop.md))
2. **Fire-and-forget tasks / weak references** ([03_async/04](../03_async/04_fire_and_forget_tasks.md))
3. **What an AST is + how codehound uses it** ([02_python/04](../02_python/04_the_ast.md), [05_tooling/01](../05_tooling/01_codehound_how_it_works.md))
