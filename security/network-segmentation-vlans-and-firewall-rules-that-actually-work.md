> 📖 **Original article:** [Network Segmentation: VLANs and Firewall Rules That Actually Work](https://www.valtersit.com/guides/security/network-segmentation-vlans-and-firewall-rules-that-actually-work/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I walked into a client site last year to find a payment processing server sitting on the same /24 as the guest WiFi. The "temporary" flat network had been running for four years. A WordPress plugin vulnerability on a marketing site gave an attacker lateral movement to cardholder data in under 40 minutes. The breach cost them their PCI compliance status, two major contracts, and the CISO's job.

If you can ping your database server from the guest WiFi, you don't have a security architecture — you have a prayer.

This guide is for engineers who know their flat network is a problem but haven't been given the ammunition to fix it. After reading, you'll understand VLANs for Layer 2 isolation, firewall rules for Layer 3/4 enforcement, and the operational discipline to keep both from rotting into chaos.

:::note[TL;DR]
- VLANs contain broadcast domains — they are not security boundaries
- Default deny at zone boundaries is non-negotiable for production
- Your IP scheme should encode the VLAN ID — if it doesn't, fix it now
- Every firewall rule needs an owner, a ticket number, and an expiration date
- Disable DTP on every port today — VLAN hopping is trivial when it's on
:::

## Prerequisites

- Access to your network gear (Cisco IOS, pfSense, or Linux with nftables)
- A maintenance window for config changes
- An accurate inventory of what's actually on your network (you'll need it)

## Introduction — Stop Treating Your Network Like a Flattened Pancake

The "one big /24" mentality is a professional liability. The 2013 Target breach should have ended this debate permanently — attackers pivoted from an HVAC vendor's credentials to the POS network because there was no real segmentation between the corporate and payment environments. PCI DSS 4.0 Requirement 1, NIST 800-125, and CIS Controls v8 Control 12 all mandate segmentation. They exist because people keep running flat networks and getting burned.

Here's what we're covering: VLANs for Layer 2 isolation, firewall rules for Layer 3/4 enforcement, and the operational discipline required to maintain both. This isn't theory — this is the architecture I've deployed in environments processing billions in transactions.

## VLAN Fundamentals — The Layer 2 Illusion of Separation

VLANs segment broadcast domains. They make it look like devices on different VLANs can't talk to each other, but that's a convenience of default configuration, not a security property. Any router or L3 switch with interfaces on both VLANs can route between them — and if you haven't configured firewall rules, that routing is unrestricted.

802.1Q tagging works by inserting a 4-byte tag into the Ethernet frame header. This tag carries a 12-bit VLAN ID, which allows switches to forward frames only to ports in the same VLAN. The critical security point: this tag is not encrypted, not authenticated, and trivially spoofable if an attacker controls a trunk port.

Here's the Cisco config pattern I use for every environment:

```cisco
vlan 10
 name WEB
vlan 20
 name APP
vlan 30
 name DB
!
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
!
interface GigabitEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 999
```

This creates three VLANs, assigns one access port, and configures a trunk with an explicit allowed VLAN list. The native VLAN is changed from the default of 1 to 999 — an unused VLAN that reduces the attack surface for VLAN hopping.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/network-segmentation-vlans-and-firewall-rules-that-actually-work/](https://www.valtersit.com/guides/security/network-segmentation-vlans-and-firewall-rules-that-actually-work/)**
