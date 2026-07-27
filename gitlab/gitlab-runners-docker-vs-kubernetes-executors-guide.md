> 📖 **Original article:** [GitLab Runners: Docker vs Kubernetes Executors Guide](https://www.valtersit.com/guides/gitlab/gitlab-runners-docker-vs-kubernetes-executors-guide/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

## Introduction

If you're still using the shared GitLab SaaS runners for anything beyond a pet project, you're gambling with your CI/CD pipeline's stability and security. I've seen teams burn two days debugging a failed deploy only to realize a shared runner was blacklisted due to cryptomining activity from another tenant. Shared runners are a honeypot.

This guide is for senior sysadmins, platform engineers, and security engineers who need to decide between self-hosted Docker executors and Kubernetes executors for GitLab CI. After reading, you'll know exactly which executor fits your team's size, workload, and tolerance for YAML. You'll also get config snippets that actually work in production — not the sanitized nonsense from official docs.

## TL;DR

:::note[TL;DR]
- Docker executor is simpler, cheaper, and ideal for teams under 20 with predictable workloads.
- Kubernetes executor provides elastic scaling and fine-grained resource control but requires a K8s operations team.
- Never run any executor with `privileged = true` unless you absolutely understand the security implications.
- Always set resource limits (`cpus`, `memory`) on both executors — a single runaway job can cripple your CI.
- Monitor your runners with Prometheus metrics and set up cache with S3-compatible storage (MinIO is fine).
:::

## Prerequisites

- A GitLab instance (self-managed or GitLab.com) with admin access to register runners.
- For Docker executor: Docker Engine installed on a Linux host (preferably not your dev machine).
- For Kubernetes executor: a Kubernetes cluster (v1.21+), `kubectl` configured, and a namespace for CI jobs.
- Basic understanding of GitLab CI configuration (`.gitlab-ci.yml`).
- A registry to store your custom helper images (optional but recommended).

## Why Not the GitLab SaaS Runners? (The Obligatory Grumpy Take)

### The cost/performance trap

Shared runners are fine for a 5-minute unit test. But when your pipeline needs 20 minutes of CPU-heavy compilation or integration tests, you're paying in queue time and unpredictable IPs. GitLab's free-tier shared runners throttle CPU after a few minutes and queue jobs behind thousands of others. I've seen builds take 45 minutes that ran in 8 on a self-hosted Docker runner.

```yaml
# .gitlab-ci.yml - force your job to use a self-hosted runner by tagging it
build:
  tags:
    - docker-runner-prod
  script:
    - make
```

If you don't add the tag, GitLab may fall back to shared runners. Always tag your jobs.

### Security isolation theater

GitLab SaaS runners run your jobs inside Docker containers, but they share the host kernel. If you're handling secrets (e.g., `CI_JOB_TOKEN` for API access) or building artifacts containing IP, that's a shared trust boundary I wouldn't cross. A teammate once had a shared runner expose the Docker socket through an environment variable — other tenants could have connected to it. Never happen again.

When you absolutely must self-host: compliance (SOC2, HIPAA), air-gapped networks, or you're just paranoid enough to be safe.

### Comparison table: GitLab SaaS runners vs self-hosted

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/gitlab/gitlab-runners-docker-vs-kubernetes-executors-guide/](https://www.valtersit.com/guides/gitlab/gitlab-runners-docker-vs-kubernetes-executors-guide/)**
