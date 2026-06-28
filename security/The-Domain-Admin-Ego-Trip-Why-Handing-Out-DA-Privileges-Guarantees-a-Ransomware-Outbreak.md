> 📖 **Original article:** [The 'Domain Admin' Ego Trip: Why Handing Out DA Privileges Guarantees a Ransomware Outbreak](https://www.valtersit.com/guides/security/The-Domain-Admin-Ego-Trip-Why-Handing-Out-DA-Privileges-Guarantees-a-Ransomware-Outbreak/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I frequently audit corporate networks where 15 different people are sitting in the `Domain Admins` group. When I ask the IT Director why the junior helpdesk tech and the database administrator hold the literal keys to the entire Active Directory forest, the answer is always some variation of, "It makes fixing printer permissions easier."

No, it doesn't. It makes you lazy. Handing out Domain Admin (DA) privileges like candy isn't an IT strategy; it's an ego trip for the IT staff and a guaranteed ransomware deployment for the business.

If your helpdesk uses a DA account to log into user workstations, you are actively facilitating your own network's destruction. 

### The Mechanics: How Mimikatz Eats Your Domain

You don't need to be hacked by a nation-state to lose your domain. An attacker only needs to phish one low-level employee. Let's say Kevin in HR clicks a bad PDF, and an attacker gets a silent, unprivileged reverse shell on his laptop.

Kevin notices his computer is acting sluggish and calls the helpdesk. The lazy helpdesk tech, using their daily driver account which happens to be in the `Domain Admins` group, RDPs into Kevin's infected workstation to take a look. 

The moment that DA account authenticates to the workstation, Windows caches their credentials in the Local Security Authority Subsystem Service (LSASS) process memory. 

The attacker, sitting quietly on Kevin's machine, runs a tool like Mimikatz. They scrape the LSASS memory and extract the helpdesk tech's NTLM hash or plain-text password. Because the tech is a DA, the attacker takes that hash, uses Pass-the-Hash to authenticate directly to the Domain Controller, and dumps the `NTDS.dit` database. 

The attacker now owns every password of every user and service account in your company. The domain is dead. 

### The Fix: Tiered Administration and LAPS

The Senior Sysadmin fix is non-negotiable: **The Tiered Administration Model**. 

Domain Admins (Tier 0) must *never* log into workstations (Tier 2) or standard application servers (Tier 1). Ever. If a DA account logs into a workstation, that workstation is now effectively a Domain Controller from a risk perspective. 

To give your helpdesk the ability to fix Kevin's computer without using a DA account, you implement the Principle of Least Privilege and deploy Microsoft LAPS (Local Administrator Password Solution). 

LAPS randomizes the built-in local administrator password on every single workstation in the domain, makes it completely unique, changes it automatically every 30 days, and stores it securely in a hidden Active Directory attribute. When the helpdesk needs to fix a PC, they look up that specific PC's LAPS password, use it, and move on. If the PC is compromised, the attacker only gets a local admin password that is useless on every other machine in the network.

### The Code & Config

Deploying LAPS is not hard. It takes 15 minutes of PowerShell on a Domain Controller to stop 90% of lateral movement attacks.

Here is the PowerShell snippet to prepare your Active Directory schema and delegate the correct permissions so your helpdesk can actually read the LAPS passwords without needing Domain Admin rights.

```powershell
# THE REAL ENGINEER'S WAY (Deploying LAPS via PowerShell)

# 1. Import the LAPS module on your Domain Controller
Import-Module AdmPwd.PS

# 2. Update the AD Schema to include the ms-Mcs-AdmPwd attributes
Update-AdmPwdADSchema
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/The-Domain-Admin-Ego-Trip-Why-Handing-Out-DA-Privileges-Guarantees-a-Ransomware-Outbreak/](https://www.valtersit.com/guides/security/The-Domain-Admin-Ego-Trip-Why-Handing-Out-DA-Privileges-Guarantees-a-Ransomware-Outbreak/)**
