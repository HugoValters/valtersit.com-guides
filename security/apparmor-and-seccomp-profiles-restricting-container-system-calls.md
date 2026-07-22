> 📖 **Original article:** [AppArmor and seccomp Profiles: Restricting Container System Calls](https://www.valtersit.com/guides/security/apparmor-and-seccomp-profiles-restricting-container-system-calls/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# AppArmor and seccomp Profiles: Restricting Container System Calls
## A Senior DevSecOps Guide to Kernel-Level Container Hardening

**Target Audience:** Platform engineers, security engineers, Kubernetes operators  
**Tone:** Direct, opinionated, battle-hardened  
**Prerequisites:** Basic Linux kernel concepts, container runtime familiarity

---

## Introduction — Why Your Containers Have Too Much Kernel Access

I've spent 15 years watching teams treat container security as a checkbox exercise. The worst part? Docker's default seccomp profile isn't a security boundary — it's a courtesy. It's designed to not break your applications, not to stop attackers.

The uncomfortable truth is that most container escapes (CVE-2022-0492, CVE-2023-25173) exploit syscalls that neither AppArmor nor seccomp block by default. Every `unshare()`, `mount()`, or `keyctl()` call is a potential privilege escalation vector. And most teams are running with the kernel equivalent of a screen door.

Here's a quick reality check:

```bash
# Show default seccomp profile in Docker
docker run --rm -it alpine sh -c 'cat /proc/$$/status | grep Seccomp'
# Returns: Seccomp: 2 (filtered, but you'd be surprised what's allowed)
```

I once audited a production cluster where 60% of containers ran with `--privileged` because "it was faster than debugging permission issues." The CISO had a stroke. The CTO didn't care. We fixed it with profiles — but it took three months of convincing.

Here's the actionable takeaway: If you're not running custom AppArmor and seccomp profiles, you're running with training wheels on a motorcycle. One wrong turn and you're eating pavement.

:::note[TL;DR]
- Default container profiles allow 300+ syscalls — most workloads need &1 | tail -20"
```

This runs `ls` with strace and shows a syscall count. You'll see maybe 30 syscalls for a simple command. Now imagine what a web server or database needs — probably 40-60, not 300.

**Comparison: Default Docker profile vs. minimal web server profile**

| Syscall Category | Default Docker | Minimal Web Server |
|-----------------|----------------|-------------------|
| File operations | ~80 syscalls | ~20 syscalls |
| Network | ~40 syscalls | ~15 syscalls |
| Process management | ~60 syscalls | ~10 syscalls |
| IPC | ~30 syscalls | ~5 syscalls |
| Misc (dangerous) | ~90 syscalls | ~5 syscalls |
| **Total** | **~300** | **~55** |

The Docker team's "safe defaults" are designed to not break anything. That's not security — that's compatibility. If you want security, you break things intentionally during development, not in production.

### AppArmor vs. seccomp — The False Dichotomy

I see teams pick one or the other. "We use AppArmor so we don't need seccomp." That's like saying "we lock the front door so we don't need to lock the windows." Lock everything.

Here's the distinction:
- **AppArmor:** Path-based mandatory access control (MAC). Good for file system, network, and capability restrictions.
- **seccomp:** Syscall filtering at the kernel level. Good for blocking specific syscalls regardless of arguments.

The gap is critical: AppArmor can't block syscalls by name. seccomp can't restrict file paths. You need both to cover the full attack surface.

This AppArmor profile snippet restricts file system access:
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/apparmor-and-seccomp-profiles-restricting-container-system-calls/](https://www.valtersit.com/guides/security/apparmor-and-seccomp-profiles-restricting-container-system-calls/)**
