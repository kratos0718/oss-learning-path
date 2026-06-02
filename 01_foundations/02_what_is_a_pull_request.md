# 01.2 — What a Pull Request actually is

> Goal: understand the entire journey of a PR from "open" to "merged," and every check that runs on it.

---

## The one-sentence definition

A **Pull Request (PR)** is a formal proposal that says: *"Here are my changes on this branch — please review them and pull them into your project."* It's the unit of contribution in open source.

The name is literal: you are *requesting* that the maintainer *pulls* your code in.

---

## The lifecycle of your PR (what actually happens)

```
1. You push a branch + open a PR
        │
2. Automated CHECKS run (CI) ──► tests, linting, formatting
        │       (red ✗ = something broke, you fix & push again)
        │
3. A human MAINTAINER reviews ──► may approve, or "request changes"
        │
4. You address comments (push more commits)
        │
5. Maintainer APPROVES + clicks Merge ──► your code is now in the project 🎉
        │
6. The PR is "closed as merged." Your contribution is permanent.
```

You lived every step of this. On **marimo #9667**, the maintainer (kiran) requested changes, you fixed them in 6 minutes, he approved and merged. On **agno #8186**, kausmeows approved and merged with "Thanks for this!"

---

## Key terms in the PR world

| Term | Meaning |
|------|---------|
| **Maintainer** | A person with permission to merge into the real repo. The gatekeeper. |
| **Review** | A maintainer reading your diff and leaving feedback. Three outcomes: **Approve**, **Comment**, **Request changes**. |
| **Diff** | The exact lines you added (green `+`) and removed (red `-`). Reviewers read the diff, not the whole file. |
| **CI (Continuous Integration)** | Automated robots that run on every PR to check it doesn't break anything. |
| **Checks / status checks** | The individual CI jobs (run tests, check formatting, etc.). Each shows ✓ or ✗. |
| **Merge** | The maintainer accepting your change into the project. The goal. |
| **"Request changes"** | The maintainer says "fix these things first." Not a rejection — a step toward merge. |
| **Squash / rebase** | Ways of combining your commits cleanly before merge. (Detail; don't sweat it.) |

---

## What is CI / "checks"? (you saw these constantly)

When you open a PR, the project's robots automatically:
- **Run the tests** — does your change break existing behavior? (You *added* tests too — see lesson 05.2.)
- **Lint** — is the code style correct? (tools: `ruff`, `black` — lesson 05.2)
- **Type-check** — do the type hints make sense? (tool: `mypy`)

If any fail (red ✗), you can't merge until you fix it. On **litellm #29417** the `lint` check failed because of formatting — you ran `black`, pushed, and it went green. That back-and-forth *is* the job.

> **CI = the project's automated quality gate.** Green checks = "this change is safe to consider." It runs without any human.

---

## What is a CLA? (you signed a few)

A **Contributor License Agreement** is a legal form some projects (mem0, litellm, marimo, Microsoft repos) make you sign once, saying "I grant you the right to use my contribution." A bot checks it. No CLA signed = can't merge. It's a one-time, per-project (or per-org) thing.

---

## Why some PRs get rejected (you saw this too)

- **transformers** closed your PR: *"No small doc fixes — it creates noise."* Some huge projects ban tiny PRs.
- **letta** had external PRs disabled entirely.
- **vllm** bans "low-value busywork PRs" and could ban your account.

Lesson you learned the hard way: **check a project's contribution rules before investing.** Not every repo wants every fix.

---

## ✅ Say this in an interview

> *"A pull request is a proposal to merge my branch into the project. When I open one, CI automatically runs the tests, linting, and type checks — if any fail I fix and push again. Then a maintainer reviews the diff and either approves or requests changes. On my marimo feature PR, the maintainer asked me to narrow my exception handling to OSError and add tests; I addressed it within minutes and he merged it. Getting from 'open' to 'merged' is really about fast, precise responses to CI and reviewers."*

Next: the open-source vocabulary you'll hear thrown around.
