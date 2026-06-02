# 01.3 — Open Source Vocabulary (the words people throw around)

> Goal: never be lost when someone says "the upstream maintainer left a review on your PR."

---

## People & roles
- **Maintainer** — has merge rights on the real repo; decides what gets in.
- **Contributor** — anyone who submits a PR (you).
- **Core team** — the maintainers + trusted regulars.
- **`author_association`** — GitHub label on a comment showing the commenter's relationship: `OWNER`, `MEMBER`, `COLLABORATOR` (have power) vs `CONTRIBUTOR`, `NONE` (community, no merge power). *You used this to tell whether a comment was from a real maintainer or just a community member.*

## Places
- **Repository (repo)** — the project.
- **Upstream** — the original/official repo you contribute *to*.
- **Fork / origin** — your personal copy you contribute *from*.
- **`main` / `master`** — the primary branch; the "official" current code.
- **`dev` / staging branch** — some repos (AutoGPT, litellm) merge PRs into a `dev` branch first, not `main`.

## Things on a repo
- **Issue** — a reported bug or feature request. A discussion, not code. You often *open an issue first* describing a bug, then a PR that says "Fixes #1234" to link them.
- **"Fixes #123" / "Closes #123"** — magic words in a PR that auto-link and auto-close the issue when merged. agno's bot *required* this ("missing issue link").
- **Label** — a tag on issues/PRs (`bug`, `enhancement`, `good first issue`, `missing-issue-link`).
- **Stars (⭐)** — how many people bookmarked the repo. A rough popularity signal. agno = 25k⭐, llama_index = 40k⭐. Contributing to high-star repos looks impressive *because* more people depend on them.
- **Release / version** — a packaged, published snapshot (e.g. `huggingface_hub v1.17.0`). When your fix ships in a release, anyone who `pip install`s that version runs *your* code. That's the strongest form of "merged."

## The robots (bots) you met
- **`github-actions[bot]`** — runs the repo's automation (CI, triage).
- **`codecov[bot]`** — reports how much of your code is covered by tests.
- **`coderabbitai[bot]` / `greptile[bot]` / `cubic-dev-ai[bot]`** — AI code reviewers. When greptile said your litellm fix "fixes a silent cache-write failure," that's a bot *validating* your bug was real.
- **`pre-commit-ci[bot]`** — auto-fixes formatting and pushes a commit.
- **CLA assistant** — checks you signed the Contributor License Agreement.

## Quality & process words
- **CLA (Contributor License Agreement)** — one-time legal sign-off some projects require.
- **CI (Continuous Integration)** — auto-run checks on every PR.
- **Lint / linter** — a tool that flags style/quality issues (`ruff`).
- **Formatter** — auto-styles code consistently (`black`).
- **Type checker** — verifies type hints (`mypy`).
- **Regression test** — a test you add that would fail if someone re-introduces the bug you fixed. (You added these to most PRs — it's what separates a serious contributor from a drive-by.)
- **Conventional commit** — a commit/PR-title format like `fix: ...`, `feat: ...`, `docs: ...`. Many repos enforce it.

## Achievements
- **Pull Shark** — a GitHub badge for getting PRs merged. You earned it.

---

## ✅ Say this in an interview

> *"I learned the open-source ecosystem hands-on: issues vs PRs, how 'Fixes #123' links them, what CI and CLAs are, and how to read whether a reviewer is an actual maintainer or a community member. I also learned the unglamorous part — respecting each project's contribution rules, because some big repos like transformers and vllm reject low-value PRs, so you have to target carefully."*

Next section: the Python concepts behind the actual bugs you fixed.
