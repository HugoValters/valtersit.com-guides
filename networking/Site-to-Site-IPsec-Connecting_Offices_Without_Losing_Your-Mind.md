> 📖 **Original article:** [Site-to-Site IPsec: Connecting Offices Without Losing Your Mind](https://www.valtersit.com/guides/networking/Site-to-Site-IPsec-Connecting_Offices_Without_Losing_Your-Mind/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I have spent a significant portion of my adult life staring at log buffers, watching two expensive routers scream at each other in a language they both ostensibly speak, yet neither understands. There is a specific kind of hell reserved for the sysadmin trying to connect a MikroTik at a branch office to a Cisco ASA at the headquarters. 

You’ve got the subnets right. You’ve got the public IPs right. You’ve even double-checked the Pre-Shared Key (PSK) five times. And yet, there it is: `phase1 negotiation failed` or the even more insulting `no proposal chosen`. 

If you’ve ever found yourself clicking through 50 different permutations of "3DES," "AES-128," "SHA1," and "MD5" just to get a tunnel to stay up for more than ten minutes, you aren't doing engineering; you're doing digital alchemy. It’s 2026. If you are still relying on legacy IKEv1 and "guessing" which encryption suite your vendor implemented correctly, you are a dinosaur. It is time to standardize, move to IKEv2, and stop treating your Site-to-Site tunnels like a delicate chemistry experiment.

### The Rant: The "Phase 1 / Phase 2" Mismatch Hell

The core problem with IPsec is that it was designed by a committee that hated simplicity. Unlike WireGuard, which has one way to do things, IPsec has five thousand. 

#### 1. The Multi-Vendor Alphabet Soup
Every vendor (Cisco, MikroTik, Fortinet, Juniper, Ubiquiti) has a slightly different idea of what "Standard" means. Cisco might default to a 24-hour lifetime for Phase 1, while MikroTik defaults to 1 day (which is almost, but not quite, the same depending on how they count seconds). One vendor might include the "Local ID" in the identity check, while another ignores it. 

The moment you try to connect two different brands, you enter the Mismatch Trap. You spend four hours on the phone with "The Cisco Guy" at HQ, both of you insisting your settings are correct, while the logs show a confusing mess of "Invalid ID Information" and "Notify: No Proposal Chosen."

#### 2. The Legacy Burden
Most people are still using IKEv1 because "that’s what the tutorial said." IKEv1 is a bloated, slow, and insecure relic. It requires multiple round-trips to establish a session, it handles NAT poorly, and it is prone to fragmentation issues. If you are still using Main Mode or Aggressive Mode in 2026, you are inviting instability.

### The Mechanics: IKEv2 and the Glory of NAT-T

If you want to keep your sanity, the first rule is: **IKEv2 only.** #### 1. IKEv2 (Internet Key Exchange v2)
IKEv2 was designed to fix everything that sucked about IKEv1. It has built-in support for MOBIKE (mobility), which means your tunnel can survive a brief IP change. It has a much more streamlined handshake (fewer packets, less latency). Most importantly, it handles **Dead Peer Detection (DPD)** as a core part of the protocol, not as a vendor-specific hack. If the tunnel drops, IKEv2 knows immediately and starts a re-negotiation.

#### 2. NAT-Traversal (NAT-T)
In the age of CGNAT and multi-layered NAT, IPsec usually dies because it relies on protocol 50 (ESP), which doesn't have "ports" like TCP or UDP. Many cheap ISP routers have no idea how to handle ESP and simply drop it. 

NAT-T solves this by wrapping the entire IPsec packet inside a UDP header on port 4500. This is mandatory for modern offices. If even one side of your tunnel is behind a NAT, you must enable NAT-T, or your Phase 2 will never initialize.

### The Fix: Standardize Your Suite (AEAD)

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/networking/Site-to-Site-IPsec-Connecting_Offices_Without_Losing_Your-Mind/](https://www.valtersit.com/guides/networking/Site-to-Site-IPsec-Connecting_Offices_Without_Losing_Your-Mind/)**
