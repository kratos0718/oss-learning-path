# 01.1 — Git and GitHub (from zero)

> Goal: understand every word in the sentence *"I forked the repo, branched, committed, and pushed to my fork."*

---

## The problem git solves

Imagine writing code and saving copies: `project_final.py`, `project_final_v2.py`, `project_really_final.py`. That's chaos. **Git** is a tool that tracks every change to your files over time, so you can see what changed, when, by whom — and undo anything.

**GitHub** is a website that hosts git projects online so people can collaborate. Git = the tool on your computer. GitHub = the cloud where projects live. (Think: git is Word's "track changes"; GitHub is Google Drive where the document lives.)

---

## The core nouns (learn these 8 words)

| Term | Plain meaning |
|------|---------------|
| **Repository (repo)** | A project folder that git is tracking. e.g. `agno`, `codehound`. |
| **Commit** | A saved snapshot of your changes, with a message describing them. Like a save point in a game. |
| **Branch** | A parallel line of work. The main line is usually called `main`. You make a *new* branch so your changes don't disturb `main` until they're ready. |
| **Remote** | A copy of the repo that lives somewhere else (usually on GitHub). The default remote is named `origin`. |
| **Clone** | Download a repo from GitHub to your computer. |
| **Push** | Upload your commits from your computer to a remote (GitHub). |
| **Pull** | Download new commits from a remote to your computer. |
| **Fork** | *Your personal copy* of someone else's GitHub repo, under your account. |

---

## Why fork? (this is the key to open source)

You don't have permission to change agno's official repo (`agno-agi/agno`) — only its maintainers do. So:

1. You **fork** it → now `kratos0718/agno` exists, a copy you fully control.
2. You **clone** your fork to your laptop.
3. You make a **branch**, change code, **commit**.
4. You **push** the branch to *your fork* on GitHub.
5. You open a **pull request** (next lesson) asking the real maintainers to pull your change into *their* repo.

This "fork → branch → commit → push → PR" loop is **exactly** what you did for every contribution. It's the universal open-source workflow.

```
agno-agi/agno  (the real repo — you can't push here)
      │  fork
      ▼
kratos0718/agno  (your copy — you CAN push here)
      │  clone
      ▼
your laptop  ── branch ── edit ── commit ── push ──► back to kratos0718/agno
                                                          │  pull request
                                                          ▼
                                              asks agno-agi/agno to take your change
```

---

## The commands you actually ran (decoded)

```bash
git clone https://github.com/kratos0718/agno.git   # download YOUR fork
git checkout -b fix/blocking-requests-in-discord   # make + switch to a new branch
# ... you edit files ...
git add libs/agno/.../client.py                    # stage the file (mark it to be saved)
git commit -m "fix: use async media.read()"        # save a snapshot with a message
git push origin fix/blocking-requests-in-discord   # upload the branch to your fork
```

- **`checkout -b`** = "create a new branch and move onto it." Branch names like `fix/...` describe the work.
- **`add`** = "stage" = tell git *which* changes go into the next commit. (You can edit 10 files but only commit 2.)
- **`commit -m`** = save the snapshot; `-m` is the message.
- **`origin`** = the remote name (your fork). Some of your repos used a second remote named `myfork`.

---

## Two more terms you saw constantly

- **`upstream`** — a second remote pointing at the *original* repo (`agno-agi/agno`), so you can pull in their latest changes. Convention: `origin` = your fork, `upstream` = the real repo.
- **merge** — combining one branch's changes into another. When a maintainer "merges your PR," they pull your branch into their `main`. That's the win condition.

---

## ✅ Say this in an interview

> *"Open source works through forking: I can't push to the project's real repo, so I fork it to my own account, clone that, make a branch for my fix, commit the change, push it to my fork, and open a pull request asking the maintainers to merge it. I did that loop for all my contributions — for example my agno Discord fix went fork → branch → commit → push → PR → merged."*

That single paragraph proves you understand the whole model. Next: what a pull request actually is.
