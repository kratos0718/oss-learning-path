# 🧭 OSS Learning Path — From 0 to Defending My Contributions

**Who this is for:** me. I made real open-source contributions (12 merged into HuggingFace hub/accelerate/peft, pydantic, marimo, mem0, unsloth, xorbitsai/inference, agno; open PRs to vLLM, Microsoft autogen, OpenAI, litellm; built a static analyzer called codehound). This repo explains **every single concept** behind that work, from absolute zero, so I can *understand and defend* it — in interviews, in conversations, in my own head.

**How to read it:** top to bottom, in order. Each file builds on the last. Every term is defined the first time it appears. Nothing is assumed.

---

## The path (read in this order)

### 01 — Foundations (what even is open source?)
- [`01_foundations/01_git_and_github.md`](01_foundations/01_git_and_github.md) — git, commit, branch, remote, fork, clone, push, pull
- [`01_foundations/02_what_is_a_pull_request.md`](01_foundations/02_what_is_a_pull_request.md) — PR, merge, review, CI, the whole flow
- [`01_foundations/03_open_source_glossary.md`](01_foundations/03_open_source_glossary.md) — maintainer, upstream, CLA, issue, "good first issue", stars

### 02 — Python you need to understand the bugs
- [`02_python/01_how_python_runs.md`](02_python/01_how_python_runs.md) — interpreter, modules, imports, the call stack
- [`02_python/02_functions_args_defaults.md`](02_python/02_functions_args_defaults.md) — arguments, defaults, the mutable-default trap (your B006 fixes)
- [`02_python/03_files_and_resources.md`](02_python/03_files_and_resources.md) — file handles, `open`, `with`, what "leaking" means (your file-handle fix)
- [`02_python/04_the_ast.md`](02_python/04_the_ast.md) — what an Abstract Syntax Tree is (the heart of codehound)

### 03 — Async Python (most of your bugs live here)
- [`03_async/01_what_is_concurrency.md`](03_async/01_what_is_concurrency.md) — concurrency vs parallelism, the event loop
- [`03_async/02_asyncio_basics.md`](03_async/02_asyncio_basics.md) — `async`/`await`, coroutines, tasks
- [`03_async/03_blocking_the_loop.md`](03_async/03_blocking_the_loop.md) — why `time.sleep`/`requests` in async is a disaster (agno #8158, #8186)
- [`03_async/04_fire_and_forget_tasks.md`](03_async/04_fire_and_forget_tasks.md) — weak references, garbage collection, dropped tasks (OpenAI #3553, litellm #29417, agno #8183)

### 04 — Your actual bugs, explained one by one
- [`04_the_bugs/README.md`](04_the_bugs/README.md) — every merged/open PR, what it was, why it mattered, in plain English

### 05 — The tooling you used
- [`05_tooling/01_codehound_how_it_works.md`](05_tooling/01_codehound_how_it_works.md) — your own tool, explained as if you're learning it fresh
- [`05_tooling/02_testing_and_ci.md`](05_tooling/02_testing_and_ci.md) — pytest, regression tests, ruff, GitHub Actions

### 06 — Quick reference
- [`06_reference/interview_cheatsheet.md`](06_reference/interview_cheatsheet.md) — 60-second answers to "explain X"
- [`06_reference/glossary.md`](06_reference/glossary.md) — every term, one-line definition, A–Z

---

## How to use this in an interview

When someone asks *"tell me about your open-source work,"* you don't recite — you **explain the bug**. Every file here ends with a **"Say this in an interview"** box: the exact 2–3 sentences that show you understand it. Learn those, understand the rest, and you'll out-explain people who've been coding for years.
