> 📖 **Original article:** [Docker on ZimaBoard: Memory Limits That Actually Work](https://www.valtersit.com/guides/Zima/docker-on-zimaboard-memory-limits-that-actually-work/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# Running Docker on ZimaBoard: Memory Constraints and Practical Workarounds

I've lost count of the ZimaBoard setups I've been called in to fix. The pattern is always the same: someone buys the board, installs Portainer, spins up a dozen containers with default settings, and then wonders why Postgres gets OOM-killed at 3 AM. The answer isn't a better container. It's understanding that you're no longer on a cloud VM with 16GB of RAM and a team of SREs watching your back.

The ZimaBoard is an ARM single-board computer. It typically has 4GB to 8GB of RAM and eMMC storage. Docker's defaults are tuned for beefy x86 hosts with resources to burn. When you run Docker with those defaults on a ZimaBoard, you're asking for trouble. This guide is for anyone running Docker on memory-constrained ARM hardware who wants their containers to stay up without constant babysitting.

By the end of this, you'll know exactly which defaults to change, how to set hard memory limits that actually work, and when to admit that a workload simply doesn't belong on this hardware.

:::note[TL;DR]
- Docker's default logging and memory settings will kill your ZimaBoard — change them immediately.
- Set `--memory` and `--memory-swap` on every container. No exceptions.
- Lower `vm.swappiness` to 10 or less to protect your SD card from thrash.
- Use Alpine-based images and multi-stage builds to cut memory usage by 50% or more.
- Skip Swarm and Kubernetes. Docker Compose is all you need on a single node.
- Monitor with a simple shell script before you invest in a full observability stack.
:::

## Prerequisites

- A ZimaBoard (4GB or 8GB model) with Debian-based OS installed
- Docker Engine 24.x or newer (`docker --version` to check)
- Root access via SSH
- Basic familiarity with `docker run` and `docker-compose.yml` syntax
- A backup of any data you care about — you're about to change system settings

## Why the ZimaBoard Is Both a Blessing and a Memory Trap

The ZimaBoard is a fantastic little device. It's cheap, quiet, sips power, and can run a surprising number of services. But it is not a cloud VM. It's an ARM SBC with limited RAM, and Docker's default configuration assumes you have resources to burn.

Look at what happens on a fresh install:

```bash
$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       287Mi       3.0Gi        12Mi       510Mi       3.2Gi
Swap:          1.0Gi          0B       1.0Gi

$ docker info | grep -A 4 "Memory"
Memory: 3.8GiB
Memory Limit: 3.8GiB
Swap Limit: 1GiB
```

That 1GB swap partition? On an SD card or eMMC, it's not a safety net — it's a performance killer. When the kernel starts swapping, every read and write to that storage medium becomes a bottleneck. Docker's default logging driver writes unbounded JSON files to disk. The build cache accumulates in `/var/lib/docker` until it eats your storage. And the default `vm.swappiness` of 60 means the kernel will happily push pages to swap even when you have RAM available.

Here's what happens when you ignore this. A user runs 12 containers with default settings. No memory limits. No log rotation. No monitoring. At 3 AM, Postgres hits the OOM killer. No alerts, no backups, no graceful shutdown. Just a dead database and a corrupted volume because the container was killed mid-write. We've all seen this. It's the classic "it worked on my laptop" fallacy, except the laptop had 16GB of RAM and an SSD that could handle the swap.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/Zima/docker-on-zimaboard-memory-limits-that-actually-work/](https://www.valtersit.com/guides/Zima/docker-on-zimaboard-memory-limits-that-actually-work/)**
