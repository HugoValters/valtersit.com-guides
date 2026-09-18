> 📖 **Original article:** [Docker on ZimaBoard: Memory Constraints & Workarounds](https://www.valtersit.com/guides/zima/docker-on-zimaboard-memory-constraints-workarounds/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

The ZimaBoard is genuinely good hardware. It's also a 6-watt passively cooled x86 box with soldered RAM and an eMMC that will die before your warranty does. Docker will run on it fine. Docker will also cheerfully OOM-kill your reverse proxy at 3 AM because you let a Jellyfin container have 1.2 GB of headroom it never asked for.

This guide is for homelab operators, self-hosters, and junior infra folks who bought a fanless x86 board expecting a Raspberry Pi with better I/O — and got a memory ceiling they can't upgrade. By the end you'll understand why three specific failure modes happen (**OOM-kill roulette**, **eMMC death by swap churn**, and **build-time OOM on the host**), and you'll have hard limits, zram tuned correctly, and a workload-sizing strategy that keeps a 2 GB box serving useful work instead of thrashing itself to death.

Every recommendation below is a tradeoff. I'll name the tradeoff. There's no getting-started fluff — I assume Debian/Ubuntu/CasaOS is installed and `docker` runs.

:::note[TL;DR]
- ZimaBoard RAM is soldered. You lose **300–400 MB** to kernel + dockerd + containerd + journald before your first container even starts.
- Never put a swapfile on eMMC. Use **zram** with a hard cap, and understand zram buys latency, not free memory.
- `deploy.resources.limits.memory` is often silently ignored in Compose. Use `mem_limit` on non-Swarm hosts.
- Set `memory.high` slightly below `memory.max` so you get throttle + PSI pressure, not a silent kill.
- Build images off the board. BuildKit's default parallelism will OOM-kill a 2 GB Apollo Lake host.
:::

## Prerequisites

- ZimaBoard 216, 432, or 832 running Debian 12+, Ubuntu 22.04+, or a CasaOS base.
- Docker Engine 24+ with cgroup v2 enabled (verify in Section 3).
- Root or `sudo` access for sysctl, systemd unit overrides, and cgroup inspection.
- A SATA SSD or NVMe (via USB) if you plan to run anything stateful. This is not optional for anything except throwaway tests.

## Section 1 — Know What You Actually Bought

The ZimaBoard is not a Raspberry Pi. It's an Apollo Lake x86 SoC — Goldmont cores, no SMT, no ECC, DDR4 soldered to the board. You cannot add RAM. Ever. The memory you ordered is the memory you get for the life of the device.

### The SKU table matters

| Model | CPU | RAM | eMMC | Realistic container count | What will actually fit |
|---|---|---|---|---|---|
| **216** | Celeron N3350 (2c) | 2 GB | 16 GB | 3–5 tiny | AdGuard, Caddy, a wiki. No DB. |
| **432** | Celeron N3350 (2c) | 4 GB | 32 GB | 6–10 small | Reverse proxy, small Postgres + PgBouncer, monitoring |
| **832** | Celeron N3450 (4c) | 8 GB | 32 GB | 12–20 medium | Media server (with QuickSync), Home Assistant, light JVM |

### Baseline memory accounting before you install anything

People consistently under-estimate the tax the plumbing applies. In testing on a fresh Debian 12 install with Docker installed and zero containers running, I typically see:

| Component | Typical RSS |
|---|---|
| Kernel + minimal userspace | 130–170 MB |
| `dockerd` | 60–90 MB |
| `containerd` | 25–40 MB |
| `docker-proxy` / shims (idle) | 20–50 MB |
| `systemd-journald` | 30–60 MB |
| **Total** | **~300–400 MB** |

That's your 2 GB board down to ~1.6 GB of *actual* headroom before workloads. Now install CasaOS, Portainer, and Watchtower and you're at 640 MB consumed at idle — before a single service runs. The 216 is not a "small home server." It's an appliance box for three well-behaved containers.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/zima/docker-on-zimaboard-memory-constraints-workarounds/](https://www.valtersit.com/guides/zima/docker-on-zimaboard-memory-constraints-workarounds/)**
