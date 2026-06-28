> 📖 **Original article:** [SSH Tunneling: The Swiss Army Knife of Networking You're Not Using](https://www.valtersit.com/guides/networking/SSH_Tunneling-The_Swiss_Army_Knife_of_Networking_Youre_Not_Using/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I recently watched a DevOps team spend three days arguing over the routing tables of a new WireGuard Site-to-Site tunnel. They just wanted to give a single developer access to a staging Prometheus dashboard that was sitting on a private subnet behind an ivory tower of firewalls. Three days of YAML, routing conflicts, and security group adjustments for a task that could have been solved in three seconds with a single CLI command.

I walked over, typed `ssh -L 9090:localhost:9090 staging-gw`, and the dashboard was live on their local machine. 

"Is that secure?" they asked.

If you don't trust the SSH tunnel you’re already using to manage the server, you have much bigger problems than a port forward. 

In the era of "Zero Trust" and complex overlay networks, we’ve forgotten the raw, elegant power of the SSH tunnel. It is the Swiss Army Knife of networking. It bypasses firewalls, encrypts insecure protocols, and punches through NAT like a hot knife through butter. But it also has sharp edges. If you don't understand the difference between `-L` and `-R`, or if you try to use it as a permanent networking backbone, you’re going to find out exactly why "TCP-over-TCP" is a phrase that makes veteran network engineers break out in a cold sweat.

### The Rant: The Over-Engineered VPN Trap

We have become obsessed with infrastructure as a service. We want a "solution" for everything. Need to access a database? "Deploy a VPN." Need to bypass a proxy? "Set up a SASE gateway." 

Don't get me wrong; VPNs have their place (as I've written about extensively). But for the surgical, temporary, or emergency tasks that define the life of a Senior Sysadmin, a VPN is a sledgehammer where a scalpel is required. 

The SSH daemon is already there. It’s audited, it’s hardened, and it’s already authenticated. Using it for port forwarding isn't "hacking" the system; it’s using the protocol for exactly what it was designed to do: provide a secure channel for data. 

If you are setting up a 50-node mesh network just so one person can check a `phpMyAdmin` page on a legacy server, you aren't being "secure"—you're being inefficient.

### The Mechanics: How SSH Port Forwarding Works

At its core, SSH tunneling is about **Encapsulation**. You are taking a raw TCP stream from a local or remote port and wrapping it inside the encrypted SSH session. The SSH client and server act as the "entry" and "exit" points for these packets.

There are three primary types of tunnels you need to master. If you have to look at the `man` page every time you use them, you haven't been in the trenches long enough.

#### 1. Local Port Forwarding (`-L`): The "Pull"
Local forwarding is used when you want to access a service on a remote network as if it were running on your local machine. 

**The Syntax:** `ssh -L [local-address:]local-port:remote-host:remote-port server`

Imagine a database server (`db-internal`) that only listens on `localhost:3306` for security reasons. You are sitting at home, connected via SSH to a jump box (`gateway`). 

When you run `ssh -L 3306:db-internal:3306 gateway`, your local SSH client starts listening on `localhost:3306`. When your local DB tool connects to `localhost:3306`, the SSH client intercepts the packets, encrypts them, sends them through the tunnel to the `gateway`, and the `gateway` then opens a connection to `db-internal:3306`. 

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/networking/SSH_Tunneling-The_Swiss_Army_Knife_of_Networking_Youre_Not_Using/](https://www.valtersit.com/guides/networking/SSH_Tunneling-The_Swiss_Army_Knife_of_Networking_Youre_Not_Using/)**
