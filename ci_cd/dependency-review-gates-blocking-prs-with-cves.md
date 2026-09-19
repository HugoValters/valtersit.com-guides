> 📖 **Original article:** [Dependency Review Gates: Blocking PRs with CVEs](https://www.valtersit.com/guides/ci_cd/dependency-review-gates-blocking-prs-with-cves/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

A PR titled "chore: bump dev tooling" regenerates `package-lock.json` while resolving a conflict. Somewhere in the 4,000-line diff, a transitive file-watcher moves to `glob-parent@5.1.1` — a high-severity ReDoS. The dev never saw it. `npm audit` runs on `main` after merge, so it's green on the PR. Eleven days later the finding shows up in prod telemetry, sitting in a pile of 400 other alerts that everyone has learned to scroll past.

Nobody was negligent. The pipeline was. There was no *gate* — there was a scanner, a report, and a hope.

This is for platform and security engineers who already have scanners and want enforcement. By the end you'll have a working GitHub Actions gate, the forge-agnostic equivalent, a policy devs won't route around, and the four metrics that prove it works.

:::note[TL;DR]
- A dependency review gate is a **policy decision point wired into the merge path** — mandatory, deterministic, scoped to the diff, and reversible under explicit conditions. Not a dashboard.
- The diff matters more than the scanner. Whole-repo scanning on a 900-dependency monorepo produces 400 findings and zero action. Delta scanning produces three findings and an argument — which is the point.
- Block on **new** CRITICAL/HIGH with a fix available, malicious packages, and license violations. Never block on pre-existing `main` debt.
- Build the escape hatch before you need it. A gate with no exception path gets deleted on a Friday night during an outage — correctly.
:::

## Prerequisites

- GitHub repository with admin rights (you will be editing branch protection)
- GitHub Actions enabled; `gh` CLI authenticated (`gh auth status`)
- For the forge-agnostic path: `syft`, `grype`, `jq`, and optionally `conftest`
- An existing lockfile — this exercise is meaningless without one

## Why Your Existing Scanning Isn't a Gate

Most orgs have three scanners and zero gates. Scan-on-merge is forensics: it tells you what already happened. That's a perfectly good job — it just isn't prevention.

### The three failure modes you already have

**Scan runs on `main` post-merge.** Detection fires after blast radius is set. Worse, the finding arrives without diff context, so it's indistinguishable from 400 pieces of legacy debt and gets triaged as "pre-existing."

**Scan runs on the PR but with `continue-on-error: true`.** You have built a very expensive log file.

```yaml
# scanner.yml — a lie told in YAML
- name: Audit dependencies
  run: npm audit --audit-level=high || true
  # `|| true` guarantees exit 0. The step name is now decorative.
```

**Scan runs on the PR, fails, and the check isn't required.** A dev with admin rights merges anyway. This is the most common configuration in the wild and it is *worse* than useless, because it manufactures false confidence that the org then cites in a postmortem.

### Detection, prevention, attribution

Three different jobs, routinely conflated. **Attribution** answers "are we affected?" — SBOM plus CVE list. **Detection** answers "what's wrong right now?" — a scan. **Prevention** answers "can this bad thing enter `main`?" — a gate. Most commercial tooling sells attribution and calls it prevention. You cannot remediate what you cannot enumerate, but enumeration is the cheap part.

### What a gate actually is

Four properties, all mandatory:

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/ci_cd/dependency-review-gates-blocking-prs-with-cves/](https://www.valtersit.com/guides/ci_cd/dependency-review-gates-blocking-prs-with-cves/)**
