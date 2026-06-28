> 📖 **Original article:** [Your Containers Are Bloated, Insecure Garbage. Let's Fix That with Distroless.](https://www.valtersit.com/guides/docker/your-containers-are-bloated-insecure-garbage-lets-fix-that-with-distroless/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

## You're Shipping an Entire OS for a Single Binary. Stop It.

Let's get one thing straight. If your `Dockerfile` starts with `FROM ubuntu:latest`, you are committing professional malpractice. I don't care if you followed some five-minute tutorial on Medium. You've taken a single, compiled application—the only thing you actually care about—and wrapped it in 300MB of useless, vulnerable Linux distribution cruft.

You have `bash`, `coreutils`, `find`, `grep`, `curl`, `wget`, `apt`, `sed`, `awk`, and hundreds of other binaries that have absolutely nothing to do with your Python API or Go microservice. Every single one of those tools is a potential attack vector. Every shared library is a CVE waiting to happen.

Remember Log4Shell? The exploit wasn't just that you could run code; it was what you could *do* with that code. Attackers weren't dropping their custom malware; they were using the `wget` and `curl` binaries you so thoughtfully included in your bloated image to exfiltrate data and download their second-stage payloads. You handed them a fully-loaded toolkit on a silver platter.

And don't get me started on the "slim" tags. `python:3.11-slim-bullseye` is a feel-good measure, a pat on the back for doing the bare minimum. It's like switching from whole milk to 2% and calling it a diet. It still has a shell. It still has a package manager (`apt`). It still has dozens of utilities. You've reduced the size, but you haven't fundamentally reduced the attack surface. Your job is to ship an *application*, not a poorly-maintained, miniature Linux distribution with your code sprinkled on top. It's time to do the job properly.

---

## What 'Distroless' Actually Means

The term "distroless" gets thrown around like it's some kind of magic. It's not. It's the logical conclusion of a simple idea: your container should contain your application and *only* what is strictly necessary to run it.

Google's Distroless images are a set of minimal base images designed to be the final stage in a multi-stage build. They contain:

1.  The language runtime (e.g., glibc, OpenJDK, the Python interpreter).
2.  Essential shared libraries required by the runtime (e.g., OpenSSL for crypto, `ca-certificates` for TLS).
3.  ...and that's it.

There is no shell. No `ls`, `cat`, or `echo`. No package manager. If an attacker gains execution inside your container, their environment is a barren wasteland. There are no tools to perform reconnaissance, download payloads, or manipulate the filesystem. They have your running process and that's it. It turns a potential RCE (Remote Code Execution) into a significantly less useful "run my application's code in weird ways" problem.

### Distroless vs. Alpine: A Quick and Dirty Takedown

"But Alpine is only 5MB!" Yes, and it's small for a reason that will eventually ruin your weekend. Alpine Linux is built on `musl libc`, a lightweight alternative to the `glibc` (GNU C Library) that underpins nearly every other major Linux distribution (Debian, Ubuntu, CentOS, etc.).

This is not a trivial difference. If your application or any of its dependencies—especially in languages like Python, Node.js, or Ruby that rely heavily on native C extensions—were compiled against `glibc`, running them on `musl` is a gamble. You're praying for perfect POSIX compliance. You'll encounter subtle, nightmarish bugs related to DNS resolution, memory allocation, or threading that only manifest under production load. It's a ticking time bomb.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/docker/your-containers-are-bloated-insecure-garbage-lets-fix-that-with-distroless/](https://www.valtersit.com/guides/docker/your-containers-are-bloated-insecure-garbage-lets-fix-that-with-distroless/)**
