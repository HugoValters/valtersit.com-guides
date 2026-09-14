> 📖 **Original article:** [GCP vs AWS Networking: VPCs, Peering, Transit Hubs](https://www.valtersit.com/guides/cloud/gcp-vs-aws-networking-vpcs-peering-transit-hubs/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# GCP vs AWS Networking: VPCs, Peering, and Transit Gateway Equivalents

I once watched a competent platform team spend six weeks migrating a hub-and-spoke AWS network into GCP. They built one VPC per region — because that's how AWS trains you to think — spun up a peering mesh between them, and then spent another two weeks debugging why their us-central1 workload couldn't see their us-east1 database even though "the VPCs were clearly connected in the console."

They weren't connected. They were the *same VPC*. GCP VPCs are global. The peering mesh was doing nothing because regional VPCs are an AWS construct, not a GCP one. The console was politely showing them a routing table with every subnet in both regions — which they'd been ignoring because they assumed it was a UI bug.

This is what happens when you port an AWS mental model into GCP without checking the semantics first. The nouns sound the same. The behavior is nothing alike. This post is for engineers who know one cloud and are picking up the other, and for multi-cloud operators tired of translating between them at 2 AM. By the end you'll know where the models diverge, what each cloud's transit hub actually does, and which pitfalls to test for before you cut over.

:::note[TL;DR]
- **AWS VPCs are regional, subnets are zonal. GCP VPCs are global, subnets are regional.** This single difference cascades into peering, firewalls, routing, and IAM.
- **VPC peering is non-transitive on both clouds** — but GCP auto-creates routes while AWS makes you add them on every route table on both sides.
- **AWS Transit Gateway is the transitive hub. GCP's equivalent is Network Connectivity Center (NCC)** — but NCC is not a feature-complete TGW clone. Route table segmentation is the gap.
- **Firewall semantics are the biggest practical difference.** GCP has native deny rules, priorities, hierarchical policies, and service-account targeting. AWS has allow-only Security Groups plus stateless NACLs.
- **Route advertisement defaults will bite you.** GCP Cloud Router only advertises regional subnets by default — not the whole VPC.
:::

## Prerequisites

- `gcloud` CLI authenticated against a GCP project with `compute.networks.*` and `networkconnectivity.*` permissions.
- AWS CLI configured with credentials that allow EC2 networking operations.
- Terraform 1.5+ if you plan to follow the IaC snippets.
- Familiarity with CIDR math. If `10.0.0.0/8` doesn't mean anything to you yet, sort that out first.

## VPC and Subnet Fundamentals: Where the Mental Models Diverge

The reason this matters: every downstream decision — peering topology, firewall scope, route propagation, IAM — inherits from the VPC boundary model. Get the boundary wrong and you rebuild the network later.

### AWS: Regional VPCs, Zonal Subnets

A VPC lives in exactly one region. A subnet lives in exactly one AZ. Route tables attach per-subnet (or as a "main" default you should override). CIDR is carved subnet by subnet and you own the plan.

A minimal three-tier carve-out looks like this:

```bash
# Create the VPC
aws ec2 create-vpc --cidr-block 10.20.0.0/16 --region us-east-1

# Create three subnets across two AZs
aws ec2 create-subnet --vpc-id vpc-0a1b2c3d --cidr-block 10.20.1.0/24 \
  --availability-zone us-east-1a
aws ec2 create-subnet --vpc-id vpc-0a1b2c3d --cidr-block 10.20.2.0/24 \
  --availability-zone us-east-1b
aws ec2 create-subnet --vpc-id vpc-0a1b2c3d --cidr-block 10.20.10.0/24 \
  --availability-zone us-east-1a
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/cloud/gcp-vs-aws-networking-vpcs-peering-transit-hubs/](https://www.valtersit.com/guides/cloud/gcp-vs-aws-networking-vpcs-peering-transit-hubs/)**
