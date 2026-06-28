> 📖 **Original article:** [The pg_hba.conf 'Trust' Trap: How to Give Your PostgreSQL Database Away for Free](https://www.valtersit.com/guides/databases/The-pg_hba_conf-Trust-Trap-How-to-Give-Your-PostgreSQL-Database-Away-for-Free/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I absolutely despise "quick start" database tutorials. A developer will spend three weeks meticulously crafting a frontend UI, but the second they get a `password authentication failed` error from PostgreSQL, they lose their minds. They hit Stack Overflow, find a heavily upvoted answer from 2013, and change their `pg_hba.conf` to `host all all 0.0.0.0/0 trust` just to get the app connected.

"It's fine, it's just for local development," they say. Six months later, that exact configuration file is deployed to the production cluster because nobody bothered to write a proper deployment manifest. 

Congratulations. You didn't configure a database; you configured an open, unauthenticated file share for your customer data. 

### The Mechanics: What "Trust" Actually Means

Developers fundamentally misunderstand what `trust` means in PostgreSQL. They assume it means "trust this internal network to send a password." 

It doesn't. It means "completely bypass the authentication module."

When a connection request hits port 5432, Postgres parses `pg_hba.conf` sequentially from top to bottom. If the source IP matches a line ending in `trust`, the database skips the password challenge entirely. It simply looks at the username asserted in the TCP startup packet. 

If the client application says, "Hi, I'm the `postgres` superuser," the database replies, "Sounds legit, here is absolute root access." 

This is an architectural disaster. If an attacker finds a basic Server-Side Request Forgery (SSRF) vulnerability in your frontend, or compromises a low-level background worker container sitting on the same Docker bridge network, they don't need to hunt for your `.env` files or reverse-engineer your password vault. They just pivot, connect to the database IP, assert the `postgres` user, and run `pg_dump`. 

### The Fix: SCRAM-SHA-256 and Subnet Isolation

The Senior DBA approach is binary: you never use `trust` over TCP/IP connections. Ever. 

You must restrict the source IP subnets explicitly to the CIDR blocks of your backend applications. You do not allow `0.0.0.0/0`. 

Furthermore, you enforce `scram-sha-256` for authentication. Do not use `md5`. MD5 was cryptographically broken over a decade ago, and it's vulnerable to pass-the-hash attacks and sniffing. Modern PostgreSQL (v14+) defaults to SCRAM (Salted Challenge Response Authentication Mechanism) for a reason. It prevents password sniffing and cryptographically proves both the client and the server know the password without transmitting it.

### The Code & Config

Here is the dumpster-fire `pg_hba.conf` that guarantees a data breach disclosure:

```text
# THE BAD WAY (A Resume Generating Event)
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     trust
host    all             all             127.0.0.1/32            trust
host    all             all             0.0.0.0/0               trust
```

Here is the hardened, DevSecOps-approved configuration. We allow `peer` authentication strictly for local UNIX sockets (so local admin scripts can run), but all TCP traffic is locked down to the backend subnet and heavily authenticated.

```text
# THE REAL ENGINEER'S WAY (Zero Trust Database)
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# 1. Allow OS-level administrators to connect locally via UNIX sockets
local   all             postgres                                peer
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/The-pg_hba_conf-Trust-Trap-How-to-Give-Your-PostgreSQL-Database-Away-for-Free/](https://www.valtersit.com/guides/databases/The-pg_hba_conf-Trust-Trap-How-to-Give-Your-PostgreSQL-Database-Away-for-Free/)**
