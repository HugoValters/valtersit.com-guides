> 📖 **Original article:** [Proxmox Resource Pools: Multi-Tenant Isolation Guide](https://www.valtersit.com/guides/proxmox/proxmox-resource-pools-multi-tenant-isolation-guide/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

A managed service provider migrated 40 customers onto a single six-node Proxmox VE cluster. Every VM lived in the default "no pool" state. Every tenant admin had been handed `PVEVMAdmin` on `/vms` during onboarding because "it just made things easier." One Tuesday afternoon, a new hire at tenant #12 ran `qm list` from a jump box and printed the entire customer base — VMIDs, hostnames, resource usage, node placement — to a shared Slack channel. Nothing was breached. Nothing was exploited. The permission model did exactly what it was told to do.

This guide is for sysadmins, MSPs, homelab-to-production operators, and platform engineers who need to split one PVE cluster across teams, departments, or paying customers. After reading it you'll be able to design pools that enforce visibility, roles that actually constrain what a tenant can touch, storage and network boundaries that survive a hostile tenant, and an audit trail that lets you prove all of it after the fact.

## TL;DR

:::note[TL;DR]
- Resource pools are an organizational construct. ACLs are the security boundary. Confuse the two and you get a nice UI folder with zero enforcement.
- PVE ACLs are **purely additive** — there is no deny rule anywhere in the model. Isolation comes from *not granting*, not from revoking.
- Tenant isolation has three tiers. Know which one you're actually buying or you'll oversell Tier 1 as Tier 3 and get burned.
- API tokens with `--privsep 1` and pool-scoped ACLs, plus SDN VNets and dedicated storage, are the minimum viable Tier 3 setup.
:::

## Prerequisites

- Proxmox VE 8.x on at least one node, cluster recommended
- `root@pam` access (or an account with `Permissions.Modify` at `/`)
- Working knowledge of `pveum` and `pvesh`
- A ZFS, LVM, or Ceph storage backend you can carve per tenant
- A managed switch or router you control for VLAN tagging
- A firewall you own — PVE's host firewall is not a substitute for network segmentation

## Proxmox Is Not a Multi-Tenant Hypervisor, and That's Fine

PVE was designed as a trusted-admin cluster management plane. The permission system exists to keep junior admins out of the storage backend, not to defend against a determined adversary sharing hardware with you. KVM is a mature hypervisor, but PVE's tenant story is a *management* control wrapped around it, and management controls fail in predictable ways when you assume they're security controls.

The threat model that should keep you up at night is mundane. It's the contractor with a `PVEAuditor` token enumerating every VMID in the cluster. It's the CI runner whose API token leaked into a public build log because somebody ran with `--privsep 0` and never rotated. It's the tenant admin who can see your storage inventory and starts asking why your Ceph cluster shows 40TB free when their contract says 500GB.

What PVE gives you out of the box: ACLs, roles, groups, users, API tokens with privilege separation, resource pools, SDN VNets, and per-VM firewalls. What you must build yourself: pool hygiene (things live where they're supposed to), storage separation, network segmentation, token lifecycle management, and audit that actually answers "who did what."

### The Permission Model in 90 Seconds

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/proxmox/proxmox-resource-pools-multi-tenant-isolation-guide/](https://www.valtersit.com/guides/proxmox/proxmox-resource-pools-multi-tenant-isolation-guide/)**
