> 📖 **Original article:** [Linux Kernel Tuning: sysctl Parameters That Matter](https://www.valtersit.com/guides/linux/linux-kernel-tuning-sysctl-parameters-that-matter/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# Linux Kernel Tuning for High-Traffic Servers

It's 03:14 and a Kubernetes node has been flapping for forty minutes. Pods are restarting, Istio sidecars are dropping health checks, and the only thing in `dmesg` is a two-word sentence repeated a thousand times: `nf_conntrack: table full, dropping packet`. The on-call engineer redeploys the affected service — traffic recovers — and everyone goes back to bed. Two weeks later it happens again. The redeploy "fixed" nothing. The conntrack table filled back up.

That story is why this article exists. Every "Linux tuning guide" on the internet is a copy-paste of a 2012 blog post that recommended `tcp_tw_recycle` and `swappiness=0` for everything. The people following those guides aren't tuning — they're cargo-culting, and they find out which knob was wrong during an incident.

This guide is for sysadmins, SREs, and platform engineers running Linux at real traffic levels — 10G+ NICs, tens of thousands of concurrent connections, containers and sidecars. After reading it you'll be able to build a measurement-driven tuning pass for the network stack, memory subsystem, file descriptor limits, and block layer, and you'll know which parameters have a failure mode that will bite you.

Three rules before we touch anything: **baseline before you change**, **understand the subsystem you're changing**, and **every parameter has a failure mode**. If you can't state the metric you expect to improve, you're not tuning.

:::note[TL;DR]
- Distro defaults are tuned for worst-case workloads. They're conservative on purpose — the accept queue, conntrack table, and socket buffers are the three that bite hardest in production.
- `net.core.somaxconn`, `nf_conntrack_max`, and `net.core.rmem_max` are the highest-yield changes for a busy web/proxy node. Most other knobs are noise.
- `tcp_tw_recycle` was removed in Linux 4.12 because it broke NAT. If a guide recommends it, close the tab.
- `/etc/security/limits.conf` is ignored by systemd services. Use `LimitNOFILE=` in a unit drop-in instead.
- Every change should be paired with a counter you watch: `nstat`, `ss -lnt`, `conntrack -C`, `mpstat -P ALL`.
:::

## Prerequisites

You need root (or sudo) on the target host, a kernel 4.12 or newer for anything TIME_WAIT related, and ideally a maintenance window or a canary node. The tooling below is either preinstalled or available in `iproute2`, `procps-ng`, `sysstat`, and `conntrack-tools`:

```bash
apt-get install -y iproute2 procps sysstat conntrack ethtool linux-tools-common   # Debian/Ubuntu
dnf install -y iproute procps-ng sysstat conntrack-tools ethtool perf            # RHEL family
```

`perf`, `trace-cmd`, and `bpftrace` are optional but invaluable when you need to prove that softirq processing — not your application — is the bottleneck. And before anything else, capture the baseline:

```bash
sysctl -a > /root/baseline-$(uname -r)-$(date +%F).txt
nstat -az > /root/nstat-$(date +%F).txt
ss -s > /root/ss-summary-$(date +%F).txt
```

That `sysctl -a` dump is your rollback plan. You will use it more often than you think.

## 1. The Case Against Copy-Pasting `sysctl.conf`

### 1.1 Why Distro Defaults Are Conservative

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/linux/linux-kernel-tuning-sysctl-parameters-that-matter/](https://www.valtersit.com/guides/linux/linux-kernel-tuning-sysctl-parameters-that-matter/)**
