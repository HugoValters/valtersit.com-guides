> 📖 **Original article:** [Distroless Docker Images: Shrink Your Attack Surface](https://www.valtersit.com/guides/docker/distroless-docker-images-shrink-your-attack-surface/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# Building Distroless Docker Images: Minimizing Attack Surface in Production

You've just spent your weekend patching 47 images because Log4Shell exploited a vulnerability in a dependency you didn't even know you had. Sound familiar? The `node:18-slim` image you've been shipping to production contains a package manager, a shell, and enough tooling to make a penetration tester's day. Every one of those components is an attack surface you don't need.

This guide is for senior DevOps and SRE engineers who've been burned by supply chain attacks and are ready to stop playing whack-a-mole with CVEs. After reading, you'll understand exactly what distroless images are, how to build them with multi-stage builds, and how to harden them for production without losing your ability to debug.

:::note[TL;DR]
- Distroless images contain only your application and its runtime dependencies — no shell, no package manager, no compilers
- Alpine's musl libc breaks native Python wheels and causes "works locally" production failures
- Multi-stage builds are non-negotiable: build in a fat image, copy only the binary to distroless
- Pin every base image by digest, not tag, and generate SBOMs in CI
- Non-root users, dropped capabilities, and seccomp profiles are mandatory complements to distroless
:::

## Prerequisites

- Docker 24+ with BuildKit enabled (`DOCKER_BUILDKIT=1`)
- A containerized application you're willing to restructure
- Grype or Trivy for vulnerability scanning
- Dive for image inspection
- Kubernetes cluster with `kubectl debug` support if you need ephemeral debugging

## Introduction — Why Your "Slim" Image Isn't Actually Secure

Let's be brutally honest: that `node:18-slim` image you're shipping is not secure. It's just smaller than the full Ubuntu image. The dirty secret is that Alpine-based images — the darling of the "I want a small image" crowd — use musl libc instead of glibc. This breaks native Python wheels, causes "works locally" production failures, and creates a compatibility nightmare that you'll discover at 2 AM during an incident.

The "curl | bash" mentality that plagues Dockerfiles — installing debugging tools, build dependencies, and random packages into production images — is how you end up with 200MB containers that contain 150 CVEs. Here's the typical "production" Dockerfile that should embarrass you:

```dockerfile
# The typical "production" Dockerfile that should embarrass you
FROM node:18-slim
RUN apt-get update && apt-get install -y curl wget git python3
COPY . /app
CMD ["npm", "start"]
```

What's actually in that image? A shell, a package manager, curl, wget, git, Python, and every transitive dependency that apt decided to pull in. If an attacker gets code execution in that container, they have everything they need for lateral movement.

During Log4Shell remediation, I watched a team spend 72 hours patching 47 images because every single one had a package manager and a shell. The remediation wasn't just updating the log4j dependency — it was rebuilding, re-scanning, and re-deploying every service. If those images had been distroless, the attack surface would have been dramatically smaller, and the remediation would have been a single base image rebuild.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/docker/distroless-docker-images-shrink-your-attack-surface/](https://www.valtersit.com/guides/docker/distroless-docker-images-shrink-your-attack-surface/)**
