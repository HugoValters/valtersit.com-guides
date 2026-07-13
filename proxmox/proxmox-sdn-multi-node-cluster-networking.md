> 📖 **Original article:** [Proxmox SDN: Multi-Node Cluster Networking](https://www.valtersit.com/guides/proxmox/proxmox-sdn-multi-node-cluster-networking/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

## Introduction – Why SDN in Proxmox Isn’t Just a Buzzword

I watched a junior admin bring down a 12-node production cluster for four hours during a maintenance window because he copy-pasted bridge configs from Node A to Node B without realizing Node B had a different physical NIC naming scheme. The VLAN tags were off by one on three bridges. Live migrations failed silently for two weeks before we noticed.

That's the problem with traditional Linux bridges in multi-node Proxmox clusters: every node's `/etc/network/interfaces` is a snowflake. VLAN tagging drifts across hosts. Manual edits accumulate like technical debt. And when a VM live migration fails because the target node's bridge doesn't have the right VLAN trunked, you're left explaining to management why "just move the VM" isn't that simple.

Proxmox SDN solves this by separating network policy from physical topology. Instead of editing config files on each node, you define zones, VNets, and subnets once—cluster-wide. The SDN controller pushes that config to every node. Your bridges become consistent, migrations work, and you stop waking up at 3 AM for VLAN tag mismatches.

This guide covers building a resilient, multi-tenant SDN with VXLAN and VLAN zones. No hand-holding. No "just works" marketing. Just what I've learned from breaking and fixing Proxmox clusters in production since VE 7.4.

:::note[TL;DR]
- Traditional bridge configs per node cause drift and migration failures in multi-node clusters
- Proxmox SDN centralizes network config across all nodes via `/etc/pve/sdn/`
- VXLAN zones support 16 million segments vs VLAN's 4096—use VXLAN for clusters >3 nodes
- Live migrations work reliably with VXLAN but require identical peer configs on all nodes
- Missing `ifupdown2` package causes SDN changes to be silently ignored
:::

## Prerequisites

- Proxmox VE 7.4+ or 8.x (tested on 8.2)
- Cluster with 3+ nodes (quorum required for SDN changes)
- L3 connectivity between all cluster nodes
- `ifupdown2` package installed on every node (`apt install ifupdown2`)
- Physical switch MTU >= 1550 if using VXLAN (1500 + 50 byte overhead)
- IGMP snooping enabled on switches if using VXLAN multicast

## Proxmox SDN Architecture – The 30,000-Foot View (Without the BS)

### Core Components – Zones, VNets, Subnets, and IPAM

Proxmox SDN has four building blocks. Understanding them before you touch the GUI saves hours of debugging.

**Zone:** Defines the transport mechanism. Options are `vlan`, `vxlan`, `qinq`, or `simple`. Pick one transport per zone—mixing them in the same zone causes pain I won't wish on anyone.

**VNet:** A logical network segment—think of it as a virtual switch. Create one VNet per tenant or workload type. I use `prod-web`, `prod-db`, `dev-app` naming.

**Subnet:** An IP range within a VNet. Must use CIDR notation and must be unique across the entire cluster. Overlap subnets between VNets? You'll get routing chaos.

**IPAM:** Proxmox's built-in IP address management. Optional, but use it if you hate maintaining IP spreadsheets. It tracks allocations per VNet and prevents duplicate IPs.

```bash
# Check current SDN status across the cluster
pvesh get /cluster/sdn/status --output-format json
```

This returns JSON showing each zone's state, VNet membership, and any errors. If you see `"state": "error"` on any zone, fix that before adding VMs.

### Why VXLAN > VLAN for Multi-Node (and When VLAN Still Wins)

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/proxmox/proxmox-sdn-multi-node-cluster-networking/](https://www.valtersit.com/guides/proxmox/proxmox-sdn-multi-node-cluster-networking/)**
