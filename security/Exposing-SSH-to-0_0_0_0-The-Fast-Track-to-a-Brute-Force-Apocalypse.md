> 📖 **Original article:** [Exposing SSH to 0.0.0.0: The Fast Track to a Brute-Force Apocalypse](https://www.valtersit.com/guides/security/Exposing-SSH-to-0_0_0_0-The-Fast-Track-to-a-Brute-Force-Apocalypse/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

Every day, a junior developer spins up a $5 DigitalOcean droplet, leaves port 22 exposed to `0.0.0.0/0`, enables password authentication, and sets the root password to `Company2026!`. 

A week later, they are posting on Reddit, utterly baffled as to why their server's CPU has been pinned at 100% for five days and their `/var/log/auth.log` is suddenly 50 gigabytes. 

Congratulations. You didn't deploy a web server; you successfully deployed a new worker node for a Russian crypto-mining botnet. Leaving SSH wide open with default configurations is the absolute fastest way to lose control of a Linux machine. 

### The Mechanics: The Background Radiation of the Internet

There is a constant, ambient background radiation of malicious scanning on the internet. Within approximately 45 seconds of your server acquiring a public IPv4 address, automated scanners will find it. 

These botnets don't care who you are. They are blindly hammering port 22 with millions of dictionary attacks and credential stuffing payloads. If your `sshd_config` allows `root` logins with a password, you have already done half their job for them—they don't even have to guess the username. 

They will throw `admin`, `root`, `ubuntu`, and `test` at your daemon ten times a second until the server either crashes from the IO overhead of writing failed auth logs, or they guess the password. Once they have a root shell, they drop a persistence script in `/etc/cron.d`, alter your authorized keys, and start hunting for AWS credentials in your environment variables.

### The Fix: Kill Passwords and Jail the Noise

The Senior Sysadmin approach to SSH is uncompromising: **Passwords are dead.** If you are using a password to authenticate to a production Linux server in the current year, you are doing it wrong. You must enforce public key authentication (specifically Ed25519 keys, stop using weak RSA), disable root login entirely, and disable password authentication. 

As a secondary layer, you implement Fail2Ban or CrowdSec. While dropping passwords makes brute-forcing mathematically impossible, the botnets will still try, filling your disk with garbage logs. Fail2Ban monitors those logs and automatically drops a firewall block on any IP that fails authentication 5 times. 

Finally, move SSH off port 22. It is not "security," it is security through obscurity—but changing the port to `50222` will instantly filter out 98% of the automated script-kiddie noise hitting your daemon.

### The Code & Config

Here is the default garbage you usually find in `/etc/ssh/sshd_config`. 

```text
# THE BAD WAY (A Botnet Welcome Mat)
Port 22
PermitRootLogin yes
PasswordAuthentication yes
```

Here is the hardened, DevSecOps-approved configuration. Open `/etc/ssh/sshd_config`, make these changes, and run `systemctl restart sshd`. (Make sure your SSH key is actually installed before you do this, or you will lock yourself out).

```text
# THE REAL ENGINEER'S WAY (Zero Trust SSH)

# Move the port to drop the automated noise
Port 50222

# Only allow Protocol 2 (Usually default, but be explicit)
Protocol 2

# THE FIX: Kill root login and password auth entirely
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no
PubkeyAuthentication yes

# Optional: Only allow specific users to even attempt login
AllowUsers deploy_user admin_user
```

To stop the log spam, install Fail2Ban (`apt install fail2ban`) and drop this configuration into `/etc/fail2ban/jail.local`:

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/Exposing-SSH-to-0_0_0_0-The-Fast-Track-to-a-Brute-Force-Apocalypse/](https://www.valtersit.com/guides/security/Exposing-SSH-to-0_0_0_0-The-Fast-Track-to-a-Brute-Force-Apocalypse/)**
