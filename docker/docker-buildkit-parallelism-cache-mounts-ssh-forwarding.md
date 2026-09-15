> 📖 **Original article:** [Docker BuildKit: Parallelism, Cache Mounts, SSH Forwarding](https://www.valtersit.com/guides/docker/docker-buildkit-parallelism-cache-mounts-ssh-forwarding/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

*Part of the valtersit.com series on container build infrastructure that respects your CI budget.*

I inherited a monorepo pipeline that took 23 minutes and 40 seconds to produce an image on every push. Same base image as the one the team had been using for two years. Same dependency tree. We didn't change the runtime, didn't upgrade Alpine, didn't buy a bigger runner. We re-pointed the `COPY --from` edges, turned on two cache mounts, and wired SSH agent forwarding instead of a baked-in deploy key. The build dropped to a hair under six minutes on the same hardware — and the biggest single win, roughly eight of those seventeen minutes, came from restructuring the stage graph so the linter, the test suite, and the dependency install could actually run at the same time.

This guide is for platform and DevOps engineers who already have BuildKit available, already have `buildx` on the runner, and are still watching a linear Dockerfile pretend it's a bash script. By the end you'll know how BuildKit's DAG scheduler works, how to write a Dockerfile that exploits it, when a cache mount is the right tool (and when it's a mutex in disguise), and how to feed a private Git dependency into a build without ever writing a credential into a layer.

While you're here, let's bury three things I still see in production repositories: `ENV DOCKER_BUILDKIT=1` inside a Dockerfile (it does nothing — that variable is read by the *client* before the build starts, never by the daemon, and never from image config), `--squash` used as a layer-shrinking trick (it's experimental, it flattens history, and it nukes the cache granularity you just paid for), and committing `id_rsa` because "the layer gets deleted in the next `RUN`." That last one is not a build optimization, it's a credential disclosure with a `docker history` receipt.

:::note[TL;DR]
- BuildKit executes stages as a **DAG**, not a list — every `COPY --from=X` is a dependency edge, and siblings with no edges between them run concurrently.
- Cache mounts (`--mount=type=cache`) are **mutable volumes that survive across builds**; they are not layers and never ship in the final image.
- `--mount=type=ssh` forwards your agent socket into the `RUN` — the key material **never touches a layer**, so there's nothing to scrub.
- None of this helps a Dockerfile that does `COPY . .` before `RUN npm ci`. Fix the ordering first.
:::

**Prerequisites:** Docker 20.10 or newer, `buildx` installed (`docker buildx version` should print a version — it ships with Docker Desktop and modern Engine packages), a CI runner with either GitHub Actions or GitLab CI, and a Dockerfile you're allowed to rewrite. Everything below assumes you can create a named builder instance.

## 1. Why BuildKit — The Legacy Builder Is a Foot-Gun

The legacy builder is not slow because it's old. It's slow because it was designed in an era where a Dockerfile was a shell script with delimiters, and multi-stage builds were an afterthought bolted onto Docker 17.05. It walks instructions top to bottom, one at a time, in one stage at a time, and it has no mechanism to express "these two things don't depend on each other."

### 1.1 What the legacy builder actually does wrong

Four concrete architectural limits matter in CI:

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/docker/docker-buildkit-parallelism-cache-mounts-ssh-forwarding/](https://www.valtersit.com/guides/docker/docker-buildkit-parallelism-cache-mounts-ssh-forwarding/)**
