> 📖 **Original article:** [Dual-WAN Failover: Because 'My Internet is Down' is Not an Option](https://www.valtersit.com/guides/networking/Dual-WAN-Failover-Because_My_Internet_is_Down_is_Not_an_Option/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I once worked with a CEO who spent $200,000 on a high-availability server cluster, redundant power supplies, and a mirrored storage array, but refused to pay $50 a month for a secondary LTE backup line. "Our fiber has a 99.9% uptime SLA," he told me.

Two weeks later, a construction crew three blocks away put a backhoe through the primary conduit. The "99.9% SLA" meant the ISP owed him a $40 credit on his next bill, while the business lost $15,000 in sales during the four-hour repair window. 

If your business relies on a single physical cable for its entire existence, you aren't an architect; you're a gambler. In the world of 2026, where cloud services and remote APIs are the lifeblood of every operation, "My internet is down" is not an acceptable status update. It is a failure of planning. 

But here is the real kicker: most people who *do* have two WAN lines have configured them so poorly that they don't actually fail over when the real disaster hits. If your "Failover" strategy is just two default routes with different distances, you are in for a very rude awakening.

### The Mechanics: The "Link Up" Delusion

Most junior admins think failover is easy. They plug ISP A into `ether1` and ISP B into `ether2`. They add two default routes in their MikroTik:
- Route 1: `dst-address=0.0.0.0/0 gateway=ISP_A distance=1 check-gateway=ping`
- Route 2: `dst-address=0.0.0.0/0 gateway=ISP_B distance=2`

They pull the cable out of `ether1`, the route disappears, the distance-2 route takes over, and they congratulate themselves. 

Here is the problem: **Cables rarely just get pulled out.** In the real world, the "failure" usually happens five miles away at the ISP's headend, or in a routing loop at their upstream provider, or a DNS failure at their gateway. Your router’s `ether1` port will still be "Up." The Link Light will be green. The ISP's local gateway at the other end of the fiber will still respond to a ping. 

To your router, everything looks fine. It keeps trying to shove packets down a dead pipe because the local link is active. Your users are down, but your routing table says "Active." This is the "Gateway is Up, Internet is Down" trap, and it’s the most common cause of failed failovers.

### The Mechanics: Recursive Routing (The "Smart" Check)

The Senior Network Engineer's fix is **Recursive Routing**. Instead of asking the router "Can you talk to the ISP's gateway?", we ask "Can you reach the actual internet through this gateway?"

We do this by creating "virtual" hops. We tell the router that a reliable 3rd-party IP on the internet—like Google’s `8.8.8.8` or Cloudflare’s `1.1.1.1`—is actually the "gateway" for our check.

#### 1. The Scope and Target Scope
In MikroTik’s RouterOS, every route has a `scope` and a `target-scope`. By manipulating these, we can force the router to look up the gateway for one route by looking at the results of another. 

We set up a route to `8.8.8.8` through our ISP A gateway. Then, we set our actual default route (`0.0.0.0/0`) to use `8.8.8.8` as the gateway. Now, the default route only stays active if the router can successfully ping `8.8.8.8` through the ISP A path. If the ISP's upstream goes down, the ping to `8.8.8.8` fails, the "recursive" route is marked as unreachable, and the distance-2 backup line instantly takes over.

### The Fix: Failover vs. Load Balancing (PCC)

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/networking/Dual-WAN-Failover-Because_My_Internet_is_Down_is_Not_an_Option/](https://www.valtersit.com/guides/networking/Dual-WAN-Failover-Because_My_Internet_is_Down_is_Not_an_Option/)**
