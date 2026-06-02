# 02.3 — Files, Resources, and the Leak You Fixed (agno #8161)

> Goal: understand what a "file handle" is, what "leaking" means, and why `with` fixes it.

---

## What happens when you open a file

```python
f = open("audio.wav", "rb")   # ask the OS for access to the file
data = f.read()               # read its contents
f.close()                     # tell the OS "I'm done"
```

`open()` doesn't just give you the data — it asks the **operating system** for a **file handle** (also called a "file descriptor"): a numbered ticket that says "this program is currently using this file." The OS keeps the file open and reserves resources for it until you `close()` it.

---

## The limit nobody thinks about: RLIMIT_NOFILE

The OS only lets each program hold a **limited number of open file handles at once** — commonly ~1024 on Linux. This limit is called `RLIMIT_NOFILE` ("resource limit: number of open files").

Every `open()` without a matching `close()` **uses up one ticket and never returns it.** That's a **leak**.

---

## 🐞 THE BUG: a leaked file handle (your agno fix)

agno's code (`OpenAITools.transcribe_audio`) did this:
```python
audio_file = open(audio_path, "rb")
transcript = client.audio.transcriptions.create(file=audio_file, ...)
return transcript
# ← audio_file is NEVER closed
```

Every time someone transcribes audio, one file handle leaks. For an agent transcribing audio **in a loop** (very common in AI apps), the leaks pile up until the program hits `RLIMIT_NOFILE` and **crashes** with "Too many open files" — often hours later, far from the real cause. Nasty to debug.

**Worse on the error path:** if `create(...)` raises an exception, the code never even reaches a close — so failures leak too.

---

## ✅ THE FIX: the `with` statement (context managers)

```python
with open(audio_path, "rb") as audio_file:
    transcript = client.audio.transcriptions.create(file=audio_file, ...)
# ← file is AUTOMATICALLY closed here, even if an exception was raised
```

The `with` block is a **context manager**. It guarantees: *whatever happens inside — success or exception — the file gets closed when the block ends.* It's Python's "clean up after yourself, always" construct.

That's your exact fix in **agno #8161**. The `with` form is non-negotiable best practice for any resource: files, network connections, locks, database sessions.

---

## How you tested it

Your reviewer (sannya-singal) pushed back on a weak test and asked for a *behavioral* one. You delivered: a test that **mocks the OpenAI client, captures the real file object handed to it, and asserts `file.closed is True` after the call** — on the success path, the error path, and across repeated calls. That proves the handle is actually closed, not just that the code "looks right." (This responsiveness is why it merged.)

---

## The general principle: resource management

A **resource** is anything you acquire and must release: file handles, sockets (network), DB connections, locks. The rule: **acquire with `with`, so release is automatic and exception-safe.** "Leaking a resource" = acquiring without releasing. Leaks are invisible until they exhaust a limit and crash the program.

---

## ✅ Say this in an interview

> *"Opening a file gives you a file handle from the OS, and there's a hard limit on how many you can hold at once. agno's transcribe_audio opened the audio file but never closed it — so an agent transcribing in a loop would slowly exhaust the file-descriptor limit and crash with 'too many open files,' and it leaked on the error path too. The fix is a `with` block, which closes the file automatically even if an exception is raised. My regression test mocks the client, captures the file object, and asserts it's closed on success, error, and repeated calls."*

Next: the AST — the core idea behind codehound.
