> 📖 **Original article:** [eBPF Network Observability: Zero-Install Kernel Tools](https://www.valtersit.com/guides/linux/ebpf-network-observability-zero-install-kernel-tools/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

It's 3:17 AM. Your phone vibrates with a PagerDuty alert: `prod-db-02` is showing a 400% network spike. You SSH in, run `ss -tunap`, and see a wall of established connections. `tcpdump` shows SYN packets going out, but no SYN-ACKs coming back. The database is up, the app is up, and everything looks fine — except it's not. You spend the next three hours restarting services, checking iptables rules, and eventually giving up and rebooting the box.

Sound familiar? That was me, six years ago. The tools I had — `tcpdump`, `ss`, `netstat` — told me *what* was happening but not *why*. To get answers, I'd have needed to install a commercial agent or compile a kernel module that would break on the next kernel update. Neither was acceptable on a production database server.

Then I discovered eBPF, and the game changed forever.

This guide is for sysadmins who need real answers about their network stack without turning their servers into a testing ground for vendor agents. By the end, you'll know how to trace TCP connection drops, identify bandwidth hogs per-process, and debug syscall-level issues — all with tools that are already sitting in your kernel.

:::note[TL;DR]
- eBPF is already in your kernel — you don't install it, you *use* it
- `bpftrace` is your new `awk` for kernel-level debugging
- Tools like `tcpdrop`, `tcplife`, and `tcpconnect` solve real production problems with zero installation
- eBPF adds 1-3% CPU overhead for observability, vs 10-20% for kernel modules
- Use eBPF for incident response, not permanent monitoring
:::

## Prerequisites

- Linux kernel 5.8+ (check with `uname -r`)
- Root or `CAP_BPF`/`CAP_PERFMON` capabilities
- `bpftool` (usually in `linux-tools-common` package)
- `bpftrace` for the one-liner examples
- BCC tools if your distro ships them (`bpfcc-tools` on Ubuntu/Debian)

## Introduction — The "I'm Not Installing That" Problem

The old reality of network observability was grim. To get per-process network visibility, you had two options: install a heavyweight agent (which required a vendor account, a license key, and a prayer that it wouldn't conflict with your existing monitoring), or compile a kernel module that would inevitably break on the next kernel update. Both were liabilities.

The new reality: eBPF (Extended Berkeley Packet Filter) has been in your kernel since 2014, and the tooling has been production-ready since kernel 5.8 (2020). You don't install eBPF — you invoke it. The kernel already has the virtual machine, the verifier, and the hooks. The tools that speak to it are either already installed or one `apt install` away.

Here's how you confirm your kernel is ready:

```bash
# Check your kernel version — 5.8+ means you're golden
uname -r
# 6.8.0-45-generic

# Check for BTF (BPF Type Format) support
ls /sys/kernel/btf/vmlinux
# /sys/kernel/btf/vmlinux

# Check bpftool is available
bpftool version
# bpftool v7.4.0
```

If that last command failed, install it: `apt install linux-tools-common` or `yum install bpftool`. That's the *only* installation you'll need.

Here's the comparison table that should convince you:

| Tool | Installation | Kernel Risk | Data Granularity | Overhead |
|------|-------------|-------------|------------------|----------|
| tcpdump | Native | Low | Packet-level | High |
| netstat/ss | Native | None | Connection-level | Low |
| Kernel module agent | Vendor installer | High | Varies | High |
| eBPF tools | None (in-kernel) | Low (sandboxed) | Event-level | Low |

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/linux/ebpf-network-observability-zero-install-kernel-tools/](https://www.valtersit.com/guides/linux/ebpf-network-observability-zero-install-kernel-tools/)**
