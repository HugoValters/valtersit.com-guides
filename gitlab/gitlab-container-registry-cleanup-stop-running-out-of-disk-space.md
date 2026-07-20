> 📖 **Original article:** [GitLab Container Registry Cleanup: Stop Running Out of Disk Space](https://www.valtersit.com/guides/gitlab/gitlab-container-registry-cleanup-stop-running-out-of-disk-space/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I got the call at 2:47 AM on a Saturday. The on-call engineer was panicking: GitLab CI had stopped working across three critical projects. The root cause? The container registry had filled its 2TB object store with untagged images from CI pipelines that ran 18 months ago. "But storage is cheap," the architecture decision from 2024 said. Tell that to the production deployment that couldn't push a security patch because the registry was full.

This guide is for senior engineers managing GitLab instances with 50+ projects. You'll learn how to implement automated cleanup policies that prevent registry bloat, recover from accidental deletions, and enforce retention rules across your entire organization — without waking up at 3 AM.

:::note[TL;DR]
- GitLab's default registry configuration has zero cleanup — you **will** run out of disk space
- Per-project cleanup policies are your first line of defense: set `keep_n` to at least 5 and `older_than` to 30 days today
- Never rely on `:latest` tags for production — use semver or commit SHA patterns with regex
- Build a scheduled CI/CD pipeline for cleanup because GitLab's built-in scheduler is unreliable
- Implement object storage versioning because GitLab has no "undo" for registry deletions
:::

## Prerequisites

Before implementing anything in this guide, you need:
- GitLab instance (self-managed or GitLab.com with Premium/Ultimate for some features)
- Admin or Maintainer access to projects you're configuring
- `curl`, `jq`, and `gitlab-rails` console access (self-managed only)
- Object storage configured (S3, GCS, or Azure Blob) — NFS users, your pain is real but this guide still applies
- Python 3.8+ for custom cleanup scripts

## Why Your Registry is a Hoarder's Paradise (and Why That's Your Fault)

### The "But Storage is Cheap" Lie

Storage is cheap. Operations at scale are not. That 2TB registry I mentioned earlier? It cost $40/month in S3 storage. The real cost was:
- **Backup times**: 6 hours to snapshot the registry volume
- **Replication lag**: 45 minutes to replicate across regions
- **Image pull latency**: 12 seconds average because the registry was scanning millions of blobs
- **Vulnerability scanning queues**: 3-day backlog because every stale image with Log4j was being scanned

Here's how to check if you're heading toward the same disaster:

```bash
# Check total registry storage from GitLab Rails console
gitlab-rails console
> Project.all.map { |p| p.container_registry_enabled ? p.container_repositories.sum(&:size) : 0 }.sum
=> 2147483648000  # That's 2TB in bytes — yes, I've seen this

# List top 10 projects by registry size
> Project.joins(:container_repositories)
         .group(:id)
         .order('SUM(container_repositories.size) DESC')
         .limit(10)
         .pluck(:path_with_namespace, Arel.sql('SUM(container_repositories.size)'))
```

### When Retention Policies Become Incident Reports

A single CI pipeline generating 50 images per day across 20 projects adds up fast: 50 × 20 × 365 = 365,000 untagged images per year. Each image averages 200MB. That's 73TB of garbage you're paying to store and scan.

The security implications are worse than the storage costs. Every stale image in your registry is a potential attack vector with known CVEs. I've seen penetration testers exploit a 2-year-old Node.js image that was still pullable by tag because "we might need it for rollback."

### GitLab's Default Behavior (or Lack Thereof)

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/gitlab/gitlab-container-registry-cleanup-stop-running-out-of-disk-space/](https://www.valtersit.com/guides/gitlab/gitlab-container-registry-cleanup-stop-running-out-of-disk-space/)**
