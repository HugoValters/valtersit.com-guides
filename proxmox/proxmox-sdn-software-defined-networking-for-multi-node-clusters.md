> 📖 **Original article:** [Proxmox SDN: Software-Defined Networking for Multi-Node Clusters](https://www.valtersit.com/guides/proxmox/proxmox-sdn-software-defined-networking-for-multi-node-clusters/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I got the call at 2:47 AM. A junior admin had deleted a VNet while five production VMs were running. All of them dropped off the network simultaneously. The `/etc/pve/sdn/` directory had no backups. We spent the next four hours reconstructing VNI-to-VLAN mappings from memory and stale monitoring logs. That was the night I stopped treating Proxmox SDN as optional infrastructure and started treating it like a core system that demands the same rigor as your storage backend.

This guide is for senior DevOps and sysadmins running Proxmox clusters with more than three nodes. If you're still editing `/etc/network/interfaces` by hand and managing VLANs through a spreadsheet, you're one misconfig away from a production outage. After reading this, you'll be able to deploy VXLAN-based SDN with head-end replication, configure VLAN-backed zones for legacy environments, implement firewall security groups, and avoid the mistakes that cost me that 2 AM call.

:::note[TL;DR]
- Proxmox SDN abstracts network configuration from hypervisor bridges into zones, VNets, and subnets
- Use VXLAN with head-end replication for clusters over 5 nodes — it scales to ~100 nodes without multicast headaches
- VLAN-backed SDN is a stepping stone: you hit the 4094 VLAN limit faster than you think
- SDN firewalls and security groups provide tenant isolation that VM-level firewalls can't match
- Backup `/etc/pve/sdn/` before every change — there's no dry-run mode
:::

## Prerequisites

- Proxmox VE 7.x or later (SDN is stable since 7.2)
- L3 routing between all cluster nodes (no STP loops)
- At least one dedicated physical NIC or VLAN trunk per node for SDN traffic
- `pvesh` CLI access on any cluster node
- For VXLAN: UDP port 4789 open between all nodes

## Core Concepts – Zones, VNets, and Subnets

### Zones = Network Domains

Zones define the encapsulation method and underlay requirements. Proxmox supports four types: VLAN, VXLAN, EVPN, and Simple. Your choice determines how far your network can scale and how much your network team will hate you.

I once audited a cluster where a team used a Simple zone for ten nodes. Simple zones are flat Layer 2 domains with no isolation. They learned about broadcast storms when a misconfigured VM generated 200 Mbps of ARP traffic that saturated every host. Use Simple zones only for single-node testing.

Here's the decision matrix:

| Zone Type | Underlay Requirement | Max Networks | Use Case |
|-----------|----------------------|--------------|----------|
| VLAN | L2 adjacency | 4094 | Small clusters, existing switch infrastructure |
| VXLAN | L3 routing + UDP 4789 | 16 million | Multi-node, multi-site, cloud/colo |
| EVPN | BGP + VTEP | Unlimited | Carrier-grade, dynamic routing |
| Simple | None | 1 | Single-node, testing only |

### VNets = Virtual Networks

VNets map to VLAN IDs or VXLAN VNIs. They're your tenant networks. Naming conventions matter: `prod-db-vlan100` tells you what it is; `vlan100` tells you nothing after you've created fifty of them.

Here's a VLAN zone configuration:

```bash
# /etc/pve/sdn/zones.cfg
vlan: prod-zone
    type vlan
    bridge vmbr1
    vlan-id 100-500
```

This creates a zone named `prod-zone` that uses bridge `vmbr1` as the trunk interface and allows VLANs 100 through 500. The bridge must exist on every node before Proxmox will activate the zone.

### Subnets = IPAM and DHCP Integration

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/proxmox/proxmox-sdn-software-defined-networking-for-multi-node-clusters/](https://www.valtersit.com/guides/proxmox/proxmox-sdn-software-defined-networking-for-multi-node-clusters/)**
