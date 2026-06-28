> 📖 **Original article:** [Still Running SMBv1? You're Basically Inviting WannaCry to Dinner](https://www.valtersit.com/guides/security/Still-Running-SMBv1-You_re-Basically-Inviting-WannaCry-to-Dinner/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

It is 2026. The fact that I still walk into enterprise environments and see Server Message Block version 1 (SMBv1) enabled on Domain Controllers because the HR department refuses to replace a multifunction Xerox scanner from 2005 is a complete dereliction of duty. 

Let me be absolutely clear: SMBv1 is a 30-year-old protocol that was deprecated before most junior sysadmins even graduated high school. Keeping it active on your network isn't a "business requirement," it is gross negligence. If you have SMBv1 running, you are effectively leaving the front door to your datacenter wide open with a neon sign pointing to the servers.

### The Mechanics: The EternalBlue Nightmare

To understand why SMBv1 is so lethal, you have to look at EternalBlue (CVE-2017-0144), the exploit that powered the WannaCry and NotPetya global meltdowns. 

This isn't a phishing attack that requires a user to click a bad link. It requires absolutely zero user interaction. If SMBv1 is enabled, an attacker (or a self-propagating worm) simply sends a specially crafted, unauthenticated packet to the target's `IPC$` share. 

Because of a massive vulnerability in how the Windows kernel driver (`srv.sys`) handles these packets, that single malformed request triggers a buffer overflow. The attacker instantly gains arbitrary code execution at the Ring 0 (SYSTEM) level. They don't just own the machine; they own the kernel. From there, the worm scans the local subnet, finds every other machine listening on TCP 445 with SMBv1 enabled, and infects the entire VLAN in a matter of seconds.

### The Fix: Nuke It From Orbit

You do not mitigate SMBv1. You exterminate it. 

The Senior Sysadmin approach is to aggressively disable the protocol across every single endpoint and server in the domain. If that legacy 2005 scanner suddenly breaks, good. Tell them to buy a $300 printer from Best Buy. If management absolutely insists that a legacy piece of manufacturing hardware *must* use SMBv1, you physically isolate that machine on a completely locked-down, air-gapped VLAN with zero routing access to the rest of the corporate network. 

### The Code & Config

Stop clicking through GUI menus. Here is the exact PowerShell and Group Policy configuration to rip SMBv1 out of your infrastructure permanently.

First, run this on your Windows Servers to kill the service immediately without a reboot:

```powershell
# THE REAL ENGINEER'S WAY (Kill SMBv1 on Windows Server)

# Check if the protocol is even enabled
Get-SmbServerConfiguration | Select EnableSMB1Protocol

# Terminate it with extreme prejudice
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force

Write-Host "SMBv1 Server service terminated." -ForegroundColor Green
```

To rip the client feature out of Windows 10/11 workstations, use the Optional Features cmdlet:

```powershell
# Kill SMBv1 Client on Windows Workstations
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart
```

Finally, to ensure no rogue machine or helpdesk tech ever turns it back on, you enforce this registry key via Group Policy Object (GPO) applied to the entire domain:

```text
# THE GPO NUKE (Enforce via Group Policy Preferences -> Registry)

Action: Update
Hive: HKEY_LOCAL_MACHINE
Key Path: SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters
Value Name: SMB1
Value Type: REG_DWORD
Value Data: 0
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/Still-Running-SMBv1-You_re-Basically-Inviting-WannaCry-to-Dinner/](https://www.valtersit.com/guides/security/Still-Running-SMBv1-You_re-Basically-Inviting-WannaCry-to-Dinner/)**
