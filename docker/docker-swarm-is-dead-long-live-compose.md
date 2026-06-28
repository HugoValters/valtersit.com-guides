> 📖 **Original article:** [Docker Swarm is Dead. Long Live Compose.](https://www.valtersit.com/guides/docker/docker-swarm-is-dead-long-live-compose/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

## You've Been Lied To

Let's get one thing straight. Someone, somewhere—a blog post, a well-meaning colleague, a YouTube tutorial—told you to run `docker swarm init` on your single-node VPS. They probably used phrases like "production-grade," "future scalability," or "access to advanced features like secrets."

They were wrong.

What you've done is participate in a cargo cult. You've mimicked the rituals of large-scale distributed systems without understanding the underlying principles, and in doing so, you've saddled yourself with a solution that is actively worse than the simple, elegant alternative. You didn't build a robust, scalable system. You built a fragile, over-engineered monument to bad advice.

For a single Docker host, Swarm mode is not just overkill; it's a technical liability. It introduces unnecessary complexity, opaque state management, and fragile failure points for absolutely zero real-world benefit. Modern Docker Compose provides every feature you actually need—secrets, configs, healthchecks, resource limits—with none of the baggage. This isn't a matter of preference. It's about using the right tool for the job, and for a single server, Swarm is a sledgehammer for a thumbtack.

This guide will dissect the technical fallacies of single-node Swarm, show you the correct way to manage your applications with Docker Compose, and provide a clear, actionable plan to migrate away from the mess you've created. It's time to stop playing orchestrator and start shipping software.

---

## The Technical Debt of Unnecessary Orchestration

The "benefits" of single-node Swarm are a mirage. When you peel back the marketing layer, you find a collection of distributed systems components that are fundamentally pointless—and often dangerous—when run on a single machine.

### The Raft Log: A Single Point of Failure You Gifted Yourself

At the heart of Docker Swarm is a Raft consensus log. This is a battle-hardened algorithm designed to keep the state of a distributed system consistent across multiple manager nodes. It's a brilliant piece of engineering for a multi-node cluster, where it ensures that if one manager fails, the others can elect a new leader and continue operating seamlessly.

On your single-node "cluster," there is no consensus. There are no other nodes. There's just a log.

This log, located in `/var/lib/docker/swarm/`, becomes the single, authoritative, and terrifyingly fragile source of truth for your entire setup. All your service definitions, secrets, configs, and network configurations are stored in this binary, opaque directory structure. If that log becomes corrupted due to a disk error, a bad Docker upgrade, or an inopportune power loss, your entire "orchestrator" is toast. Congratulations, you've traded a stateless Docker host (where the source of truth is a version-controlled text file) for a stateful one with a fragile distributed systems component that has no peers to save it.

```bash
# The fragile heart of your single-node "cluster".
# If this directory gets corrupted, have fun rebuilding your services from memory.
# You've turned your server's state from a text file in git into this binary mess.
sudo ls -l /var/lib/docker/swarm/raft/

# total 8
# drwx------ 2 root root 4096 Apr 19 15:30 snap
# drwx------ 2 root root 4096 Apr 19 15:30 wal
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/docker/docker-swarm-is-dead-long-live-compose/](https://www.valtersit.com/guides/docker/docker-swarm-is-dead-long-live-compose/)**
