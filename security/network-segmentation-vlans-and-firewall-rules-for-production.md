> 📖 **Original article:** [Network Segmentation: VLANs and Firewall Rules for Production](https://www.valtersit.com/guides/security/network-segmentation-vlans-and-firewall-rules-for-production/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# Network Segmentation: Isolating Services with VLANs and Firewall Rules

## Introduction: Why Your Flat Network is a Liability

I once walked into a post-mortem where a single compromised WordPress plugin led to an attacker pivoting from a public web server to a PCI-compliant payment database in under 90 seconds. How? The network was flat — one VLAN, one broadcast domain, no firewall rules between services. The attacker ran `arp-scan`, found the DB server, and started exfiltrating credit card numbers over port 3306. The CISO asked me, "How do we stop this?" My answer: VLANs and firewall rules, applied correctly.

Flat networks are fine for your home lab. In production, they're a breach waiting to happen. The problem is blast radius — if an attacker compromises one service, they can pivot laterally to everything. VLANs create logical barriers at Layer 2, and firewall rules enforce who talks to whom at Layers 3-4. Together, they're the minimum viable defense for any multi-service environment.

This guide is for senior engineers who already know what a VLAN is but need to design, implement, and maintain segmentation that actually works. After reading, you'll be able to kill lateral movement in your network, avoid the common misconfigurations that make segmentation useless, and sleep better at night.

:::note[TL;DR]
- Flat networks let attackers pivot laterally — segment everything with VLANs and firewalls
- Use 5-10 VLANs for "trust zones" (web, app, db, management, DMZ), not one per microservice
- Default-deny firewall rules between VLANs; stateful inspection is mandatory
- Version-control your firewall rules and monitor dropped packets
- Test segmentation with `nmap` and `tcpdump`, not just `ping`
:::

## Prerequisites

- Access to a managed switch (Cisco IOS, Aruba, or similar) with VLAN support
- A Linux host (Ubuntu 22.04+ or Debian 12) with `iptables` or `nftables` installed
- Basic understanding of IP addressing and subnetting
- Root or sudo access on the Linux router/firewall

## VLAN Fundamentals: The Logical Barrier You're Probably Misconfiguring

### VLAN Tagging 101 (802.1Q) — What Most Tutorials Get Wrong

VLAN tagging using 802.1Q inserts a 4-byte tag into Ethernet frames to identify which VLAN a packet belongs to. Most tutorials skip the critical part: **disable DTP (Dynamic Trunking Protocol) on every switch port that shouldn't be a trunk**. DTP negotiates trunking automatically, and if an attacker plugs into an access port that has DTP enabled, they can negotiate a trunk and perform VLAN hopping — accessing traffic from other VLANs.

Here's the Cisco IOS config to kill DTP on access ports:

```cisco
interface GigabitEthernet0/1
 description Web Server Access Port
 switchport mode access
 switchport access vlan 10
 no switchport trunk dynamic
 spanning-tree portfast
```

The `switchport mode access` forces the port to access mode and disables DTP negotiation. The `spanning-tree portfast` skips STP listening/learning for faster convergence on end-host ports. On Aruba/ProVision switches, the equivalent is:

```bash
interface 1/1
   vlan access 10
   no vlan trunk dynamic
```

In Linux bridging for virtualized environments (e.g., KVM with Linux bridge), you configure VLAN tagging at the interface level:

```bash
ip link add link eth0 name eth0.10 type vlan id 10
ip addr add 10.0.10.1/24 dev eth0.10
ip link set eth0.10 up
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/network-segmentation-vlans-and-firewall-rules-for-production/](https://www.valtersit.com/guides/security/network-segmentation-vlans-and-firewall-rules-for-production/)**
