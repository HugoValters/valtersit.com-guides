> 📖 **Original article:** [Proxmox Network Configuration: Bridges, Bonds, and VLANs](https://www.valtersit.com/guides/proxmox/proxmox-network-configuration-bridges-bonds-and-vlans/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I once spent a Sunday afternoon restoring a client's 12-node Proxmox cluster because someone thought `balance-rr` on a single switch was a good idea. The switch's MAC table filled up in 47 seconds, the STP reconvergence took down their Ceph storage network, and three production databases went read-only. The root cause wasn't a hardware failure — it was a network config that looked fine in a 3-node lab.

This guide is for senior sysadmins who manage Proxmox in production. You already know how to install Proxmox and create a VM. What you need is the battle-tested knowledge of how to design and debug network configurations that survive kernel updates, switch failures, and junior admin mistakes. By the end, you'll be able to architect multi-VLAN, multi-bond setups with confidence and debug the inevitable failures without panicking.

:::note[TL;DR]
- Default `vmbr0` on a single NIC is for testing, not production
- Use LACP bonding (`mode=4`) for any host with redundant paths
- Separate storage traffic (Ceph) from VM traffic with dedicated bridges and VLANs
- Always test network changes with `ifreload -a` before rebooting
- Version control your `/etc/network/interfaces` files
:::

## Prerequisites

- Proxmox VE 7.x or 8.x installed (physical or virtual)
- Root shell access to the Proxmox host
- At least two physical NICs if you're following the bonding examples
- A managed switch that supports LACP (802.3ad) and VLAN tagging
- IPMI/iDRAC/iLO access for out-of-band recovery

## Why Default Proxmox Networking Will Bite You in Production

The default Proxmox installer creates a single bridge (`vmbr0`) attached to one NIC. This works beautifully for a home lab with three VMs. In production, it's a single point of failure that will fail at the worst possible moment.

### The "It Worked in My Lab" Trap

Here's the default config that Proxmox generates:

```bash
# /etc/network/interfaces — default, dangerous for production
auto lo
iface lo inet loopback

auto eno1
iface eno1 inet manual

auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100/24
    gateway 192.168.1.1
    bridge-ports eno1
    bridge-stp off
    bridge-fd 0
```

This config has exactly one failure domain: `eno1`. If that cable gets unplugged, the NIC fails, or the switch port goes down, your entire Proxmox host goes dark. No SSH, no web interface, no VMs.

I had a client who lost all connectivity during a routine kernel update. The update triggered a driver reload on `eno1`, the NIC came back with a different MAC address (thanks to a firmware bug), and the bridge refused to forward traffic. Without a bond for failover, they were down for two hours while someone drove to the datacenter.

### The Hidden Cost of Complexity

I'm not saying every Proxmox host needs four bonds and eight VLANs. If you're running a single host with three VMs for your home media server, a single bridge is fine. The complexity tax is real — every bond, VLAN, and bridge you add is another thing that can break.

But if you're running:
- A multi-node Proxmox cluster with HA
- Ceph or any other distributed storage
- Multiple tenants with separate network requirements

Then you need proper network segmentation. If you're running Ceph on a single bridge shared with VM traffic, you're already fired. Ceph's OSD heartbeat traffic and replication will fight with your production VMs for bandwidth, and when the cluster gets busy, you'll see OSD flapping and slow requests.

### The One File to Rule Them All

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/proxmox/proxmox-network-configuration-bridges-bonds-and-vlans/](https://www.valtersit.com/guides/proxmox/proxmox-network-configuration-bridges-bonds-and-vlans/)**
