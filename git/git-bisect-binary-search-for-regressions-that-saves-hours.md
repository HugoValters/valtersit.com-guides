> 📖 **Original article:** [Git Bisect: Binary Search for Regressions That Saves Hours](https://www.valtersit.com/guides/git/git-bisect-binary-search-for-regressions-that-saves-hours/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

A caching layer nobody touched in six weeks quietly tripled p99 latency last Tuesday. The grafana dashboard lit up at 03:11. By 04:00 the on-call had ruled out the obvious suspects — no deploy in the last 18 hours, no config change, no infra incident. By 05:30 they had checked out four commits by hand, rebuilt, redeployed to staging, and confirmed each one "looks fine." By 07:00, exhausted, they gave up and rolled back a release from three weeks prior that *probably* wasn't the cause. The regression was introduced 47 commits back by a subtle interaction between a header rename and a reverse proxy cache rule. `git blame` pointed at neither commit. `git log -p` on the cache module showed 400 diffs, none individually suspicious.

This is the shape of the problem git bisect solves. It turns a linear walk through N commits into a binary search over log₂(N). Forty-seven commits becomes 6 tests. Two hundred and fifty-six commits becomes 8. If you can express the bug as a shell predicate that returns 0 or 1, you never hunt a regression by hand again. This guide is for engineers who can already `git log` and `git blame` but reach for a manual checkout loop when things break. By the end you'll be writing `git bisect run` scripts, wiring bisect into CI on demand, and knowing exactly which failure modes produce confident garbage — and how to catch them.

:::note[TL;DR]
- Bisect is binary search over the commit DAG. If it takes N manual tests to find a regression, bisect finds it in roughly log₂(N).
- `git bisect run [script]` is the only version worth doing more than once a year. Manual bisect is a teaching tool.
- Exit codes are the entire interface: 0 good, 1–127 bad, 125 skip, 128+ abort.
- Non-monotonic bugs and flaky tests are the two failure modes that produce *wrong* answers, not just slow ones. Harden the predicate.
- CI images clone shallow by default. `git fetch --unshallow` before you try to bisect there.
:::

## Prerequisites

- Git 2.30 or newer (bisect itself is ancient, but `--first-parent` and term-renaming behavior stabilized around then — check `git --version`).
- A repository with real history. Shallow clones will not work.
- A reproducible way to run the failure. If you can't script the check, bisect cannot help you.
- For CI integration: a container image with the toolchain and enough disk to hold the object store.

## The 3 AM Problem: Someone Else's Commit Broke Your Service

The hard part of regression hunting is not the algorithm. It is that the bug is usually emergent — a behavior that arises from the *combination* of two or three innocuous commits that merged over the course of a sprint. `git blame` shows you who last touched a line, not who changed the sequence of calls that made the line slow. `git log -p path/to/module` shows you a wall of diffs where every individual change looks fine in isolation.

The manual anti-pattern — checkout, rebuild, redeploy to staging, retest, repeat — is the worst possible way to spend four hours. It is O(N) and it is usually wrong at 3 AM because humans get sloppy by commit 12.

The math is the entire argument:

| Commits between good and bad | Linear worst case | Bisect worst case | Bisect typical |
|---|---|---|---|
| 8 | 8 | 3 | 2–3 |
| 16 | 16 | 4 | 3–4 |
| 64 | 64 | 6 | 5–6 |
| 256 | 256 | 8 | 7–8 |
| 1024 | 1024 | 10 | 9–10 |

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/git/git-bisect-binary-search-for-regressions-that-saves-hours/](https://www.valtersit.com/guides/git/git-bisect-binary-search-for-regressions-that-saves-hours/)**
