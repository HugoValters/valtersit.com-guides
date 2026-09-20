> 📖 **Original article:** [Git Worktrees: Multiple Working Directories Without Clones](https://www.valtersit.com/guides/git/git-worktrees-multiple-working-directories-without-clones/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# Git Worktrees: Multiple Working Directories Without Multiple Clones

A 2019 incident still lives in my postmortem template: an on-call engineer patched the payments repo in a clone sitting at `~/hotfixes/payments`, pushed `hotfix/prod-500`, and went to bed. That clone was 400 commits behind `main`. The patch applied cleanly to code that had been refactored out of production three weeks earlier. The merge produced a green CI run and a red Friday — four hours of rollback, a post-incident review, and a rule: *no more long-lived copies of the repository.*

The problem wasn't that engineer. The problem was the workflow that made a second directory of the same repo look like a normal thing to do. `git clone` twice, three times, five times — one for `main`, one for the feature, one for the hotfix, one for a review, one leftover from that migration you did in Q1. Each of them has its own object database, its own remote config, its own reflog, and its own drift from "the truth."

Git worktrees solve this. One repository, one object database, N working directories — each with its own `HEAD`, index, and checkout, and all of them sharing refs, objects, remotes, hooks, config, and stash. You get the isolation of a separate checkout with none of the duplication.

This guide is for sysadmins, SREs, and security engineers who already know Git and are tired of managing clones like pets. By the end you'll understand the on-disk layout, the exact command set you actually use, the production workflows worth building around it, and the landmines (hooks, submodules, IDE indexing, the shared stash) that will page you at 3 AM if you don't respect them.

:::note[TL;DR]
- Worktrees share one `.git` object database across many working directories. Same objects and refs everywhere; separate `HEAD`, index, and checkout per directory.
- `git worktree add`, `list`, `remove`, `lock`, `prune`, `repair` — that's the whole surface area. Learn all six.
- Hooks, stash, config, and remotes are shared. Per-worktree isolation for those does not exist. Design around that, not against it.
- Put worktrees under a single root (e.g., `~/wt/[repo]/[name]`), on the same filesystem as the parent `.git`, and script the lifecycle.
:::

**Prerequisites.** Git ≥ 2.5 for `git worktree` at all. Git ≥ 2.15 if you care about `--recurse-submodules` behaving. Git ≥ 2.20 for per-worktree config (`git config --worktree`). Git ≥ 2.23 for `git switch`, which you want anyway. Older versions will "work" until they don't — check `git --version` before you build muscle memory on a feature that was added at 2.31. A POSIX shell and a filesystem that supports the layout below (ext4, APFS, xfs — anything but a WSL `/mnt/c` mount, which cross-links `gitdir` badly).

## The Clone Tax Nobody Audits

The tell isn't a corrupt object database. It's a `~/projects/` directory that looks like this:

```bash
$ du -sh ~/projects/*
2.1G    ~/projects/payments-dev
2.1G    ~/projects/payments-hotfix
2.1G    ~/projects/payments-review
```

Six gigabytes because the repo has a bloated `.git` from years of unchecked binary commits, times three — for the same content, three times over. Every `git fetch` you forget to run in one of those three directories is a future incident. Every remote URL you edit in one is a future "wait, why is my push going to the fork?" Email. Every credential helper that re-resolves on clone is an audit finding waiting to happen.

The symptoms you'll recognize:

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/git/git-worktrees-multiple-working-directories-without-clones/](https://www.valtersit.com/guides/git/git-worktrees-multiple-working-directories-without-clones/)**
