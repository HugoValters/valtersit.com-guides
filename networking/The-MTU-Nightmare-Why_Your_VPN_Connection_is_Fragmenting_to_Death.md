> 📖 **Original article:** [The MTU Nightmare: Why Your VPN Connection is Fragmenting to Death](https://www.valtersit.com/guides/networking/The-MTU-Nightmare-Why_Your_VPN_Connection_is_Fragmenting_to_Death/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I’ve lost count of how many times I’ve had to explain this to "Senior" DevOps engineers who think they’ve discovered a ghost in the machine. The symptoms are always the same: The VPN tunnel is "Up." You can ping the remote gateway. You can even SSH into the server. But the moment you run `ls -al` in a directory with 500 files, or try to `scp` a database dump, the terminal just... hangs. 

The website starts to load—you see the favicon and the title in the tab—and then it spins forever, eventually timing out with a "Connection Reset." 

You check the logs. Nothing. You check the CPU. Idle. You check the bandwidth. Plenty. 

Congratulations. You aren't being hacked, and your app isn't buggy. You are suffering from an **MTU mismatch**, and your packets are being ruthlessly discarded by a "black hole" router that someone—likely a junior security admin with an itchy trigger finger—configured to block the very protocol meant to prevent this exact disaster.

### The Mechanics: The 1500-Byte Delusion

To understand why your VPN is dying, you have to understand the **Maximum Transmission Unit (MTU)**. On a standard Ethernet network, the MTU is almost universally 1500 bytes. This is the largest frame that can be sent across the physical wire without being broken into pieces.

When your application sends a packet, it assumes it has 1500 bytes of "room." It builds a 1500-byte IP packet. But your application is inside a VPN tunnel.

#### 1. The VPN Tax (Overhead)
A VPN is an encapsulation protocol. It takes your original 1500-byte packet and wraps it inside *another* packet (the tunnel header) to send it across the internet. 
- **WireGuard** adds roughly 60-80 bytes of overhead.
- **OpenVPN** or **IPsec** can add 60 to 100+ bytes depending on the cipher and padding.

If your original packet was already 1500 bytes, and you add 80 bytes of WireGuard headers, you now have a 1580-byte packet. When that packet hits the physical network interface, the OS realizes it’s too big for the 1500-byte limit of the physical wire.

#### 2. Fragmentation vs. The "Don't Fragment" Bit
In a perfect world, the router would just "fragment" the packet—break it into two smaller pieces. But modern TCP/IP implementations almost always set the **DF (Don't Fragment)** bit. Why? Because fragmentation is computationally expensive and introduces massive latency. 

When a 1580-byte packet with the DF bit set hits a router with a 1500-byte MTU, the router is supposed to drop the packet and send back an ICMP message: **Type 3, Code 4: "Destination Unreachable, Fragmentation Needed and DF set."**

This message includes the "Next-Hop MTU," telling the sender exactly how small the packet needs to be to fit. This process is called **Path MTU Discovery (PMTUD)**.

### The "Black Hole" Router: A Security Admin's Crime

Here is where the nightmare begins. Many "security-conscious" admins think that ICMP is "scary" or "dangerous." They configure their firewalls to `DROP` all ICMP traffic, thinking they are hiding from pings. 

By dropping all ICMP, they have effectively blinded the network. Your server sends the 1580-byte packet. The intermediate router drops it and sends the "Fragmentation Needed" message. Your firewall sees the ICMP message, thinks it’s an attack, and drops it. 

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/networking/The-MTU-Nightmare-Why_Your_VPN_Connection_is_Fragmenting_to_Death/](https://www.valtersit.com/guides/networking/The-MTU-Nightmare-Why_Your_VPN_Connection_is_Fragmenting_to_Death/)**
