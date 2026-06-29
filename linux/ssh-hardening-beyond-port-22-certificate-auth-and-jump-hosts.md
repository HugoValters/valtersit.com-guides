> 📖 **Original article:** [SSH Hardening Beyond Port 22: Certificate Auth and Jump Hosts](https://www.valtersit.com/guides/linux/ssh-hardening-beyond-port-22-certificate-auth-and-jump-hosts/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# SSH Hardening Beyond Port 22: Certificate Authentication and Jump Hosts

I took over a security engagement for a mid-stage startup that had "hardened" their SSH infrastructure. They'd moved SSH to port 2222, installed fail2ban, and felt smug about it. Their CTO told me, "We're secure — we even changed the default port."

Four hours after I deployed a simple port scan and credential stuffing attack against their new port, I was root on their CI/CD runner. They had 47 servers with password authentication still enabled, root login permitted on 12 of them, and a single jump host that ran a full desktop environment. The "hardened" SSH setup lasted less than an afternoon against someone who actually tried.

This guide is for DevOps engineers, security engineers, and platform teams who manage more than 10 servers and want to stop playing whack-a-mole with SSH security. By the end, you'll understand why port changes are cosmetic theater, how to implement SSH certificate authentication that scales, and how to design jump hosts that don't become your network's front door with a broken lock.

:::note[TL;DR]
- Changing SSH port is security theater if you still use password auth or weak keys
- SSH certificate authentication eliminates standing keys, provides built-in expiry, and scales to thousands of servers
- Jump hosts must be disposable, minimal, and authenticated via certs — not password vaults
- Short-lived certificates (1-8 hours) from Vault or a custom CA are the gold standard
- Session recording and audit logging turn SSH from a blind spot into an auditable control
:::

## Prerequisites

- OpenSSH 8.0+ (most modern distros)
- Root or sudo access on at least one server to modify `sshd_config`
- A Linux or macOS workstation for CA key generation
- Basic familiarity with SSH key pairs and `sshd_config` directives

## Why Changing Port 22 Is the Least Interesting Thing You Can Do

### The Port-Knocking Fallacy

Moving SSH from port 22 to 2222 or 8022 is the security equivalent of locking your front door but leaving the window open. Attackers scan all 65535 ports in under 30 seconds with tools like masscan or zmap. A non-default port only filters out script kiddies running default scans — and even that's debatable.

Here's the `sshd_config` that the startup used:

```bash
# /etc/ssh/sshd_config (the "hardened" version)
Port 2222
PermitRootLogin yes
PasswordAuthentication yes
```

And here's what `journalctl` showed after I scanned their new port:

```
Jun 15 14:23:01 jumpbox sshd[1234]: Failed password for root from 203.0.113.42 port 54321 ssh2
Jun 15 14:23:02 sshd[1235]: Failed password for root from 203.0.113.42 port 54322 ssh2
Jun 15 14:23:03 sshd[1236]: Accepted password for root from 203.0.113.42 port 54323 ssh2
```

Three seconds between finding the port and getting root. The port change added zero security.

### Attack Surface Reduction Starts at Authentication, Not Port

| Control | Security Level | Operational Overhead | Scalability | Auditability |
|---------|---------------|---------------------|-------------|--------------|
| Port obfuscation | Low (security theater) | Low | N/A | None |
| SSH key-pair auth | Medium | Medium (key management) | Poor (no expiry/revocation) | Minimal |
| SSH certificate auth | High | Medium (CA setup) | Excellent (built-in expiry, revocation) | Full |

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/linux/ssh-hardening-beyond-port-22-certificate-auth-and-jump-hosts/](https://www.valtersit.com/guides/linux/ssh-hardening-beyond-port-22-certificate-auth-and-jump-hosts/)**
