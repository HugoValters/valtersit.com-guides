> 📖 **Original article:** [DNS Leaks: The Silent Privacy Killer in Your \"Secure\" VPN](https://www.valtersit.com/guides/networking/DNS_Leaks-The_Silent_Privacy_Killer_in_Your -Secure-VPN/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I love the marketing fluff around consumer VPNs. "Military-grade encryption!" "One-click to total invisibility!" "Hide from the government!" The average user clicks a big green button, sees their IP change to an exit node in Switzerland, and thinks they are suddenly a ghost in the machine.

Meanwhile, I’m sitting on the other side of a network audit, watching their browser dutifully send a DNS query for `suspicious-medical-search.com` directly to their local ISP’s gateway at `192.168.1.1` over the unencrypted WAN interface.

Congratulations. You’ve successfully encrypted your traffic, but you’ve left the phonebook lookup wide open. Your ISP is watching your activities with a bucket of popcorn, and your "anonymity" is a complete and utter delusion. If you haven't hardened your DNS routing, your VPN is nothing more than a shiny, expensive hood ornament.

### The Mechanics: DNS is the Ultimate Snitch

To understand a DNS leak, you have to understand that your operating system is inherently lazy. It wants the fastest path to information. 

When you type a URL into your browser, your computer needs to resolve that name to an IP address before it can even *begin* to route traffic through your VPN tunnel. This is the DNS (Domain Name System) lookup. By default, your computer is configured to use the DNS servers provided by your local router via DHCP—usually your ISP’s servers or 8.8.8.8 (Google).

#### 1. The Metric Trap
In a "secure" VPN setup, the VPN client creates a virtual network interface (like `tun0` or `wg0`) and tries to push a new DNS configuration to the OS. However, the Linux kernel and Windows IP stack use a system called **Interface Metrics**. If your local physical Ethernet or WiFi interface has a lower metric (higher priority) than your VPN interface, the OS might decide that the local DNS server is "closer" and more reliable. 

The OS sends the DNS query out of the physical gateway *outside* the tunnel. The ISP sees the request, logs it, and fulfills it. Only *after* the IP is resolved does the computer then send the actual data through the encrypted VPN tunnel. The damage is already done. 

#### 2. Split-Tunneling Disasters
Many modern VPNs use "Split-Tunneling" to allow local traffic (like printing to your HP LaserJet) while tunneling internet traffic. If this is misconfigured, the OS will often default to the "local" DNS resolver for *everything*, leaking every single domain you visit to the local network.

#### 3. systemd-resolved: The Linux Admin's Headache
On modern Linux distributions (Ubuntu, Fedora), `systemd-resolved` manages DNS. It acts as a local stub resolver. If your VPN client doesn't explicitly talk to `systemd-resolved` to update its internal state, the daemon might keep using the "upstream" ISP servers it found during the initial boot, completely ignoring the VPN's intended DNS settings.

### The Mechanics of the Fix: Enforcing the Tunnel

The Senior Privacy Engineer's approach is based on one rule: **If the DNS query can't go through the tunnel, it doesn't go out at all.**

#### 1. The DNS Kill Switch
A true VPN setup should include a firewall-level "Kill Switch." You configure your local firewall (UFW, iptables, or the Windows Advanced Firewall) to drop any outgoing traffic on port 53 (UDP/TCP) that isn't originating from the VPN interface or directed at the VPN's internal DNS resolver.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/networking/DNS_Leaks-The_Silent_Privacy_Killer_in_Your -Secure-VPN/](https://www.valtersit.com/guides/networking/DNS_Leaks-The_Silent_Privacy_Killer_in_Your -Secure-VPN/)**
