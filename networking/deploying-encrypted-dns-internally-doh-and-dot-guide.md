> 📖 **Original article:** [Deploying Encrypted DNS Internally: DoH and DoT Guide](https://www.valtersit.com/guides/networking/deploying-encrypted-dns-internally-doh-and-dot-guide/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I was on-call when it happened. 3 AM on a Tuesday, and every single production server in our us-east-1 region started resolving `api.internal.example.com` to a server in Belarus. Our recursive DNS resolver had been MITM'd by an attacker who'd gained access to a network tap in the colo facility. Four hours of chaos, a panicked VP, and a post-mortem that boiled down to: "Our DNS was plaintext over UDP, and we trusted the network."

That was 2022. If you're still running plaintext DNS internally in 2026, you're not just behind — you're negligent. This guide is for senior sysadmins and security engineers who need to deploy encrypted DNS (DoH and DoT) in their infrastructure without breaking everything. By the end, you'll have a working deployment, a migration plan, and the scars to know what goes wrong.

:::note[TL;DR]
- Plaintext DNS is a massive security liability: on-path tampering, passive surveillance, and cache poisoning at the network edge
- Use DoT for internal infrastructure (predictable, low overhead), DoH for external-facing resolvers (evasion, compliance)
- Start with Unbound or CoreDNS — skip the overengineered solutions
- Certificate management is the #1 cause of failure: automate rotation or suffer at 3 AM
- Migrate gradually with opportunistic mode first, then enforce with firewall rules
:::

## Prerequisites

- A Linux server (Ubuntu 22.04+ or RHEL 9+) for your DNS resolver — don't use your laptop
- Root access and basic familiarity with `systemd`, `openssl`, and `tcpdump`
- A DNS zone you control for testing (e.g., `dns.internal.example.com`)
- `kdig` installed (`apt install knot-dnsutils` or `yum install knot-dnsutils`)
- At least 30 minutes of uninterrupted time — this isn't a 5-minute config

## Why You Should Care About Encrypted DNS (And Why Plaintext DNS Is a Security Liability)

Plaintext DNS over UDP is the cockroach of network protocols — it's everywhere, it's ugly, and it survives anything. Here's what it exposes you to:

**On-path tampering:** Any device between your client and the resolver can modify DNS responses. This isn't theoretical — I've seen attackers redirect traffic to phishing pages by poisoning ARP tables and injecting fake DNS responses.

**Passive surveillance:** Your DNS queries reveal every service, every internal hostname, and every external dependency you have. An attacker who sniffs your DNS traffic knows your infrastructure better than your CMDB does.

**Cache poisoning at the network edge:** If an attacker can inject a single fake DNS response before the legitimate one arrives, your resolver caches it for the TTL. This is how the Kaminsky attack worked, and it's still effective against poorly configured resolvers.

The false comfort of "internal network" security needs to die. In 2026, your internal network is not a trusted zone — it's a blast radius. If you're still running `dnsmasq` with no encryption, you're basically leaving the keys in the ignition with the engine running.

Here's what plaintext DNS looks like on the wire — run this on any server that's doing DNS queries:

```bash
# Capture a single DNS query to show how much leaks
tcpdump -i any -X -c 1 'udp port 53 and dst port 53'
```

This will show your query in clear text: hostnames, timestamps, and source IPs. Every network hop between you and the resolver sees this. If you're thinking "but we use DNSSEC," you're missing the point — DNSSEC validates response integrity but does nothing for confidentiality or transport security.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/networking/deploying-encrypted-dns-internally-doh-and-dot-guide/](https://www.valtersit.com/guides/networking/deploying-encrypted-dns-internally-doh-and-dot-guide/)**
