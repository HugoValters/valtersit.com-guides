> 📖 **Original article:** [Network Segmentation: VLANs and Firewall Rules for Service Isolation](https://www.valtersit.com/guides/security/network-segmentation-vlans-and-firewall-rules-for-service-isolation/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I've spent 15 years cleaning up the aftermath of flat networks. The worst was a 2020 incident where a Redis instance, left exposed on the default VLAN, became an attacker's highway. Within four hours, they'd pivoted through 200+ VMs, exfiltrated customer data, and planted cryptominers across three availability zones. The CISO's words still echo: "We thought the firewall at the edge was enough."

This guide is for platform engineers and DevOps teams who've outgrown the "just add a security group" mindset. You'll learn how VLANs and firewall rules actually work together—not just theory, but production-hardened configs that prevent lateral movement. By the end, you'll be able to design a segmented network that contains breaches instead of amplifying them.

:::note[TL;DR]
- Flat networks are indefensible; one compromised service becomes your entire blast radius.
- VLANs provide Layer 2 isolation, but firewall rules enforce actual access control between segments.
- Group services by trust level, not by count—"one VLAN per service" creates management hell.
- Egress filtering catches 90% of breaches; allow-all-outbound is a sieve.
- Automate everything with IaC; hand-jammed rules are technical debt with interest.
- Test segmentation monthly with port scans—trusting your config is how breaches happen.
:::

## Prerequisites

- Access to a network device (Cisco IOS, Linux bridge, or cloud VPC) for VLAN configuration
- Basic understanding of OSI model layers 2-4
- A firewall platform (iptables/nftables on Linux, or cloud-native like AWS Security Groups)
- `nmap` and `tcpdump` installed on test machines for verification
- Terraform or similar IaC tool if automating (recommended)

## Why Segmentation Matters — The "Flat Network is a Crime" Argument

A flat network is the architectural equivalent of putting all your servers in one room with the door unlocked. It's convenient for the first deployment, but the moment an attacker finds one vulnerability—an unpatched web app, a Redis without auth, a Jenkins with default creds—they own everything. Lateral movement becomes trivial because there are no boundaries to cross.

The principle is simple: least privilege applied to the network layer. Every service should only talk to exactly what it needs, on exactly the ports required, and nothing else. This isn't paranoia—it's physics. You cannot defend what you cannot isolate.

**Q: "But my startup has 5 services—do I really need VLANs?"**
Yes. You won't stay a startup forever, and retrofitting segmentation into a production network is a nightmare that involves downtime, broken dependencies, and late-night rollbacks. Start segmented from day one, even if it's just three VLANs. Your future self will thank you.

## VLAN Fundamentals — Not Just for Network Engineers Anymore

### What VLANs Actually Do (and Don't Do)

VLANs isolate broadcast domains at Layer 2. That's it. They prevent ARP storms from crossing segments and force traffic through a router for inter-VLAN communication. But they are not encryption, not authentication, and not a security boundary on their own. If you trunk VLANs together without firewall rules, you've built a flat network with extra steps.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/network-segmentation-vlans-and-firewall-rules-for-service-isolation/](https://www.valtersit.com/guides/security/network-segmentation-vlans-and-firewall-rules-for-service-isolation/)**
