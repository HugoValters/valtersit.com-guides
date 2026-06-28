> 📖 **Original article:** [Port 3389 to the World: How to Lose Your Company Data Over the Weekend](https://www.valtersit.com/guides/security/Port-3389-to-the-World-How-to-Lose-Your-Company-Data-Over-the-Weekend/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

It’s 4:30 PM on a Friday. The head accountant insists they need to finish payroll from home over the weekend. The junior IT guy, wanting to be helpful and avoid setting up a VPN profile, logs into the edge firewall, creates a quick NAT rule forwarding TCP 3389 straight to the internal terminal server, and goes to the pub. 

By Monday morning, every file server, domain controller, and backup repository on the network ends in `.lockbit`. 

If you are exposing Remote Desktop Protocol (RDP) directly to the public internet in the current year, you are not administering a network. You are operating a honeypot. It is the single most amateur mistake in Windows system administration, and it is the primary vector for ransomware groups globally.

### The Mechanics: The 48-Hour Guarantee

You might think your obscure static IP address is flying under the radar. It isn't. 

Mass-scanning botnets map the entire IPv4 address space in a matter of hours. When you open port 3389, it takes roughly 45 minutes for automated scanners to find it and flag your IP in a Telegram channel. 

Once the port is discovered, the attack branches into two paths:
1. **Credential Stuffing:** Automated scripts hammer your login prompt with leaked credentials, default passwords, and variations of `Administrator` and `Company2026!`. If you don't have aggressive Active Directory account lockout policies enabled, they will eventually brute-force their way in.
2. **Protocol Exploits:** If the server is missing a few critical Windows Updates, attackers just fire off a known RDP vulnerability payload—like the infamous BlueKeep (CVE-2019-0708). This gives them pre-authentication remote code execution at the SYSTEM level. 

Once they have an RDP session, the game is over. They deploy Cobalt Strike, disable Windows Defender, dump LSASS memory to harvest Domain Admin credentials, and move laterally across your hypervisors. 

### The Fix: Kill the NAT Rule

The Senior Sysadmin approach to RDP is absolute: **Never, under any circumstances, expose 3389 to the internet.** If users need remote access, you implement layers:
1. **The Network Layer:** Users must connect to a proper VPN (WireGuard or IPsec) *before* they can even route to the internal subnet where the RDP server lives.
2. **The Gateway Layer:** If you absolutely cannot use a VPN client, deploy a Remote Desktop Gateway (RD Gateway) behind a reverse proxy. This encapsulates the RDP traffic over HTTPS (TCP 443).
3. **The Identity Layer:** Enforce Multi-Factor Authentication (MFA) via Entra ID, Duo, or Okta at the VPN or Gateway level. Passwords alone are mathematically obsolete.

### The Code & Config

Here is what the firewall rule looks like on an amateurly managed network. Delete this immediately.

```text
# THE BAD WAY (Ransomware Welcome Mat)
# Edge Firewall NAT / Port Forwarding
Rule: 10
Action: ALLOW
Protocol: TCP
Source: ANY (0.0.0.0/0)
Destination Port: 3389
Forward IP: 192.168.10.50 (Internal Terminal Server)
```

Your edge firewall should simply drop all inbound 3389 traffic. 

Inside the network, you must harden the RDP host itself. At a bare minimum, enforce Network Level Authentication (NLA). NLA requires the user to authenticate *before* the server establishes a full RDP session and allocates memory, completely neutralizing entire classes of pre-auth exploits like BlueKeep.

Run this snippet in an elevated PowerShell prompt to force NLA on your Windows Servers, or push it via Group Policy (GPO):

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/Port-3389-to-the-World-How-to-Lose-Your-Company-Data-Over-the-Weekend/](https://www.valtersit.com/guides/security/Port-3389-to-the-World-How-to-Lose-Your-Company-Data-Over-the-Weekend/)**
