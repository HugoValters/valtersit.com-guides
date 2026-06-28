> 📖 **Original article:** [Elasticsearch Split-Brain: When Your Cluster Develops Multiple Personalities](https://www.valtersit.com/guides/databases/Elasticsearch_Split-Brain-When_Your_Cluster_Develops_Multiple_Personalities/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I once walked into a disaster recovery room where a mid-sized e-commerce company was losing its mind. They had built a "high availability" Elasticsearch cluster with exactly two nodes to save on infrastructure costs. Everything was fine until a momentary network blip—a transient micro-outage in the top-of-rack switch—lasted for about three seconds. 

By the time the network recovered, the company didn't have one cluster anymore. They had two independent clusters that both claimed to be the "original." Both nodes had promoted themselves to master, both were happily ingesting new logs and orders, and both were diverging further apart by the millisecond. 

Congratulations. You just encountered "Split-Brain," and you are about to find out that merging two diverging Elasticsearch clusters is about as fun as un-scrambling an egg. 

### The Mechanics: The Quorum Math You Ignored

To understand why this happens, you have to understand how Elasticsearch nodes decide who is in charge. In any distributed system, you need a "Master" node to manage the cluster state—deciding where shards live, creating indices, and handling node departures.

When a network partition occurs (a "split"), nodes on both sides of the divide look around. If they can't see the current master, they hold an election. 

#### The 2-Node Trap
In a 2-node cluster, if Node A stops seeing Node B, Node A thinks: "Well, Node B must be dead. I am now the master." Simultaneously, Node B thinks: "Node A is gone. I am now the master." 

Since there is no tie-breaker, you now have two masters. This is a "Split-Brain" scenario. Your application's load balancer will continue sending data to both. Some search results will come from Node A, some from Node B. When the network link finally heals, the nodes will see each other, realize they both think they are master, and the cluster will likely panic or—worse—one node will "win" and overwrite the other's data, leading to permanent data loss.

### The Fix: The Rule of Three (and Odd Numbers)

The Senior DevOps fix is non-negotiable: **You never, ever run a production cluster with fewer than three master-eligible nodes.** Distributed systems rely on a concept called "Quorum." Quorum is the minimum number of votes required to make a decision. The mathematical formula for a safe quorum is: 
` (Number of Master-Eligible Nodes / 2) + 1 `

- In a 3-node cluster, Quorum is 2. If one node drops, the other two can see each other, reach Quorum (2), and continue safely. If the network splits into a 1-node side and a 2-node side, only the 2-node side can reach Quorum. The 1-node side knows it is isolated and stays in a "follower" state, refusing to accept writes. 
- In a 2-node cluster, Quorum is 2. If the network splits, neither side can reach 2 votes. The cluster correctly freezes until the nodes can talk again. 

**Wait, then why did the 2-node cluster in my rant fail?** Because the admins forgot to configure the `minimum_master_nodes` setting, allowing a single node to elect itself.

### The Evolution of the Fix: Legacy vs. Modern

If you are running an ancient version of Elasticsearch (pre-7.0), you had to manually calculate and set `discovery.zen.minimum_master_nodes`. If you messed up the math or forgot to update it when adding nodes, you were asking for a split-brain.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/Elasticsearch_Split-Brain-When_Your_Cluster_Develops_Multiple_Personalities/](https://www.valtersit.com/guides/databases/Elasticsearch_Split-Brain-When_Your_Cluster_Develops_Multiple_Personalities/)**
