> 📖 **Original article:** [Tailscale & ZeroTier: Why You're Fighting CGNAT and Losing](https://www.valtersit.com/guides/networking/Tailscale_ZeroTier-Why_Youre_Fighting_CGNAT-and-Losing/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I recently talked to a developer who spent three days trying to set up a WireGuard tunnel to his home server. He had the config perfect. He had the port forwarded on his router. He had the dynamic DNS updating every five minutes. But no matter what he did, he couldn't get a handshake. 

I told him to check his WAN IP on his router and compare it to what "WhatIsMyIP" told him. Sure enough, his router showed a `100.64.x.x` address. 

"You're behind CGNAT," I said. "Your ISP isn't giving you a public IP. You're effectively behind a firewall that you don't own and can't control. You can't forward a port if the port doesn't belong to you."

If you’re still trying to use traditional "inbound" VPNs in an era where ISPs are hoarding IPv4 addresses like digital dragons, you aren't fighting a technical battle; you're fighting a losing war against the exhaustion of the internet's address space. It’s 2026. If you want to connect your devices without losing your mind to "Double NAT" hell, it’s time to embrace the "magic" of overlay networks like Tailscale and ZeroTier.

### The Rant: The CGNAT Death Trap

Carrier-Grade NAT (CGNAT), specifically defined in RFC 6598, is the ISP's solution to the fact that we ran out of IPv4 addresses years ago. Instead of giving every customer a unique public IP, the ISP puts thousands of customers behind a single public IP. Your router gets a private address in the `100.64.0.0/10` range. 

This works fine for 99% of people who just want to browse TikTok. But the moment you want to host a VPN, a Minecraft server, or a Plex instance, you hit a brick wall. Because you don't have a unique public IP, there is no way for a packet from the outside world to "find" your router. Traditional port forwarding is dead. 

You can call your ISP and beg for a "Static IP" (which they will happily charge you $15/month for), or you can stop living in the past and use a mesh VPN that was designed to handle this exact nightmare.

### The Mechanics: How the "Magic" Works

Tailscale and ZeroTier are often described as "magic" because you just install them, log in, and suddenly your devices can talk to each other as if they were on the same switch, regardless of firewalls, NAT, or CGNAT. 

This isn't magic; it's sophisticated orchestration using three core technologies: **STUN**, **TURN**, and **UDP Hole Punching**.

#### 1. STUN (Finding the Exit)
STUN (Session Traversal Utilities for NAT) is how a device finds out what it "looks like" to the outside world. Your server sends a packet to a STUN server on the public internet. The STUN server replies: "Hey, I saw your packet coming from public IP `203.0.113.5` on port `54321`." 

Now your server knows its external "mapped" address.

#### 2. UDP Hole Punching (The Handshake)
This is the brilliant part. Most firewalls are "stateful." They block incoming packets by default, but they allow outgoing packets. Crucially, if you send an outgoing UDP packet to a destination, the firewall opens a temporary "hole" to allow a response from that same destination to come back in.

Tailscale’s coordination server tells both Peer A and Peer B: "Okay, both of you send a UDP packet to each other's mapped STUN addresses *at the same time*." 

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/networking/Tailscale_ZeroTier-Why_Youre_Fighting_CGNAT-and-Losing/](https://www.valtersit.com/guides/networking/Tailscale_ZeroTier-Why_Youre_Fighting_CGNAT-and-Losing/)**
