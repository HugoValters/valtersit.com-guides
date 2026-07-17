> 📖 **Original article:** [Redis Cluster vs Sentinel: Choosing the Right HA Setup](https://www.valtersit.com/guides/databases/redis-cluster-vs-sentinel-choosing-the-right-ha-setup/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# Redis Cluster vs Sentinel: Choosing the Right High-Availability Setup

## Introduction

I've spent the last decade untangling Redis HA setups that someone "just threw together" in production. The worst one? A fintech company running Sentinel with two instances and `quorum=1` because "it's just a cache." Their payment processing pipeline served stale session data for 47 minutes during a network blip. The CTO's words: "We thought it was working." That's the problem — Redis high availability looks easy until it isn't.

Redis Sentinel was born in 2012 as a separate process to monitor and automatically failover master-replica setups. Redis Cluster followed in 2015, baked into the server itself, adding automatic sharding and distributed failover. Both solve availability, but they solve *different* problems. Picking the wrong one costs you weekends — and sometimes data.

This guide is a decision framework, not marketing fluff. By the end, you'll know exactly which HA model fits your workload, how to configure it without shooting yourself in the foot, and what failure modes to expect at 3 AM.

**What you'll need**: Redis 6.x or later, a test environment with at least 3 nodes, and a healthy distrust of default configurations. No code examples here — this is scene-setting.

## Understanding the Problem — What "High Availability" Actually Means for Redis

### The Three Dimensions of Availability

High availability isn't a binary state. It's a tradeoff across three dimensions:

- **Recovery Point Objective (RPO)**: How much data loss can you tolerate? Seconds? Minutes? None?
- **Recovery Time Objective (RTO)**: How fast must the system recover? Sub-second? 10 seconds? A minute?
- **Consistency During Partition**: What happens when the network splits? Do you serve stale data, reject writes, or risk data loss?

Here's a war story that illustrates the cost of ignoring this: A client thought Sentinel gave them sharding. It didn't. Their "cache" — actually a critical session store — served stale data for 47 minutes during a partition because Sentinel promoted a replica that was behind the master. They had no idea because their application never checked replication lag.

The fundamental difference between Sentinel and Cluster shows up in their core commands:

```bash
# Sentinel: asks "who's the master?" — returns one IP:port
redis-sentinel get-master-addr-by-name mymaster

# Cluster: asks "where's this key?" — returns slot and node mapping
redis-cli --cluster check 10.0.0.1:6379
```

Sentinel gives you a single endpoint. Cluster gives you a topology. That's the difference between "which node is in charge" and "where does my data live."

### Common Misconceptions (and Who's Selling Them)

**"Sentinel is simpler"** — until you have 5+ instances and need to update configs on each one independently. Sentinel has no central configuration management. Every config change means touching every Sentinel node.

**"Cluster is automatic"** — until you hit resharding bugs. I've seen `redis-cli --cluster rebalance` take down a production cluster because it moved slots without checking hash tag boundaries. The "automatic" part is a lie; it's *automated*, not automatic.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/redis-cluster-vs-sentinel-choosing-the-right-ha-setup/](https://www.valtersit.com/guides/databases/redis-cluster-vs-sentinel-choosing-the-right-ha-setup/)**
