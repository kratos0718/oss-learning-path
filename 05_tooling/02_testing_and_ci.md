# 05.2 — Testing, Linting, and CI

> Goal: explain the quality tools that appear on every PR, and what a "regression test" really is.

---

## Why tests exist

A **test** is code that checks other code does what it should. You run the tests; if they pass, you have evidence the code works. If a future change breaks something, a test fails and warns you *before* it ships.

For open source this is essential: maintainers won't merge a fix they can't verify won't break things. **Adding tests is what separated your PRs from drive-by contributions.**

---

## pytest — the test runner

Python's standard test tool. A test is just a function whose name starts with `test_` containing `assert` statements:
```python
def test_addition():
    assert 1 + 1 == 2          # passes
```
Run `pytest` and it finds and runs every `test_*` function, reporting pass/fail.

---

## What a "regression test" is (the important one)

A **regression** = when a bug you already fixed comes *back* (regresses) because someone later changed the code. A **regression test** is a test written specifically to **fail if your fixed bug ever returns.**

Your examples:
- **B006 (mem0):** `assert inspect.signature(f).parameters["messages"].default is None` — fails if someone reverts the default back to `[]`.
- **blocking sleep (agno):** uses `inspect.getsource` to assert the function no longer contains `time.sleep` — fails if reverted.
- **file handle (agno):** a *behavioral* test — mocks the OpenAI client, captures the real file object, and asserts `file.closed is True` after the call. Proves the fix *works*, not just that the code looks right.
- **fire-and-forget (OpenAI):** asserts the task is held in the tracking set while pending and released once done.

> A regression test is your fix's bodyguard: it stands watch forever so the bug can't sneak back.

### Behavioral vs structural tests
- **Structural** (`inspect.getsource` / `signature`): checks the *code shape*. Fast, but weaker — a reviewer (sannya-singal) called one "weak."
- **Behavioral**: actually runs the code and checks the *outcome* (e.g. the file really is closed). Stronger. When asked, you upgraded to behavioral — and it merged.

---

## Linters and formatters (the style robots)

- **Linter** (`ruff`) — flags quality issues: unused imports, mutable defaults (B006!), undefined names. Catches bugs *and* style.
- **Formatter** (`black`, `ruff format`) — automatically rewrites code to a consistent style (spacing, line length). No debates — the tool decides.
- **Type checker** (`mypy`) — verifies your type hints are consistent.

On **litellm #29417**, the `lint` CI check failed purely on formatting. You ran `black`, pushed, it went green. That's the daily rhythm: CI tells you what's wrong, you fix fast.

---

## CI and GitHub Actions

**CI (Continuous Integration)** = automatically running all of the above (tests, lint, type-check) on every PR. **GitHub Actions** is GitHub's built-in CI system — you describe jobs in a `.github/workflows/*.yml` file and GitHub runs them on every push.

Your codehound repo *has* a GitHub Actions workflow: it runs the tests on Python 3.9 / 3.11 / 3.12 and even **self-scans** (runs codehound on its own code). That's a sign of a serious project.

```
push to PR ──► GitHub Actions wakes up ──► runs: pytest, ruff, mypy
                                              │
                          all pass ──► green ✓ (mergeable)
                          any fail ──► red ✗ (fix & push again)
```

---

## ✅ Say this in an interview

> *"Every serious PR needs tests, especially a regression test — one written to fail if the exact bug I fixed ever comes back, like asserting via inspect.signature that a default is None, or behaviorally mocking a client and asserting the file handle is closed. CI runs these plus linting (ruff) and formatting (black) automatically on every push via GitHub Actions; when a check fails — like a black formatting failure on my litellm PR — I fix and re-push until it's green. My own tool codehound has CI across three Python versions and even scans its own source."*

Next: the quick-reference cheatsheet.
