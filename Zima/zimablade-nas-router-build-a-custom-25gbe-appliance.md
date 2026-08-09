> 📖 **Original article:** [ZimaBlade NAS Router: Build a Custom 2.5GbE Appliance](https://www.valtersit.com/guides/Zima/zimablade-nas-router-build-a-custom-25gbe-appliance/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

You know what's worse than paying $800 for a "gaming router" with a dual-core ARM chip and 512MB of RAM? Finding out that same router has a telnet daemon listening on the WAN interface three months after you bought it. That's not hyperbole — that's the QNAP QTS exploit from 2019 that hit unpatched NAS boxes like a sledgehammer. Commercial "appliances" aren't security products. They're Linux boxes with a pretty UI, worse update discipline, and a 3x markup.

This guide is for people who understand that a router is just a computer with multiple network interfaces, and a NAS is just a computer with disks. The ZimaBlade — an x86_64 single-board computer with a PCIe 3.0 x4 slot, 16GB RAM ceiling, and a $150 entry point — can replace $1,200 of commercial hardware with better performance, full control, and no vendor lock-in.

By the end of this guide, you'll have a single ZimaBlade running Proxmox VE as the hypervisor, OPNsense as a hardened router with IDS/IPS, and TrueNAS Scale as a ZFS-backed NAS. You'll have VLANs from day one, PCIe passthrough done right, and a monitoring stack that tells you when things are dying — before they die. This is not a beginner project. If you don't know what a VLAN is, go read something else first.

:::note[TL;DR]
- A ZimaBlade 7700K with 16GB RAM, an Intel I226-V NIC, and an LSI HBA costs ~$400 and replaces $1,200 of commercial hardware
- Run Proxmox VE as the hypervisor — it's free, Debian-based, and has the best PCIe passthrough support
- Pass the PCIe NIC directly to OPNsense and the SATA HBA directly to TrueNAS — no virtualization overhead
- VLANs from day one: Management, Trusted, IoT, Guest, and WAN. Segregate or get pwned by your smart bulb
- RAID is not backup. Set up offsite replication before you need it, not after you lose 12TB of family photos
:::

## Prerequisites

Before you start, you need:

- A ZimaBlade (7700K or 7900 recommended — more on that below)
- 16GB DDR4 SODIMM (the maximum supported)
- An Intel-based PCIe NIC (I225-V or I226-V — do NOT buy Realtek)
- An LSI/Broadcom HBA in IT mode for SATA drives (if you're using more than one HDD)
- A managed switch that supports VLANs (used for trunking)
- A USB drive (minimum 8GB) for the Proxmox installer
- At least 2 SATA HDDs and/or 2 NVMe drives for ZFS mirrors

## Hardware Selection — Don't Skimp on the Wrong Components

### ZimaBlade Model Comparison

The ZimaBlade comes in three flavors. The differences matter more than you think.

| Model | CPU Cores/Threads | Max RAM | PCIe Lanes | TDP | Idle Power Draw | Price | What You Actually Need |
|-------|-------------------|---------|------------|-----|-----------------|-------|----------------------|
| 7700 | 4C/8T (Intel N100) | 16GB | 4 (3.0) | 6W | ~5W | $150 | 2.5GbE routing, light NAS |
| 7700K | 4C/8T (Intel N305) | 16GB | 4 (3.0) | 15W | ~8W | $220 | 2.5GbE + Suricata IDS, multiple VMs |
| 7900 | 8C/8T (Intel Core i3-N305) | 16GB | 4 (3.0) | 15W | ~9W | $280 | 10GbE routing, multiple VMs, heavy ZFS |

**Buy the 7900 if you can afford it.** The extra cores matter when you're running two VMs, Suricata IDS, and ZFS compression simultaneously. The 7700 base model will work, but you'll be CPU-bound the moment you enable IDS and start a ZFS scrub at the same time.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/Zima/zimablade-nas-router-build-a-custom-25gbe-appliance/](https://www.valtersit.com/guides/Zima/zimablade-nas-router-build-a-custom-25gbe-appliance/)**
