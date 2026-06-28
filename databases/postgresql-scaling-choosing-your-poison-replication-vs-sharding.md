> 📖 **Original article:** [PostgreSQL Scaling: Choosing Your Poison (Replication vs. Sharding)](https://www.valtersit.com/guides/databases/postgresql-scaling-choosing-your-poison-replication-vs-sharding/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

## Introduction: The "Web Scale" Cargo Cult

Let's get one thing straight. You read a blog post from Uber, Netflix, or some other company with a valuation larger than the GDP of a small country, and now you're convinced you need to "shard your database for web scale." You've started using terms like "distributed data layer" in planning meetings. Your developers are whiteboarding Zookeeper integrations. Stop. Just stop.

Ninety-nine percent of the applications out there don't have Uber's problems, but their teams are absolutely chomping at the bit to give themselves Uber's operational overhead. They're building a solution for a problem they will likely never have, and in the process, they're turning a perfectly manageable monolith into a distributed nightmare that will haunt their on-call rotation for years.

This isn't a simple choice between two equally valid options. This is a diagnosis. Replication is a treatment for a common set of symptoms: read-heavy workloads and the need for high availability. Sharding is radical, invasive surgery for a terminal condition: a single machine can no longer physically handle your write volume or data size. It introduces a level of architectural and operational cancer that you are, in all likelihood, not prepared for.

So here's the core message, the only thing you need to remember if you get bored and wander off to look at Grafana dashboards: **Start with replication.** Optimize your queries. Tune your configuration. Squeeze every last drop of performance out of your vertical scaling. Buy a server with more RAM than you think is physically possible. Only when your primary node is literally on fire from write contention, when `iostat` looks like a seismograph during an earthquake, should you even whisper the word "shard."

---

### Section 1: Diagnose Before You Operate: What Problem Are You *Actually* Solving?

Before you start architecting your globally-distributed-multi-region-active-active-synergy-platform, do yourself a favor: stop talking about solutions and look at your damn monitoring. What metric is actually red-lining? Your gut feeling is irrelevant. Your CEO's vague notions of "slowness" are actively harmful. Data is all that matters. `pg_stat_activity` is your friend. `EXPLAIN ANALYZE` is your bible. Let's look at the symptoms.

#### Sub-Header: Symptom 1: Read-Bound Workloads

*   **Description:** The primary database server's CPU is pegged at 100%, and when you look at `htop`, you see a dozen `postgres: user db host(pid) SELECT` processes consuming all the cycles. Your application feels sluggish for users trying to view dashboards, browse product catalogs, or read articles. Writes, however, are still snappy. You have a business intelligence team that loves running `SELECT COUNT(*) FROM huge_table;` without a `WHERE` clause, locking up resources for everyone else.
*   **Diagnosis:** This is a classic read scalability problem. Your write capacity is perfectly fine; you just can't serve all the read requests from one box. You're trying to make one waiter serve a 200-table restaurant.
*   **Prescription:** **Replication.** Full stop. This is not a debate. Spin up one or more read replicas, point your application's read traffic at them, and send your BI team to their own dedicated replica where they can run their table scans to their heart's content without affecting production. Problem solved.

#### Sub-Header: Symptom 2: Write-Bound Workloads

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/postgresql-scaling-choosing-your-poison-replication-vs-sharding/](https://www.valtersit.com/guides/databases/postgresql-scaling-choosing-your-poison-replication-vs-sharding/)**
