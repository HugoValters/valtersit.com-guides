> 📖 **Original article:** [Proxmox ZFS ARC: Why Your Hypervisor is Starving VMs of Memory](https://www.valtersit.com/guides/databases/Proxmox-ZFS-ARC-Why-Your-Hypervisor-is-Starving-VMs-of-Memory/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I’ve seen it happen to every single person who builds their first "serious" Proxmox node. They spend $2,000 on a shiny new server with 128GB of high-speed ECC RAM. They install Proxmox on a ZFS mirror, boot up three tiny Linux VMs that should be using maybe 8GB of RAM total, and then they log into the web UI only to see a terrifying 95% memory utilization on the host.

Their first instinct is panic. "Is there a memory leak in KVM? Did I misconfigure the ballooning driver? Is Proxmox just a bloated mess?" 

The answer is none of the above. What you’re seeing is the **ZFS Adaptive Replacement Cache (ARC)** doing exactly what it was designed to do: eat as much memory as physically possible to speed up your disk I/O. The problem is that while this behavior is fantastic for a dedicated NAS like TrueNAS, it is a catastrophic default for a hypervisor where your primary goal is to run virtual machines with guaranteed, non-swappable memory.

### The Mechanics: The "Adaptive" Greed of ZFS

To understand why your RAM has disappeared, you have to understand that ZFS is not just a filesystem; it’s a volume manager and a caching engine rolled into one. At the heart of its performance is the ARC. 

Most filesystems use a simple "Least Recently Used" (LRU) cache. ZFS is smarter. The ARC tracks both "Most Recently Used" (MRU) and "Most Frequently Used" (MFU) data. It balances these two lists dynamically to ensure that the data you need is almost always sitting in your fast RAM rather than on your slow mechanical disks or even your NVMe drives.

#### The Proxmox Default
By default, the ZFS implementation on Linux (OpenZFS) is configured to use up to **50% of your total physical RAM** for the ARC. 

On a 128GB system, ZFS will happily claim 64GB of RAM the moment you start doing any significant disk I/O—like installing an OS, running a backup, or just starting up a handful of VMs. To the Linux kernel, this memory is technically "available" because ZFS is supposed to release it if the system needs it for something else. 

#### The Hypervisor Conflict
Here is where the theory falls apart in production. Virtual machines (KVM/QEMU processes) are very "heavy" memory users. When you start a VM with 16GB of assigned RAM, the Linux kernel needs to find 16GB of contiguous physical pages immediately. 

While ZFS is *supposed* to shrink the ARC when memory pressure increases, it doesn't always happen fast enough. The result is a violent tug-of-war between the ARC and your VMs. If the ARC doesn't prune its cache fast enough, the Linux OOM (Out Of Memory) killer wakes up. Since it can't "kill" the kernel-level ZFS cache, it looks for the biggest user-space process it can find—which is usually your most important database VM—and murders it to save the host.

### The Mechanics of the Fix: Hard-Limiting the Beast

The Senior Proxmox Admin approach is to never let the host guess how much RAM it can use. You must treat the ZFS ARC like a service that needs a strict budget. 

On a hypervisor, your RAM priority should look like this:
1. **The OS/Hypervisor:** 2GB to 4GB.
2. **The Virtual Machines:** Whatever you've assigned (the bulk of your RAM).
3. **The ZFS ARC:** Whatever is left over, but rarely more than 8GB to 16GB.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/Proxmox-ZFS-ARC-Why-Your-Hypervisor-is-Starving-VMs-of-Memory/](https://www.valtersit.com/guides/databases/Proxmox-ZFS-ARC-Why-Your-Hypervisor-is-Starving-VMs-of-Memory/)**
