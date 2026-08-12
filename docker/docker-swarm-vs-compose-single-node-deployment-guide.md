> 📖 **Original article:** [Docker Swarm vs Compose: Single-Node Deployment Guide](https://www.valtersit.com/guides/docker/docker-swarm-vs-compose-single-node-deployment-guide/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

You've built the perfect `docker-compose.yml`. It runs flawlessly on your laptop, the tests pass, and you're feeling good. You deploy to your single $20 VPS, go to bed, and wake up to a 3 AM page: your site is down. The container crashed, and nothing brought it back. This isn't a code problem—it's an orchestration problem.

I've watched this exact scenario kill more deployments than any code bug. The issue is that Docker's marketing has convinced you that Compose is a production tool. It's not. It's a development convenience that happens to run containers in production if you're lucky. When you have a single node, you face a critical choice: do you need a full orchestrator, or just a better process manager?

This guide is for engineers running production workloads on single VPS instances. After reading this, you'll understand the fundamental philosophical difference between Compose and Swarm, know exactly when to use each, and have the ammunition to defend your choice in your next architecture review. We're covering config management, service discovery, rolling updates, and the dirty secret of Swarm's networking on a single host.

:::note[TL;DR]
- Compose is a recipe; Swarm is a controller. One defines, the other enforces.
- Swarm's self-healing and rollback capabilities are worth the 50-100 MB RAM overhead on any production node.
- Use `mode: host` for published ports in single-node Swarm to bypass the useless ingress load balancer.
- Docker Secrets beats environment variables for any credential, even on a single node.
- If you're on a 512 MB VPS, use Compose. Otherwise, Swarm is the production-grade choice.
:::

**Prerequisites:** Docker Engine 24.x+, basic familiarity with `docker run`, a healthy fear of `latest` tags, and a production server that's been keeping you up at night.

## The False Dichotomy: Why "Single Node" Doesn't Mean "Simple"

The biggest misconception I hear is: "I have one server, so I just need to run containers. Compose is easier, so I'll use it. Swarm is for clusters." This thinking is dangerously wrong and will cost you a weekend.

The real difference comes down to **desired state vs. imperative scripts**. Compose is a recipe—it defines what should run, but it doesn't enforce anything. If a process crashes, it stays dead unless you manually added `restart: unless-stopped`, which is a band-aid, not a solution. Swarm is a controller—it constantly reconciles the current state with the desired state. If a container dies, the Swarm manager schedules a replacement. This is a fundamental philosophical difference, not a feature list.

Here's the deceptive `restart` policy that gives you false confidence:

```yaml
# docker-compose.yml - The "It Works" Lie
version: "3.8"
services:
  web:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
```

```yaml
# docker-stack.yml - The Same Service in Swarm
services:
  web:
    image: nginx:alpine
    ports:
      - target: 80
        published: 80
        mode: host
```

Notice what's missing in the Swarm file? No `restart` policy. Swarm doesn't need it—if the container dies, the orchestrator creates a new one. The `restart` in Compose is a local Docker daemon feature that doesn't handle dependencies. If your database takes 30 seconds to initialize and your app crashes 5 times in that window, Compose gives up. Swarm keeps trying. This is the first production killer.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/docker/docker-swarm-vs-compose-single-node-deployment-guide/](https://www.valtersit.com/guides/docker/docker-swarm-vs-compose-single-node-deployment-guide/)**
